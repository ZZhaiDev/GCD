# GCD
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
