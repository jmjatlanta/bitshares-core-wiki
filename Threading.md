Threading in BitShares Core
===========================

*Work in progress*

BitShares uses `boost::thread` for parallel execution. However, due to the single-threaded internal database, threads are used sparingly.

On top boost::thread, BitShares implements a kind of cooperative multitasking (`using boost::context`). I. e. a thread can have tasks queued for processing, and whenever a task either completes or is blocked in a mutex, the thread switches to the next task in the queue.

Per default there are two main threads, a database thread and a P2P thread. In addition, several more threads are started for asynchronous IO operations (using `boost::asio`). These are available through `fc::default_io_service()`.

With https://github.com/bitshares/bitshares-core/pull/1360 there will be another thread pool. It is mostly meant for parallel processing of computationally intensive things that do not otherwise interfere with each other. This pool is accessed using `fc::do_parallel()`.

P2P Thread
----------

The P2P thread is "owned" by `node_impl`. It has several permanent tasks for handling (accepting, creating, monitoring, closing) P2P connections.

P2P communication is based on messages. Messages start with a message header containing message type and payload size, followed by the payload and padded to a multiple of 16 bytes. Incoming messages concerning the P2P layer are handled within the P2P thread, messages concerning the database (mostly blocks and transactions) are handed over to the database thread. "handed over" means a task is created within the database thread.

Database thread
---------------

The database thread is also the "main" thread of `witness_node`.

Tasks
-----

An `fc::task` consists of two parts - a functor to be executed, and a promise that is fulfilled upon completion of the functor with its return value.

An `fc::promise` is usually paired with an `fc::future`. The future owns the promise (in the sense of RAII) through a `shared_ptr`. This means that at least one future for a given task must exist until the task is completed, because otherwise the promise goes out of scope and the thread that's processing the task will have a problem.
