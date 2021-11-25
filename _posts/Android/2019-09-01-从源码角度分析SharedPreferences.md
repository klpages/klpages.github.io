---
layout:     post
title:      从源码角度分析SharedPreferences
subtitle:   
date:       2019-09-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---
### 一、概述   
SharedPreferences简称SP,我相信大家对它很熟悉，开发时经常会用到。它主要的用途是以key-value(键值对)的形式存储一些轻量级的数据(比如应用的配置信息等),数据的保存格式为xml。

#### 1.1 SP的特性/使用规范    
a、只能存储少量数据      
b、不要频繁调用edit()方法/不要频繁实例化Editor     
c、commit()运行在主线程，用于同步提交数据，可以拿到返回结果；apply()运行在子线程，用于异步提交数据，不会返回数据提交结果。

了解了SP的特性后，我们可以在它的特性前加个“why”，然后带着这些疑问去探索它的源码。

### 二、SharedPreferences源码分析    

#### 2.1 初始化SharedPreferences   
我们使用SP时，初始化方式如下：   
```java
	//方式一
	SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(context);

	//方式二
	SharedPreferences sharedPreferences = getSharedPreferences("test", MODE_PRIVATE);

	//方式三
	......
```
上面的初始化方式最终都是调用new SharedPreferencesImpl(file,model)方法来进行实例化的，因此我们可直接查看SharedPreferencesImpl的源码。SharedPreferences是接口，SharedPreferencesImpl是它的实现类   

```java
final class SharedPreferencesImpl implements SharedPreferences {
    @GuardedBy("mLock")
    private boolean mLoaded = false;//标记是否已读取SP文件(是否已加载到内存)
     //构造方法
     SharedPreferencesImpl(File file, int mode) {
        mFile = file;
        mBackupFile = makeBackupFile(file);
        mMode = mode;
        mLoaded = false;
        mMap = null;
        mThrowable = null;
	//从磁盘读取sp文件
        startLoadFromDisk();
    }

    private void startLoadFromDisk() {
        synchronized (mLock) {
            mLoaded = false;
        }
	//创建线程从磁盘加载sp文件
        new Thread("SharedPreferencesImpl-load") {
            public void run() {
                loadFromDisk();
            }
        }.start();
    }

    private void loadFromDisk() {
        synchronized (mLock) {
            if (mLoaded) {
                return;
            }
            if (mBackupFile.exists()) {
                mFile.delete();
                mBackupFile.renameTo(mFile);
            }
        }

        // Debugging
        if (mFile.exists() && !mFile.canRead()) {
            Log.w(TAG, "Attempt to read preferences file " + mFile + " without permission");
        }

	//用于存放从sp中读取的数据
        Map<String, Object> map = null;
        StructStat stat = null;
        Throwable thrown = null;
        try {
            stat = Os.stat(mFile.getPath());
            if (mFile.canRead()) {
                BufferedInputStream str = null;
                try {
                    str = new BufferedInputStream(
                            new FileInputStream(mFile), 16 * 1024);
		    //xml文件解析
                    map = (Map<String, Object>) XmlUtils.readMapXml(str);
                } catch (Exception e) {
                    Log.w(TAG, "Cannot read " + mFile.getAbsolutePath(), e);
                } finally {
                    IoUtils.closeQuietly(str);
                }
            }
        } catch (ErrnoException e) {
            // An errno exception means the stat failed. Treat as empty/non-existing by
            // ignoring.
        } catch (Throwable t) {
            thrown = t;
        }

        synchronized (mLock) {
            mLoaded = true;
            mThrowable = thrown;

            // It's important that we always signal waiters, even if we'll make
            // them fail with an exception. The try-finally is pretty wide, but
            // better safe than sorry.
            try {
                if (thrown == null) {
                    if (map != null) {
                        mMap = map;
                        mStatTimestamp = stat.st_mtim;//最近一次更改时间
                        mStatSize = stat.st_size;//文件大小
                    } else {
                        mMap = new HashMap<>();
                    }
                }
                // In case of a thrown exception, we retain the old map. That allows
                // any open editors to commit and store updates.
            } catch (Throwable t) {
                mThrowable = t;
            } finally {
	        //唤醒处于等待状态的线程
                mLock.notifyAll();
            }
        }
    }
}
```

