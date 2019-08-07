# Guava future 源码解析
## 1、前言
&emsp;&emsp; 在前两篇文章中简单阐述了Java Future 和Guava ListenableFuture及其相关的应用。我们发现Guava ListenableFuture提供了比Java Future更加强大的功能，而在Google Guava并发包中，某些情况下，Futures这个类起到了不可或缺的作用，而ListenableFuture实现非阻塞的原理是其提供了回调机制(机制)，下面将阐述其中的回调机制的实现，主要对Futures的addCallback方法源码进行详细的剖析。

## 2、Guava Futures简介
&emsp;&emsp;Google Guava框架的 com.google.common.util.concurrent包是并发相关的包，它是对JDK自带concurrent包中Future和线程池相关类的扩展，从而衍生出一些新类，并提供了更为广泛的功能。在项目中常用的该包中类如下所示：

&emsp;&emsp;**ListenableFuture**：该接口扩展了Future接口，增加了addListener方法，该方法在给定的excutor上注册一个监听器，当计算完成时会马上调用该监听器。不能够确保监听器执行的顺序，但可以在计算完成时确保马上被调用。

&emsp;&emsp;**FutureCallback**：该接口提供了OnSuccess和onFailure方法。获取异步计算的结果并回调。 

&emsp;&emsp;**MoreExecutors**：该类是final类型的工具类，提供了很多静态方法。例如listeningDecorator方法初始化ListeningExecutorService方法，使用此实例submit方法即可初始化ListenableFuture对象。 

&emsp;&emsp; **ListenableFutureTask**：该类是一个适配器，可以将其它Future适配成ListenableFuture。

&emsp;&emsp; **ListeningExecutorService**：该类是对ExecutorService的扩展，重写了ExecutorService类中的submit方法，并返回ListenableFuture对象。 

&emsp;&emsp; **JdkFutureAdapters**：该类扩展了FutureTask类并实现ListenableFuture接口，增加了addListener方法。 

&emsp;&emsp; **Futures**：该类提供和很多实用的静态方法以供使用。

## 3、Futures.addCallback方法源码剖析
&emsp;&emsp; 下面将模拟异步发送请求，并对请求结果进行(回调)监听。这里使用Spring框架提供的AsyncRestTemplate，来发送http请求，并获取一个org.springframework.util.concurrent.ListenableFuture对象，此时的对象是spring框架中的ListenableFuture对象。由于org.springframework.util.concurrent包中只提供了最基本的监听功能，没有其它额外功能，这里将<font color=red>**其转化成Guava中的ListenableFuture，用到了JdkFutureAdapters这个适配器类。**</font>(以下源码来自guava-18.0.jar)

```
AsyncRestTemplate tp = new AsyncRestTemplate();
org.springframework.util.concurrent.ListenableFuture<ResponseEntity<Object>> response = tp
        .getForEntity("http://blog.csdn.net/pistolove", Object.class);
ListenableFuture<ResponseEntity<Object>> listenInPoolThread = JdkFutureAdapters.listenInPoolThread(response);
Futures.addCallback(listenInPoolThread, new FutureCallback<Object>() {
    @Override
    public void onSuccess(Object result) {
        System.err.println(result.getClass());
        System.err.printf("success", result);
    }

    @Override
    public void onFailure(Throwable t) {
        System.out.printf("failure");
    }
});
```

---

&emsp;&emsp;Futures的addCallback方法通过传入ListenableFuture和FutureCallback（一般情况FutureCallback实现为内部类）来实现回调机制。
```
//com.google.common.util.concurrent.Futures
  public static <V> void addCallback(ListenableFuture<V> future,
      FutureCallback<? super V> callback) {
    addCallback(future, callback, directExecutor());
  }
 ```

&emsp;&emsp;在addCallback方法中，我们发现多了一个 directExecutor()方法，这里的 directExecutor()方法返回的是一个枚举类型的线程池，这样做的目的是提高性能，而线程池中的execute方法实质执行的是所的传入参数Runnable 的run方法，可以把这里的线程池看作一个”架子”。

