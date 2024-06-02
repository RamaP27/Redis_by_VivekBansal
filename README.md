# Redis_by_VivekBansal

Redis is an in-memory storage that is often used for various purposes like caching, geo search, session storage, analytics, message broker, streaming engine, etc. When it comes to building a high throughput and low latency application, Redis might be one of the most popular technologies that companies use for numerous use cases. I’ve been part of three companies so far, and all of them use Redis on a large scale.

The thing that makes Redis so popular among companies is its speed and support for various use cases that probably no other NoSQL database provides. So, while you would want to choose Redis for its low latency, its support for different use cases that may come in the future makes you want to stick to this solution.

Thanks for reading Curious Engineer! Subscribe for free to receive new posts and support my work.

Pledge your support

Let’s take a moment to reflect upon what makes Redis so fast. At a high level, you can attribute it to the following factors:

[1] In-memory access instead of disk
Redis accesses data from in-memory, unlike other traditional databases that perform reads/writes on the disk. Reading data from in-memory is faster because it’s in the buffer (which means no I/O wait time to read it from disk). Data access from the memory can be faster by several orders of magnitude compared to accessing the data from the disk. For example, refer to the image below:


Credits: ByteByteGo
Please note that Redis is an in-memory storage which does not mean it does not provide persistence. There are mainly three ways if you want to permanently persist data in Redis:

RDB(Redis database): This performs point-in-time snapshots of the dataset at specified intervals.

AOF(Append only file) log: AOF persists all the write commands received by the server which helps in reconstructing the original dataset (for whatever reason) at a point in the past by replaying the commands.

AOF + RDB: Use both strategies in the same Redis instance.

[2] Non-blocking I/O and I/O Multiplexing
Basics first: Let’s try to understand what is Blocking I/O

“I/O = Input/Output operation refers to the communication between a computer and the outside world”

Blocking I/O is a kind of I/O operation where the main thread is suspended or blocked until the I/O operation in progress is completed. Until then, the main thread cannot continue to execute the rest of the code.

Let’s try to understand the behavior of blocking I/O in a server context. Imagine a server-client model. The following steps take place:

A client connects to the server.

The server thread responsible for that connection gets blocked while waiting to read data from the client’s request.

This data is typically stored in a buffer until it's fully read and ready for processing.

While the thread is blocked, it cannot handle other requests.

Until the data is fully read, the server thread is completely blocked and it cannot do anything else. This is called blocking I/O. The simplest conclusion from this is that we cannot serve more than one connection within a single thread.

What can we do to solve Blocking I/O?

Let’s try increasing the number of threads. Even if we use a multi-core CPU to achieve concurrency, there are a still few problems:

Limited concurrency: If the server has a limited number of threads, it can only handle a few concurrent connections before all threads become blocked waiting for data. This can significantly impact the server's responsiveness and ability to handle new requests.

Slow performance: Let’s assume that some threads won’t be blocked on doing I/O operations and are ready to execute, then we need to use mutexes or semaphores to ensure data correctness. Using mutexes or semaphores can lead to slow execution because it allows only one thread to be executed at a given point.

Inefficient resource utilization: While the thread is blocked, the server's CPU and other resources are idle, leading to resource wastage.

What is a Non-blocking I/O?

Non-blocking I/O is an I/O operation that doesn't block the program or thread. Instead of waiting for the operation to complete, the program receives an error code (e.g., EAGAIN or EWOULDBLOCK) indicating it's not ready yet. This allows the program to continue checking other file descriptors for data availability.

Server with Non-blocking I/O

When a server starts, it uses accept() to listen for incoming connections on its socket. Setting the O_NONBLOCK flag on the server's socket enables accept() to return an error if no connection is available, preventing indefinite blocking. This allows the main thread to continue processing other tasks while waiting for connections.

The Non Blocking I/O alone does not solve the problem as the server’s main thread needs to poll again from time to time to see if there are any incoming connections.

What is I/O Multiplexing?

I/O multiplexing is a concept where a single thread or process monitors multiple I/O operations simultaneously. It enables a single thread to handle many connections or I/O operations without getting blocked.

For example: The select() system call allows you to monitor multiple sockets (file descriptors) simultaneously. It returns only when one or more of these sockets are ready for a specific operation (reading, writing, or error).

I/O multiplexing along with Non-Blocking I/O does not solve the problem yet. While select() itself is non-blocking, the main thread still needs to loop through the returned file descriptors to identify which ones are ready, leading to some overhead.

Async I/O along with Non-blocking and Multiplexing

Using the epoll API: The epoll API is similar to poll(2) in that it allows you to monitor multiple file descriptors for I/O events. However, unlike poll which returns the number of ready descriptors, epoll provides a list of the actual ready file descriptors. This eliminates the need for a separate loop to identify ready descriptors, making it more efficient. This approach is similar to how event loop systems work, where the system actively notifies the program about events instead of requiring constant polling.

Conclusion
Well, that’s it!! If someone asks you, how Redis achieve high throughput(manages multiple connections) and low latency despite being single-threaded, remember the following points:

Redis realized that Network I/O is slow, so they persist their data in memory so that writes can be performed faster.

Redis is single-threaded. This avoids all the lock contention due to mutexes and semaphores.

Non-blocking I/O along with I/O multiplexing helps Redis to handle multiple TCP connections in an asynchronous manner.

Additional Info: Redis is single-threaded and thus atomic. This means if there is a single instance running, then you don’t have to worry about handling concurrent transactions because all the transactions will be executed sequentially. While one command is running, no other command will interrupt the process and thus there is zero context switching among the concurrent transactions.

The other way to put this is: if there are 10 clients who are trying to increment a counter in the Redis, then it’s guaranteed that the value would be 10 only and not 9 or 8 or 7, basically not less than 10.

That’s it, folks for this edition of the newsletter. Please consider liking and sharing with your friends as it motivates me to bring you good content for free. If you think I am doing a decent job, share this article in a nice summary with your network. Connect with me on Linkedin or Twitter for more technical posts in the future!

Book exclusive 1:1 with me here.

Thanks for reading Curious Engineer! Subscribe for free to receive new posts and support my work.