从上述源码中(尤其是我加注释的地方)我们可以得到如下2点结论：   
1. 数据以xml格式保存  
2. 当实例化SP时系统会将SP的内容全部读取到内存中，这也是sp为什么只能存储少量数据的原因   

另外我们注意到系统是异步(开了新线程)加载sp到内存的，那我们在使用getxxx()、setxxx()以及edit()时并没有判断sp是否已加载到内存，那为什么从没报过错呢？难道是运气？答案是否定的，因为这些方法是阻塞等待的(详见2.2小节)，直到sp被加载后内存后调用mLock.notifyAll()来唤醒它们时，它们才会操作内存

#### 2.2 获取数据 getXXX()    
SP的获取方式笔者就不赘述了，此处以getString()方法为例进行分析    

```java
 @Override
    @Nullable
    public String getString(String key, @Nullable String defValue) {
        synchronized (mLock) {
            awaitLoadedLocked();//检测SP是否加载到内存
            String v = (String)mMap.get(key);
            return v != null ? v : defValue;
        }
    }


 @GuardedBy("mLock")
    private void awaitLoadedLocked() {
        if (!mLoaded) {
            // Raise an explicit StrictMode onReadFromDisk for this
            // thread, since the real read will be in a different
            // thread and otherwise ignored by StrictMode.
            BlockGuard.getThreadPolicy().onReadFromDisk();
        }
        while (!mLoaded) {
            try {
                mLock.wait();//SP未加载到内存时则进入等待状态
            } catch (InterruptedException unused) {
            }
        }
        if (mThrowable != null) {
            throw new IllegalStateException(mThrowable);
        }
    }
```

### 三、Editor 源码分析  
#### 3.1 实例化Editor   
我们在实例化Editor时一般都是调用SharedPreferences的edit()方法，下面我们先来看一下该源码：   

```java
 @Override
    public Editor edit() {
        // TODO: remove the need to call awaitLoadedLocked() when
        // requesting an editor.  will require some work on the
        // Editor, but then we should be able to do:
        //
        //      context.getSharedPreferences(..).edit().putString(..).apply()
        //
        // ... all without blocking.
        synchronized (mLock) {
            awaitLoadedLocked();//判断sp是否加载到内存
        }

        return new EditorImpl();//
    }

```
从上面可以看出实例化Editor时返回的是EditorImpl对象，EditorImpl是SharedPreferences的内部类，是Editor的实现类。          
**从这里我们也可以看出不建议多次调用edit()方法的原因，因未每调用一次就会new一个对象，太频繁的话会引起内存抖动，严重的话还会OOM**   

#### 3.2 插入数据 putxxx()    
此处以putString()为例   
```java
 public final class EditorImpl implements Editor {
        private final Object mEditorLock = new Object();

        @GuardedBy("mEditorLock")
        private final Map<String, Object> mModified = new HashMap<>();

        @GuardedBy("mEditorLock")
        private boolean mClear = false;

        @Override
        public Editor putString(String key, @Nullable String value) {
            synchronized (mEditorLock) {
                mModified.put(key, value);//将数据存放到mModified
                return this;
            }
        }
 }
```

从上面我们可以看到putxxx()方法只是将数据存放到mModified中，所以才会有下面的提交方法    

#### 3.2 提交数据  commit()/apply()

##### 3.2.1 commit()      
同步提交，有返回结果     