//创建一个单实例的线程 接口需要显著的性能开销 提高性能
```
  public static Executor directExecutor() {
    return DirectExecutor.INSTANCE;
  }

  /** See {@link #directExecutor} for behavioral notes. */
  private enum DirectExecutor implements Executor {
    INSTANCE;
    @Override public void execute(Runnable command) {
      command.run();
    }
  }
```
&emsp;&emsp; 在具体的addCallback方法中，首先判断FutureCallback是否为空，然后创建一个线程，这个线程的run方法中会获取到一个value值，这里的value值即为http请求的结果，然后将value值传入FutureCallback的onSuccess方法，然后我们就可以在onSuccess方法中执行业务逻辑了。这个线程是如何执行的呢？继续往下看，发现调用了ListenableFuture的addListener方法，将刚才创建的线程和上一步创建的枚举线程池传入。

//增加回调
```
  public static <V> void addCallback(final ListenableFuture<V> future,
      final FutureCallback<? super V> callback, Executor executor) {
    Preconditions.checkNotNull(callback);

    //每一个future进来都会创建一个独立的线程运行
    Runnable callbackListener = new Runnable() {
      @Override
      public void run() {
        final V value;
        try {
          // TODO(user): (Before Guava release), validate that this
          // is the thing for IE.
          //这里是真正阻塞的地方，直到获取到请求结果
          value = getUninterruptibly(future);
        } catch (ExecutionException e) {
          callback.onFailure(e.getCause());
          return;
        } catch (RuntimeException e) {
          callback.onFailure(e);
          return;
        } catch (Error e) {
          callback.onFailure(e);
          return;
        }
        //调用callback的onSuccess方法返回结果
        callback.onSuccess(value);
      }
    };

    //增加监听，其中executor只提供了一个架子的线程池
    future.addListener(callbackListener, executor);
  }
 ```

&emsp;&emsp; 在addListener方法中，将待执行的任务和枚举型线程池加入ExecutionList中，ExecutionList的本质是一个链表，将这些任务链接起来。具体可参考下方代码注释。
```

      @Override
    public void addListener(Runnable listener, Executor exec) {
    //将监听任务和线程池加入到执行列表中
      executionList.add(listener, exec);

    //This allows us to only start up a thread waiting on the delegate future when the first listener is added.
    //When a listener is first added, we run a task that will wait for the delegate to finish, and when it is done will run the listeners.

      //这允许我们启动一个线程来等待future当第一个监听器被加入的时候
      //当第一个监听器被加入，我们将启动一个任务等待future完成，一旦当前的future完成后将会执行监听器

      //判断是否有监听器加入
      if (hasListeners.compareAndSet(false, true)) {
      //如果当前的future完成则立即执行监听列表中的监听器,执行完成后返回
        if (delegate.isDone()) {
          // If the delegate is already done, run the execution list
          // immediately on the current thread.
          //执行监听列表中的监听任务
          executionList.execute();
          return;
        }

        //如果当前的future没有完成，则启动线程池执行其中的任务，阻塞等待直到有一个future完成，然后执行监听器列表中的监听器
        adapterExecutor.execute(new Runnable() {
          @Override
          public void run() {
            try {
              /*
               * Threads from our private pool are never interrupted. Threads
               * from a user-supplied executor might be, but... what can we do?
               * This is another reason to return a proper ListenableFuture
               * instead of using listenInPoolThread.
               */
              getUninterruptibly(delegate);
            } catch (Error e) {
              throw e;
            } catch (Throwable e) {
              // ExecutionException / CancellationException / RuntimeException
              // The task is done, run the listeners.
            }

            //执行链表中的任务
            executionList.execute();
          }
        });
      }
    }
  }
  ```

