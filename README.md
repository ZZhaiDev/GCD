# GCD
#### GCD 要点
* 同步执行（sync）vs 异步执行（async）
* 队列（Dispatch Queue），串行队列（Serial Dispatch Queue），并发队列（Concurrent Dispatch Queue），主队列（Main Dispatch Queue），全局并发队列（Global Dispatch Queue）
* GCD 栅栏方法：**dispatch_barrier_async**
    - 我们有时需要异步执行两组操作，而且第一组操作执行完之后，才能开始执行第二组操作。这样我们就需要一个相当于 栅栏 一样的一个方法将两组异步执行的操作组给分割起来，当然这里的操作组里可以包含一个或多个任务。这就需要用到dispatch_barrier_async 方法在两个操作组间形成栅栏。
* GCD 延时执行方法：**dispatch_after**
    - dispatch_after 方法并不是在指定时间之后才开始执行处理，而是在指定时间之后将任务追加到主队列中。严格来说，这个时间并不是绝对准确的
* GCD 一次性代码（只执行一次）：**dispatch_once**
    - 我们在创建单例、或者有整个程序运行过程中只执行一次的代码时，我们就用到了 GCD 的 dispatch_once 方法。使用 dispatch_once 方法能保证某段代码在程序运行过程中只被执行 1 次，并且即使在多线程的环境下，dispatch_once 也可以保证线程安全。
* GCD 队列组：**dispatch_group**
    - 分别异步执行2个耗时任务，然后当2个耗时任务都执行完毕后再回到主线程执行任务。这时候我们可以用到 GCD 的队列组。dispatch_group_enter、dispatch_group_leave
    — 调用队列组的 dispatch_group_notify 回到指定线程执行任务。或者使用 dispatch_group_wait 回到当前线程继续向下执行（会阻塞当前线程）
* GCD 信号量：**dispatch_semaphore**
    - **必看**： https://medium.com/@roykronenfeld/semaphores-in-swift-e296ea80f860
    - A semaphore consist of a **threads queue** and a **counter value**
    - Call **wait()** each time before using the shared resource. We are basically asking the semaphore if the shared resource is available or not. If not, we will wait.
    - Call **signal()** each time after using the shared resource. We are basically signaling the semaphore that we are done interacting with the shared resource.
    - 保持线程同步，将异步执行任务转换为同步执行任务
    - 可保证线程安全，为线程加锁，多个线程操作一个数据/可实现下载15首歌曲，但是同时最多下载三首
    - **dispatch_semaphore_create**：创建一个 Semaphore 并初始化信号的总量
    - **dispatch_semaphore_signa**l：发送一个信号，让信号总量加 1
    - **dispatch_semaphore_wait**：可以使总信号量减 1，信号总量小于 0 时就会一直等待（阻塞所在线程），否则就可以正常执行
    
#### What is GCD?
Grand Central Dispatch (GCD) is a low-level API for managing concurrent operations. It can help you improve your app’s responsiveness by deferring computationally expensive tasks to the background. It’s an easier concurrency model to work with than locks and threads.

#### Why do We Use GCD?
Threads are complex and dangerous, gcd makes asynchronous programming easier and safer by hiding threads from the developer

#### What is Thread Safe?
Any framework can run in background thread is thread safe, UIkit and core data are not thread safe

#### What is Readers&Writers problems or Concurrent Queue with Barrier
* What if there was a way that you can ensure no writing occurs while reading and no reading occurs while writing using concurrency? Well, there is a way to this using `Concurrent Queue with Barrier`. 
* If we insert a barrier to the write operation, we ensure that the writing will occur after all the reading in the queue is performed and that no reading occurs while writing.

#### How to Use GCD to make a Thread Safe Array
* Thread Safe Array
    ```swift
    class SafeArray<T> {
      private let concurrentQueue = DispatchQueue(label: "concurrentQueue", attributes: .concurrent)
      private var array = [T]()
      public func append(_ value: T) {
          concurrentQueue.sync(flags: .barrier) { self.array.append(value)  }
      }
      var last: T? {
          var result: T?
          concurrentQueue.sync {  result = array.last }
          return result
      }
      var count: Int {
          var result = 0
          concurrentQueue.sync {  result = array.count  }
          return result
      }
    }
    ```
* Test Thread Safe Array VS Non Thread Safe Array
    ```swift
            let array = SafeArray<Any>()    // safe array
            //let array = [Int]()           // non safe array
            var iterations = 1000
            let start = Date().timeIntervalSince1970
            DispatchQueue.concurrentPerform(iterations: iterations) { index in
                let last = (array.last ?? 0) as! Int
                array.append(last + 1)
                DispatchQueue.global().sync {
                    iterations -= 1
                    guard iterations <= 0 else { return }
                    let message = String(format: "safe loop took %.3f seconds, count: %d.",
                                         Date().timeIntervalSince1970 - start,
                                         array.count)
                    print(message)
                }
            }
    ```
* Result
    - Safe Array: took 0.026 seconds, array.count == 1000.
    - Non Safe Array: Will Crash, or array.count << 1000.
    
### Other Resources.
   * [Concurrency in Swift: Reader Writer Lock](https://medium.com/@dmytro.anokhin/concurrency-in-swift-reader-writer-lock-4f255ae73422)
   * [Dispatch Barriers in Swift 3](https://medium.com/@oyalhi/dispatch-barriers-in-swift-3-6c4a295215d6)
   * [Creating Thread-Safe Arrays in Swift](https://basememara.com/creating-thread-safe-arrays-in-swift/)
   * [Create thread safe array in Swift](https://stackoverflow.com/questions/28191079/create-thread-safe-array-in-swift)
   * [Swift 线程安全数组](https://bignerdcoding.com/archives/58.html)