```java
 public final class EditorImpl implements Editor {
        private final Object mEditorLock = new Object();

        @GuardedBy("mEditorLock")
        private final Map<String, Object> mModified = new HashMap<>();

        @GuardedBy("mEditorLock")
        private boolean mClear = false;

       @Override
        public boolean commit() {
            long startTime = 0;

            if (DEBUG) {
                startTime = System.currentTimeMillis();
            }


            MemoryCommitResult mcr = commitToMemory();//将数据提交到内存

	    //将内存数据同步到文件
            SharedPreferencesImpl.this.enqueueDiskWrite(
                mcr, null /* sync write on this thread okay */);
            try {
                mcr.writtenToDiskLatch.await();//进入等待状态, 直到写入文件的操作完成
            } catch (InterruptedException e) {
                return false;
            } finally {
                if (DEBUG) {
                    Log.d(TAG, mFile.getName() + ":" + mcr.memoryStateGeneration
                            + " committed after " + (System.currentTimeMillis() - startTime)
                            + " ms");
                }
            }
	    //通知监听者, 并在主线程回调onSharedPreferenceChanged()方法
            notifyListeners(mcr);
            return mcr.writeToDiskResult;// 返回提交结果
        }

	//将数据提交到内存
	private MemoryCommitResult commitToMemory() {
            long memoryStateGeneration;
            List<String> keysModified = null;
            Set<OnSharedPreferenceChangeListener> listeners = null;
            Map<String, Object> mapToWriteToDisk; //此map中的所有数据都会写入磁盘

            synchronized (SharedPreferencesImpl.this.mLock) {
                // We optimistically don't make a deep copy until
                // a memory commit comes in when we're already
                // writing to disk.
		//mDiskWritesInFlight表示当前有多少个正在进行的写入磁盘的操作，如果mDiskWritesInFlight>0，表示在前面有某个写操作正把 mMap 的内容写到硬盘。此时我们不能直接修改 mMap，否则硬盘的数据的一致性会有问题（比方说，部分 key 是旧的，部分是新的）。拷贝一份 mMap 后，我们就可以安全地进行修改了。
                if (mDiskWritesInFlight > 0) {
                    // We can't modify our mMap as a currently
                    // in-flight write owns it.  Clone it before
                    // modifying it.
                    // noinspection unchecked
                    mMap = new HashMap<String, Object>(mMap);//构造一个与mMap 具有相同映射的 HashMap
                }
		//将mMap赋值给mapToWriteToDisk,即浅拷贝(其中一个数据发生变化后另一个数据也会发生相应的改变)，因此每次提交数据时sp会将所有//的数据全部更新一次，因此sp中的数据越多，sp提交数据所需时间会越久
                mapToWriteToDisk = mMap;
                mDiskWritesInFlight++;

                boolean hasListeners = mListeners.size() > 0;
                if (hasListeners) {
                    keysModified = new ArrayList<String>();
                    listeners = new HashSet<OnSharedPreferenceChangeListener>(mListeners.keySet());
                }

                synchronized (mEditorLock) {
                    boolean changesMade = false;

                    if (mClear) {//如果mClear为true,则清空mapToWriteToDisk
                        if (!mapToWriteToDisk.isEmpty()) {
                            changesMade = true;
                            mapToWriteToDisk.clear();
                        }
                        mClear = false;
                    }

                    for (Map.Entry<String, Object> e : mModified.entrySet()) {
                        String k = e.getKey();
                        Object v = e.getValue();
                        // "this" is the magic value for a removal mutation. In addition,
                        // setting a value to "null" for a given key is specified to be
                        // equivalent to calling remove on that key.
			//删除特殊数据
                        if (v == this || v == null) {
                            if (!mapToWriteToDisk.containsKey(k)) {
                                continue;
                            }
                            mapToWriteToDisk.remove(k);
                        } else {
			    //判断数据是否发生变化
                            if (mapToWriteToDisk.containsKey(k)) {
                                Object existingValue = mapToWriteToDisk.get(k);
                                if (existingValue != null && existingValue.equals(v)) {
                                    continue;//数据没发生更改则跳出此次循环
                                }
                            }
			    //更新数据
                            mapToWriteToDisk.put(k, v);
                        }
			//标记数据是否变化
                        changesMade = true;
                        if (hasListeners) {
                            keysModified.add(k);
                        }
                    }

                    mModified.clear();//清空EditorImpl中的mModified数据

                    if (changesMade) {
			//数据发生更改后将此数加加，后文写入文件时会用到
                        mCurrentMemoryStateGeneration++;
                    }

                    memoryStateGeneration = mCurrentMemoryStateGeneration;
                }
            }
            return new MemoryCommitResult(memoryStateGeneration, keysModified, listeners,
                    mapToWriteToDisk);
        }

	//启动磁盘写入任务  
	private void enqueueDiskWrite(final MemoryCommitResult mcr,
                                  final Runnable postWriteRunnable) {
	//因为由commit()方法进入，postWriteRunnable为null
        final boolean isFromSyncCommit = (postWriteRunnable == null);

        final Runnable writeToDiskRunnable = new Runnable() {
                @Override
                public void run() {
                    synchronized (mWritingToDiskLock) {
			//写入磁盘
                        writeToFile(mcr, isFromSyncCommit);
                    }
                    synchronized (mLock) {
                        mDiskWritesInFlight--;
                    }
                    if (postWriteRunnable != null) {
                        postWriteRunnable.run();
                    }
                }
            };

        // commit()方法会进入此判断
        if (isFromSyncCommit) {
            boolean wasEmpty = false;
            synchronized (mLock) {
		//commitToMemory过程会加1,则wasEmpty=true
                wasEmpty = mDiskWritesInFlight == 1;
            }
            if (wasEmpty) {
		//调用新线程进行文件写入
                writeToDiskRunnable.run();
                return;
            }
        }

	//此处不会执行，因为上面调用了return
        QueuedWork.queue(writeToDiskRunnable, !isFromSyncCommit);
    }

    //文件写入
    @GuardedBy("mWritingToDiskLock")
    private void writeToFile(MemoryCommitResult mcr, boolean isFromSyncCommit) {
        long startTime = 0;
        long existsTime = 0;
        long backupExistsTime = 0;
        long outputStreamCreateTime = 0;
        long writeTime = 0;
        long fsyncTime = 0;
        long setPermTime = 0;
        long fstatTime = 0;
        long deleteTime = 0;

        if (DEBUG) {
            startTime = System.currentTimeMillis();
        }

        boolean fileExists = mFile.exists();

        if (DEBUG) {
            existsTime = System.currentTimeMillis();

            // Might not be set, hence init them to a default value
            backupExistsTime = existsTime;
        }

        // Rename the current file so it may be used as a backup during the next read
        if (fileExists) {
            boolean needsWrite = false;

            // Only need to write if the disk state is older than this commit
	    //判断数据是否需要写入
            if (mDiskStateGeneration < mcr.memoryStateGeneration) {
                if (isFromSyncCommit) {
                    needsWrite = true;
                } else {
                    synchronized (mLock) {
                        // No need to persist intermediate states. Just wait for the latest state to
                        // be persisted.
			//apply()会执行此处代码，3.2.2小节会介绍apply()方法会延迟100毫秒后才执行写入操作，延迟的目的是为了避免不必要的写//入操作，例：如果我们频繁调用apply()且间隔时间小于100毫秒，那么只有最后一次的apply()操作才满足此处的条件，才会将//needsWrite设为true
                        if (mCurrentMemoryStateGeneration == mcr.memoryStateGeneration) {
                            needsWrite = true;
                        }
                    }
                }
            }

            if (!needsWrite) {
                mcr.setDiskWriteResult(false, true);
                return;
            }

            boolean backupFileExists = mBackupFile.exists();

            if (DEBUG) {
                backupExistsTime = System.currentTimeMillis();
            }

            if (!backupFileExists) {
                if (!mFile.renameTo(mBackupFile)) {
                    Log.e(TAG, "Couldn't rename file " + mFile
                          + " to backup file " + mBackupFile);
                    mcr.setDiskWriteResult(false, false);
                    return;
                }
            } else {
                mFile.delete();
            }
        }

        // Attempt to write the file, delete the backup and return true as atomically as
        // possible.  If any exception occurs, delete the new file; next time we will restore
        // from the backup.
        try {
            FileOutputStream str = createFileOutputStream(mFile);

            if (DEBUG) {
                outputStreamCreateTime = System.currentTimeMillis();
            }

            if (str == null) {
                mcr.setDiskWriteResult(false, false);
                return;
            }
	    //写入数据
            XmlUtils.writeMapXml(mcr.mapToWriteToDisk, str);

            writeTime = System.currentTimeMillis();

            FileUtils.sync(str);

            fsyncTime = System.currentTimeMillis();

            str.close();
            ContextImpl.setFilePermissionsFromMode(mFile.getPath(), mMode, 0);

            if (DEBUG) {
                setPermTime = System.currentTimeMillis();
            }

            try {
                final StructStat stat = Os.stat(mFile.getPath());
                synchronized (mLock) {
                    mStatTimestamp = stat.st_mtim;
                    mStatSize = stat.st_size;
                }
            } catch (ErrnoException e) {
                // Do nothing
            }

            if (DEBUG) {
                fstatTime = System.currentTimeMillis();
            }

            // Writing was successful, delete the backup file if there is one.
            mBackupFile.delete();

            if (DEBUG) {
                deleteTime = System.currentTimeMillis();
            }

            mDiskStateGeneration = mcr.memoryStateGeneration;
	    //返回写入成功并唤醒等待线程
            mcr.setDiskWriteResult(true, true);

            if (DEBUG) {
                Log.d(TAG, "write: " + (existsTime - startTime) + "/"
                        + (backupExistsTime - startTime) + "/"
                        + (outputStreamCreateTime - startTime) + "/"
                        + (writeTime - startTime) + "/"
                        + (fsyncTime - startTime) + "/"
                        + (setPermTime - startTime) + "/"
                        + (fstatTime - startTime) + "/"
                        + (deleteTime - startTime));
            }

            long fsyncDuration = fsyncTime - writeTime;
            mSyncTimes.add((int) fsyncDuration);
            mNumSync++;

            if (DEBUG || mNumSync % 1024 == 0 || fsyncDuration > MAX_FSYNC_DURATION_MILLIS) {
                mSyncTimes.log(TAG, "Time required to fsync " + mFile + ": ");
            }

            return;
        } catch (XmlPullParserException e) {
            Log.w(TAG, "writeToFile: Got exception:", e);
        } catch (IOException e) {
            Log.w(TAG, "writeToFile: Got exception:", e);
        }

        // Clean up an unsuccessfully written file
        if (mFile.exists()) {
            if (!mFile.delete()) {
                Log.e(TAG, "Couldn't clean up partially-written file " + mFile);
            }
        }
	//返回写入失败并唤醒等待线程
        mcr.setDiskWriteResult(false, false);
    }
 }
```