&emsp;&emsp; 在ExecutionList的add方法中，判断是否执行完成，如果没有执行完成，则放入待执行的链表中并返回，否则调用executeListener方法执行任务，在executeListener方法中，我们发现执行的是线程池的execute方法，而execute方法实质的是调用了任务线程的run方法，这样最终会调用OnSuccess方法获取到执行结果。
```
  //将任务放入ExecutionList中
    public void add(Runnable runnable, Executor executor) {
    // Fail fast on a null.  We throw NPE here because the contract of
    // Executor states that it throws NPE on null listener, so we propagate
    // that contract up into the add method as well.
    Preconditions.checkNotNull(runnable, "Runnable was null.");
    Preconditions.checkNotNull(executor, "Executor was null.");

    // Lock while we check state.  We must maintain the lock while adding the
    // new pair so that another thread can't run the list out from under us.
    // We only add to the list if we have not yet started execution.
    //判断是否执行完成，如果没有执行完成，则放入待执行的链表中
    synchronized (this) {
      if (!executed) {
        runnables = new RunnableExecutorPair(runnable, executor, runnables);
        return;
      }
    }
    // Execute the runnable immediately. Because of scheduling this may end up
    // getting called before some of the previously added runnables, but we're
    // OK with that.  If we want to change the contract to guarantee ordering
    // among runnables we'd have to modify the logic here to allow it.
    //执行监听
    executeListener(runnable, executor);
  }
 ```
 
&emsp;&emsp; execute方法是执行任务链表中的任务，由于先加入的任务会依次排列在链表的末尾，所以需要将链表翻转。然后从链表头开始依次取出任务执行并放入枚举线程池中执行。
```
    //执行监听链表中的任务
    public void execute() {
    // Lock while we update our state so the add method above will finish adding
    // any listeners before we start to run them.
    //创建临时变量保存列表，并将成员变量置空让垃圾回收
    RunnableExecutorPair list;
    synchronized (this) {
      if (executed) {
        return;
      }
      executed = true;
      list = runnables;
      runnables = null;  // allow GC to free listeners even if this stays around for a while.
    }


    // If we succeeded then list holds all the runnables we to execute.  The pairs in the stack are
    // in the opposite order from how they were added so we need to reverse the list to fulfill our
    // contract.
    // This is somewhat annoying, but turns out to be very fast in practice.  Alternatively, we 
    // could drop the contract on the method that enforces this queue like behavior since depending
    // on it is likely to be a bug anyway.

    // N.B. All writes to the list and the next pointers must have happened before the above 
    // synchronized block, so we can iterate the list without the lock held here.

    //因为先加入的监听任务会在连边的末尾，所以需要将链表翻转
    RunnableExecutorPair reversedList = null;
    while (list != null) {
      RunnableExecutorPair tmp = list;
      list = list.next;
      tmp.next = reversedList;
      reversedList = tmp;
    }

    //从链表头中依次取出监听任务执行
    while (reversedList != null) {
      executeListener(reversedList.runnable, reversedList.executor);
      reversedList = reversedList.next;
    }
  }
  ```
