---
layout:     post
title:      Rxjava2中CompositeDisposable解除订阅注意事项
subtitle:   
date:       2020-02-21
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

在Rxjava2中可以使用CompositeDisposable的dispose()和clear()2个方法批量解除订阅，但是这2个方法却有很大的区别，如果不知道其中区别而你一旦又用错了地方，那你会很难找到问题的原因。    

让我们先来看一下CompositeDisposable的相关源码：    
```java
/**
 * A disposable container that can hold onto multiple other disposables and
 * offers O(1) add and removal complexity.
 * 一个可以容纳多个disposable的容器，添加和删除复杂度为O(1)
 */
public final class CompositeDisposable implements Disposable, DisposableContainer {

    OpenHashSet<Disposable> resources;

    volatile boolean disposed;

    @Override
    public void dispose() {
        if (disposed) {
            return;
        }
        OpenHashSet<Disposable> set;
        synchronized (this) {
            if (disposed) {
                return;
            }
	    //关注点
            disposed = true;
            set = resources;
            resources = null;
        }

        dispose(set);
    }

  
    @Override
    public boolean add(@NonNull Disposable d) {
        ObjectHelper.requireNonNull(d, "d is null");
	//如果CompositeDisposable处于disposed状态,那么所有加进来的disposable都会被自动取消
        if (!disposed) {
            synchronized (this) {
                if (!disposed) {
                    OpenHashSet<Disposable> set = resources;
                    if (set == null) {
                        set = new OpenHashSet<Disposable>();
                        resources = set;
                    }
                    set.add(d);
                    return true;
                }
            }
        }
        d.dispose();
        return false;
    }

   
    /**
     * Atomically clears the container, then disposes all the previously contained Disposables.
     * 自动清除容器，释放容器中包含的所有的Disposable
     */
    public void clear() {
        if (disposed) {
            return;
        }
        OpenHashSet<Disposable> set;
        synchronized (this) {
            if (disposed) {
                return;
            }

            set = resources;
            resources = null;
        }

        dispose(set);
    }

    
    /**
    *  dispose()和clear()都是调用此方法释放Disposable
     * Dispose the contents of the OpenHashSet by suppressing non-fatal
     * Throwables till the end.
     * @param set the OpenHashSet to dispose elements of
     */
    void dispose(OpenHashSet<Disposable> set) {
        if (set == null) {
            return;
        }
        List<Throwable> errors = null;
        Object[] array = set.keys();
	//遍历容器释放Disposable
        for (Object o : array) {
            if (o instanceof Disposable) {
                try {
                    ((Disposable) o).dispose();
                } catch (Throwable ex) {
                    Exceptions.throwIfFatal(ex);
                    if (errors == null) {
                        errors = new ArrayList<Throwable>();
                    }
                    errors.add(ex);
                }
            }
        }
        if (errors != null) {
            if (errors.size() == 1) {
                throw ExceptionHelper.wrapOrThrow(errors.get(0));
            }
            throw new CompositeException(errors);
        }
    }
}
```
源码中重点部分我已添加注释，从源码中可以看出dispose()和clear()都能解除订阅，但是dispose在解除订阅的时候将CompositeDisposable容器的状态更改为dispose状态，如果你后面继续往这个容器中添加Disposable的话，那么你新加入的Disposable将无法完成指定任务，会被直接取消。

如果你使用dispose()取消订阅的话，你一定要特别注意CompositeDisposable的生命周期，而且在新加入Disposable时应该判断一下CompositeDisposable的状态，以免影响你的任务。