##### 3.2.2 apply()
异步提交数据，没有返回结果   
```java
 public final class EditorImpl implements Editor {
        private final Object mEditorLock = new Object();

        @GuardedBy("mEditorLock")
        private final Map<String, Object> mModified = new HashMap<>();

        @GuardedBy("mEditorLock")
        private boolean mClear = false;

	@Override
        public void apply() {
            final long startTime = System.currentTimeMillis();
	     //提交到内存   详见3.2.1
            final MemoryCommitResult mcr = commitToMemory();
            final Runnable awaitCommit = new Runnable() {
                    @Override
                    public void run() {
                        try {
                            mcr.writtenToDiskLatch.await();
                        } catch (InterruptedException ignored) {
                        }

                        if (DEBUG && mcr.wasWritten) {
                            Log.d(TAG, mFile.getName() + ":" + mcr.memoryStateGeneration
                                    + " applied after " + (System.currentTimeMillis() - startTime)
                                    + " ms");
                        }
                    }
                };

	    //将awaitCommit加入到排队队列中
            QueuedWork.addFinisher(awaitCommit);

            Runnable postWriteRunnable = new Runnable() {
                    @Override
                    public void run() {
                        awaitCommit.run();
			//awaitCommit执行完成后被移除
                        QueuedWork.removeFinisher(awaitCommit);
                    }
                };
	    //异步写入磁盘
            SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable);

            // Okay to notify the listeners before it's hit disk
            // because the listeners should always get the same
            // SharedPreferences instance back, which has the
            // changes reflected in memory.
            notifyListeners(mcr);
        }
	
	//启动磁盘写入任务  
	private void enqueueDiskWrite(final MemoryCommitResult mcr,
                                  final Runnable postWriteRunnable) {
	//因为由apply()方法进入，postWriteRunnable不为null
        final boolean isFromSyncCommit = (postWriteRunnable == null);

        final Runnable writeToDiskRunnable = new Runnable() {
                @Override
                public void run() {
                    synchronized (mWritingToDiskLock) {
			//写入磁盘
                        writeToFile(mcr, isFromSyncCommit);
                    }
                    synchronized (mLock) {
                        mDiskWritesInFlight--;
                    }
                    if (postWriteRunnable != null) {
                        postWriteRunnable.run();
                    }
                }
            };

        // apply()方法不会进入此判断
        if (isFromSyncCommit) {
            boolean wasEmpty = false;
            synchronized (mLock) {
                wasEmpty = mDiskWritesInFlight == 1;
            }
            if (wasEmpty) {
                writeToDiskRunnable.run();
                return;
            }
        }

	//将写入磁盘的任务交给工作队列负责
        QueuedWork.queue(writeToDiskRunnable, !isFromSyncCommit);
    }
}
```
QueuedWork 是一个全局的工作队列，addFinisher 添加进去的 runnable 不会立即执行，仅仅是放到一个链表里。当某些操作需要等待 QueuedWork 所有工作执行完毕时，就调用 QueuedWork.waitToFinish()，在这个方法里面会取出之前添加的所有 addFinisher 的任务，一个一个执行。