&emsp;&emsp; 在上文中，可以发现每当对一个ListenerFuture增加回调时，都会创建一个线程，而这个线程的run方法中会获取一个value值，这个value值就是通过下面的getUninterruptibly方法获取到的，我们可以发现在方法中调用了while进行阻塞，一直等到future获取到结果，即发送的http请求获取到数据后才会终止并返回。可以看出，回调机制将获取结果中的阻塞分散开来，即使现在有100个线程在并发地发送http请求，那么也只是创建了100个独立的线程并行阻塞，那么运行的总时间则会是这100个线程中最长的时间，而不是100个线程的时间相加，这样就实现了异步非阻塞机制。
```
  //用while阻塞直到获取到结果
    public static <V> V getUninterruptibly(Future<V> future)
      throws ExecutionException {
    boolean interrupted = false;
    try {
      while (true) {
        try {
          return future.get();
        } catch (InterruptedException e) {
          interrupted = true;
        }
      }
    } finally {
      if (interrupted) {
        Thread.currentThread().interrupt();
      }
    }
  }
```
&emsp;&emsp; 这里实质上执行了线程的run方法，并进行阻塞。

 //执行监听器，调用线程池的execute方法，这里线程池并没有提供额外的功能，只提供了执行架子，实际上执行的是监听任务runnable的run方法
  //而在监听任务的run方法中，会阻塞获取请求结果，请求完成后回调，已达到异步执行的效果
  ```
    private static void executeListener(Runnable runnable, Executor executor) {
    try {
      executor.execute(runnable);
    } catch (RuntimeException e) {
      // Log it and keep going, bad runnable and/or executor.  Don't
      // punish the other runnables if we're given a bad one.  We only
      // catch RuntimeException because we want Errors to propagate up.
      log.log(Level.SEVERE, "RuntimeException while executing runnable "
          + runnable + " with executor " + executor, e);
    }
  }
```
&emsp;&emsp; 使用JdkFutureAdaoter适配Spring中的ListenableFuture达到异步调用的结果。在future.get方法中到底阻塞在什么地方呢？ 
通过调试发现最后调用的是BasicFuture中的阻塞方法。详情见下方源码和中文注释，这里不累赘。
```
 FutureAdapter:
    //这里的get方法会调用BasicFuture中的get方法进行阻塞，直到获取到结果
    @Override
    public T get() throws InterruptedException, ExecutionException {
        return adaptInternal(this.adaptee.get());
    }
```
    BasicFuture:
    //在这里会判断当前future是否执行完成，如果没有完成则会等待，一旦执行完成则返回结果。
```
   public synchronized T get() throws InterruptedException, ExecutionException {
        while (!this.completed) {
            wait();
        }
        return getResult();
    }
```

    FutureAdapter:
    //这里通过判断状态是否success，如果success则返回成功，如果new, 则阻塞等待结果直到返回，然后改变状态。
    
```
    @SuppressWarnings("unchecked")
    final T adaptInternal(S adapteeResult) throws ExecutionException {
        synchronized (this.mutex) {
            switch (this.state) {
                case SUCCESS:
                    return (T) this.result;
                case FAILURE:
                    throw (ExecutionException) this.result;
                case NEW:
                    try {
                        T adapted = adapt(adapteeResult);
                        this.result = adapted;
                        this.state = State.SUCCESS;
                        return adapted;
                    }
                    catch (ExecutionException ex) {
                        this.result = ex;
                        this.state = State.FAILURE;
                        throw ex;
                    }
                    catch (Throwable ex) {
                        ExecutionException execEx = new ExecutionException(ex);
                        this.result = execEx;
                        this.state = State.FAILURE;
                        throw execEx;
                    }
                default:
                    throw new IllegalStateException();
            }
        }
    }
```
  //这个方法判断
```
    public boolean completed(final T result) {
        synchronized(this) {
            if (this.completed) {
                return false;
            }
            this.completed = true;
            this.result = result;
            notifyAll();
        }
        if (this.callback != null) {
            this.callback.completed(result);
        }
        return true;
    }
```
    
## 4、总结
&emsp;&emsp;本文主要剖析了Futures.callback方法的源码，我们只需要一个ListenableFuture的实例，就可以使用该方法来实现回调机制。假设在我们的主线程中，有n个子方法要发送http请求，这时，我们可以创建n个ListenableFuture，对这n个ListenableFuture增加监听，这n个请求就是异步且非阻塞的，这样不旦主线程不会阻塞，而且会大大减少总的响应时间。那Futures.callback是如何实现并发的呢？通过源码，我们发现，对于每一个ListenableFuture，都会创建一个独立的线程对其进行监听，也就是这n个ListenableFuture对应着n个相互独立的线程，而在每一个独立的线程中会各自调用Future.get方法阻塞。

&emsp;&emsp; 本文只是简单地讲解了Futures.callback方法，并未作深入的探讨和研究，希望本文对你有所帮助
