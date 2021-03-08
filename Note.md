- 链接：
    - CompletableFuture：
    [CompletableFuture](https://www.baeldung.com/java-completablefuture)
    [CompletableFuture](https://blog.csdn.net/david_lua/article/details/102258484)
    [CompletableFuture](https://www.cnblogs.com/swiftma/p/7424185.html)
    [CompletableFuture](https://blog.csdn.net/u013591094/article/details/110682973?spm=1001.2014.3001.5501)
    - Stream:
    [stream](https://blog.csdn.net/y_k_y/article/details/84633001?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-6.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-6.control)
    [stream](https://blog.csdn.net/hu10131013/article/details/108368809?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-7.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-7.control)
    [stream](https://www.cnblogs.com/lbys/p/14043917.html)
    [stream](https://www.jianshu.com/p/dd5121c8fa89?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)
    - mock:
    [mock](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)
    [mock](https://github.com/hehonghui/mockito-doc-zh#12)
[类加载过程](https://www.cnblogs.com/gxyandwmm/p/9650144.html)

> 3.4

-CompletableFuture: 
  - 静态方法构建 异步任务:
    - supplyAsync: 可以获取任务结果
      1. static &lt;U&gt; CompletableFuture&lt;U&gt; supplyAsync(Supplier&lt;U&gt; supplier) `不建议使用`
      2. static &lt;U&gt; CompletableFuture&lt;U&gt; supplyAsync(Supplier&lt;U&gt; supplier, Executor executor)
      ps: `U为返回值的类型,带Excutor参数表示使用自定义的线程池执行任务`,否则使用默认提供的ForkJoin.commonPool执行异步任务
    - runAsync: 无法获取任务结果
      1. static CompletableFuture<?Void> runAsync(Runnable runnable) `不建议使用`
      2. static CompletableFuture<?Void> runAsync(Runnable runnable, Executor executor)
      ps: `带Excutor参数表示使用自定义的线程池执行任务`,否则使用默认提供的ForkJoin.commonPool执行异步任务
    - 创建一个已完成的任务,并且设置结果为value
      1. static &lt;U&gt; CompletableFuture&lt;U&gt; completedFuture(U value)
  > 注意:<br/>
  > 1. 由于ForkJoin.commonPool线程池,采用的是 当前环境cpu核心数-1 确定的线程池中线程数量,如果存在I/O型任务,容易造成任务阻塞,所以: 在使用
  CompletableFuture的时候,建议根据任务类型来设置线程池中线程数量,即使用自定义Executor<br/>
  > 2. 如何根据任务类型设置线程数量: cpu密集型任务: 线程数量=cpu核心数 ; I/O密集型任务: 线程数量=cpu核心数*2`(也可以再 +1)`<br/>
  - 任务结果处理:
    - 判断任务是否完成:
      - boolean `isDone()`
    - 获取任务结果:
      - T `get()` 
        - 任务正常结束: 返回值就是结果;任务异常结束,会抛出对应的异常
        - 可能抛出的异常:
          - `CancellationException`: 任务被取消
          - `ExecutionException`: 任务在执行过程中抛出了异常
          - `InterruptedException`: 执行任务的线程在waiting过程中被中断
      - T `get(long timeout, TimeUnit unit)`
        - 基本与get相同,只是多了一个超时异常(TimeOutException)
      - T `join()`
        - 可能获取到正常的任务结果,也可能获取到任务执行中抛出的异常,任务执行过程中的异常会被封装成 CompletionException
      - T `getNow(T valueIfAbsent)`
        - 如果任务已经执行完成,获取任务的结果或者任务抛出的异常;如果任务没有执行完成,就返回 `valueIfAbsent`参数
        - 抛出的异常与 `join()`一样
    - 任务处理:
    > `任务`指的就是调用下面这些个方法的 CompletableFuture对象
    > `取消cancel`在CompletableFuture中,被当成是异常的一种特殊形式
      - boolean `complete(T value)`
        - 尝试立即完成任务,并将任务结果设置为value,(get等方法获取到的结果就是value);
        - 如果因为调用这个方法使任务状态转换成`完成`就返回true,否则返回false
      - boolean `completeExceptionally(Throwable ex)`
        - 尝试立即完成任务,并将任务状态设置为异常完成,设置异常为ex(get等方法就会抛出 异常)
        - 如果因为调用这个方法使任务状态转换成`完成`就返回true,否则返回false
      - boolean `cancel(boolean mayInterruptIfRunning)`
        - 尝试立即取消任务,成功取消就返回true,否则返回false
        - 任务如果在未完成时被取消,任务的状态就变成了 异常结束,get()等方法会抛出CancellationException
        - 与当前任务相关联的任务,也会因为这个CancellationException而异常结束
      - boolean `isCancelled()`
        - 判断任务是否在正常结束之前,被取消掉了就返回true;否则,返回false
      -  boolean `isCompletedExceptionally()`
        - 判断任务是不是异常完成
        - 不仅仅包括以下情况: cancel(),completedExceptionally()等
      - void `obtrudeValue(T value)`
        - `这个方法只是为了错误恢复场景而设置的`
        - 强制设置任务的结果,无论任务是否完成
        - 这个方法可能造成与之关联的任务的结果,也被设置为value.
      - void `obtrudeException(Throwable ex)`
        - `这个方法只是为了错误恢复场景而设置的`
        - 强制设置任务异常结束,无论任务是否完成
      - int `getNumberOfDependents()`
        - 获取有多少任务是关联着当前任务的
    - 任务执行顺序编排:
        - 串行关系: `前置任务`指:调用这些个方法的CompletableFuture,`新任务`指 参数action
          - `thenRun`: 不关心前置任务的结果,只是需要等待前置任务执行完成才开始执行 新任务
            - CompletableFuture<Void> thenRun(Runnable action)
            - CompletableFuture<Void> thenRunAsync(Runnable action)
            - CompletableFuture<Void> thenRunAsync(Runnable action, Executor executor)
          - `thenAccept`: 等待前置任务执行结束之后,接收前置任务的返回值作为action的参数,执行 `新任务`,`新任务无返回值`
            - CompletableFuture<Void> thenAccept(Consumer<? super T> action)
            - CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action)
            - CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action, Executor executor)
          - `thenApply`: 等待前置任务执行结束之后,接收前置任务的返回值作为action的参数,执行 `新任务`,`新任务有返回值`
            - &lt;U&gt; CompletableFuture&lt;U&gt; thenApply(Function<? super T,? extends U> fn)
            - &lt;U&gt; CompletableFuture&lt;U&gt; thenApplyAsync(Function<? super T,? extends U> fn)
            - &lt;U&gt; CompletableFuture&lt;U&gt; thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)
          - `thenCompose`: 等待前置任务执行结束之后,接收前置任务的返回值作为action的参数,执行 `新任务`,`新任务有返回值,且返回值类型为CompletionStage或其子类`
            - public <U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn)
            - public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn)
            - public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn, Executor executor)
        - 并行关系: 前置任务都并行?都异步?
          - 前置任务都执行完才执行新任务
              - `runAfterBoth`:
                - public CompletableFuture<Void> runAfterBoth(CompletionStage<?> other, Runnable action)
                - public CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other, Runnable action)
                - public CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other, Runnable action, Executor executor)
              - `thenAcceptBoth`: 
                - public <U> CompletableFuture<Void> thenAcceptBoth(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action)
                - public <U> CompletableFuture<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action)
                - public <U> CompletableFuture<Void> thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T, ? super U> action, Executor executor)
              - `thenCombine`:
                - public <U,V> CompletableFuture<V> thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
                - public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
                - public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn, Executor executor)        
         - 前置任务其中一个完成执行,就执行新任务
              - `runAfterEither`
              - `acceptEither`
              - `applyToEither`
        - 异常处理:
              - `exceptionally`: 等价于catch
              - `handle`: 等价于finally
              - `whenComplete`: 等价于finally
        - 多任务的简单组合:
              - `allOf`:
              - `anyOf`:

> 3.5

- 线程池:
    允许核心线程空闲太久被关闭的操作: (核心线程默认是不允许被关闭的)可以通过 void allowCoreThreadTimeOut方法去开启核心线程的空闲太久关闭
    - void allowCoreThreadTimeOut(Boolean value)
        - 设置是否允许核心线程空闲太久关闭
        - 空闲太久关闭策略: 空闲时间超过keepAliveTime参数就算空闲太久,然后关闭超过时间的线程
        - value : 是否允许核心线程空闲太久关闭,true允许,false不允许
    - boolean allowCoreThreadTimeOut()
        - 查询线程池是否允许核心线程空闲太久被关闭
        - 返回值: 
           - true: 线程池允许核心线程在空闲时间超过keepAlivetime参数的时候,关闭核心线程
           - false: 线程池不允许核心线程空闲太久被关闭
- 多态:
    父类中调用的方法,如果被子类重写了,那么其调用的是子类的方法.因为父类中 setXXX() 等价于 this.setXXX(),所以满足多态的条件父类引用指向子类对象,重写.
    ```java
    /**
    这个例子中有两处都体现的前面的那个规则
    第一处,
        1. new B()的时候,由于子类构造器总是会先去调用父类构造器,先去初始化父类,所以在new B的时候,会先去new A,
        2. 然后调用setI(即this.setI()),然后调用子类B的setI(),将B.I = 60.
        3. b构造器执行完毕后,根据类加载的顺序将 B.I = 5
    第二处:
        a.fun
    */
    public ExtendsTest{
        public static void main(String[] args){
            A a = new B();
            a.fun();
        }
    }
    class A{
        int i = 7;
        A(){
            setI(20);
            System.out.println(i);
        }
        void setI(int i){
            this.i = 2*i;
        }
        void fun(){
            func();
            System.out.pringtln("这是父类方法 fun");
        }
        void func(){
            System.out.println("这是父类方法 func");
        }
    }
    
    class B{
        int i = 5;
        B(){
          System.out.println(i);
        }
        @Override
        void setI(int i){
            this.i = 3*i;
        }
        @Override
        void fun(){
            super.func();
        }
        @Override
        void func(){
             System.out.println("这是子类的func方法");
        }
    }
    ```

> 3.8
java线程分类:
    1. 用户线程
    2. 守护线程: 