在 awaitCommit 里面，我们调用了 mcr.writtenToDiskLatch.await() 来等待数据写入硬盘，所以这里的把 awaitCommit 放到 QueuedWork 里，就提供了一种机制，让外界等待文件的写入操作。读者可以到 ActivityThread 中搜一下 QueuedWork.waitToFinish，会发现在 activity/service stop 的时候，都会执行这个操作，从而保证在应用退出前 SP 已经写入硬盘。

QueuedWork相关源码：  
```java
public class QueuedWork {
    private static final long DELAY = 100;
    @GuardedBy("sLock")
    private static boolean sCanDelay = true;

    public static void queue(Runnable work, boolean shouldDelay) {
        Handler handler = getHandler();//getHandler返回的是一个HandlerThread对象

        synchronized (sLock) {
            sWork.add(work);
	    //因为是由apply()方法进入，因此shouldDelay为true
            if (shouldDelay && sCanDelay) {
		//默认延迟100毫秒后处理写入操作
                handler.sendEmptyMessageDelayed(QueuedWorkHandler.MSG_RUN, DELAY);
            } else {
                handler.sendEmptyMessage(QueuedWorkHandler.MSG_RUN);
            }
        }
    }

    private static Handler getHandler() {
        synchronized (sLock) {
            if (sHandler == null) {
                HandlerThread handlerThread = new HandlerThread("queued-work-looper",
                        Process.THREAD_PRIORITY_FOREGROUND);
                handlerThread.start();

                sHandler = new QueuedWorkHandler(handlerThread.getLooper());
            }
            return sHandler;
        }
    }
}
```



**友情提示：代码中的注释均为重点**

### 四、总结    
至此，SP的源码已阅读完毕，我想前文中所提示的3个疑问应该都已找到答案，我们在这里再进行一些简单的总结：    

1、 SP只能存储少量数据     
2、不要频繁调用edit()方法/不要频繁实例化Editor     
3、commit()运行在主线程，用于同步提交数据，可以拿到返回结果；     
apply()运行在子线程，用于异步提交数据，不会返回数据提交结果。   
另外数据每次提交时都会将之前所有的数据重新写入，因此一定不要存储大量数据      

4、对于一些需要频繁写入的场景推荐使用Tencent的MMKV组件，它借助mmap提高了IO效率并且可以实现增量更新操作            
<a href="https://github.com/Tencent/MMKV/blob/master/readme_cn.md" target="_blank">点此查看MMVK</a>
