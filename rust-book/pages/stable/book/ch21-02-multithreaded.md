---
type: Web Page
title: From Single-Threaded to Multithreaded Server - The Rust Programming Language
resource: https://doc.rust-lang.org/stable/book/ch21-02-multithreaded.html
timestamp: '2026-07-13T09:33:08.854356+00:00'
---

[From a Single-Threaded to a Multithreaded Server](#from-a-single-threaded-to-a-multithreaded-server)

Right now, the server will process each request in turn, meaning it won’t process a second connection until the first connection is finished processing. If the server received more and more requests, this serial execution would be less and less optimal. If the server receives a request that takes a long time to process, subsequent requests will have to wait until the long request is finished, even if the new requests can be processed quickly. We’ll need to fix this, but first we’ll look at the problem in action.

[Simulating a Slow Request](#simulating-a-slow-request)

We’ll look at how a slowly processing request can affect other requests made to
our current server implementation. Listing 21-10 implements handling a request
to */sleep* with a simulated slow response that will cause the server to sleep
for five seconds before responding.

We switched from `if` to `match` now that we have three cases. We need to
explicitly match on a slice of `request_line` to pattern-match against the
string literal values; `match` doesn’t do automatic referencing and
dereferencing, like the equality method does.

The first arm is the same as the `if` block from Listing 21-9. The second arm
matches a request to */sleep*. When that request is received, the server will
sleep for five seconds before rendering the successful HTML page. The third arm
is the same as the `else` block from Listing 21-9.

You can see how primitive our server is: Real libraries would handle the recognition of multiple requests in a much less verbose way!

Start the server using `cargo run`. Then, open two browser windows: one for
*http://127.0.0.1:7878* and the other for *http://127.0.0.1:7878/sleep*. If you
enter the */* URI a few times, as before, you’ll see it respond quickly. But if
you enter */sleep* and then load */*, you’ll see that */* waits until `sleep`
has slept for its full five seconds before loading.

There are multiple techniques we could use to avoid requests backing up behind a slow request, including using async as we did Chapter 17; the one we’ll implement is a thread pool.

[Improving Throughput with a Thread Pool](#improving-throughput-with-a-thread-pool)

A *thread pool* is a group of spawned threads that are ready and waiting to
handle a task. When the program receives a new task, it assigns one of the
threads in the pool to the task, and that thread will process the task. The
remaining threads in the pool are available to handle any other tasks that come
in while the first thread is processing. When the first thread is done
processing its task, it’s returned to the pool of idle threads, ready to handle
a new task. A thread pool allows you to process connections concurrently,
increasing the throughput of your server.

We’ll limit the number of threads in the pool to a small number to protect us from DoS attacks; if we had our program create a new thread for each request as it came in, someone making 10 million requests to our server could wreak havoc by using up all our server’s resources and grinding the processing of requests to a halt.

Rather than spawning unlimited threads, then, we’ll have a fixed number of
threads waiting in the pool. Requests that come in are sent to the pool for
processing. The pool will maintain a queue of incoming requests. Each of the
threads in the pool will pop off a request from this queue, handle the request,
and then ask the queue for another request. With this design, we can process up
to * N* requests concurrently, where 

*is the number of threads. If each thread is responding to a long-running request, subsequent requests can still back up in the queue, but we’ve increased the number of long-running requests we can handle before reaching that point.*

`N`This technique is just one of many ways to improve the throughput of a web server. Other options you might explore are the fork/join model, the single-threaded async I/O model, and the multithreaded async I/O model. If you’re interested in this topic, you can read more about other solutions and try to implement them; with a low-level language like Rust, all of these options are possible.

Before we begin implementing a thread pool, let’s talk about what using the pool should look like. When you’re trying to design code, writing the client interface first can help guide your design. Write the API of the code so that it’s structured in the way you want to call it; then, implement the functionality within that structure rather than implementing the functionality and then designing the public API.

Similar to how we used test-driven development in the project in Chapter 12, we’ll use compiler-driven development here. We’ll write the code that calls the functions we want, and then we’ll look at errors from the compiler to determine what we should change next to get the code to work. Before we do that, however, we’ll explore the technique we’re not going to use as a starting point.

[Spawning a Thread for Each Request](#spawning-a-thread-for-each-request)

First, let’s explore how our code might look if it did create a new thread for every connection. As mentioned earlier, this isn’t our final plan due to the problems with potentially spawning an unlimited number of threads, but it is a starting point to get a working multithreaded server first. Then, we’ll add the thread pool as an improvement, and contrasting the two solutions will be easier.

Listing 21-11 shows the changes to make to `main` to spawn a new thread to
handle each stream within the `for` loop.

As you learned in Chapter 16, `thread::spawn` will create a new thread and then
run the code in the closure in the new thread. If you run this code and load
*/sleep* in your browser, then */* in two more browser tabs, you’ll indeed see
that the requests to */* don’t have to wait for */sleep* to finish. However, as
we mentioned, this will eventually overwhelm the system because you’d be making
new threads without any limit.

You may also recall from Chapter 17 that this is exactly the kind of situation where async and await really shine! Keep that in mind as we build the thread pool and think about how things would look different or the same with async.

[Creating a Finite Number of Threads](#creating-a-finite-number-of-threads)

We want our thread pool to work in a similar, familiar way so that switching
from threads to a thread pool doesn’t require large changes to the code that
uses our API. Listing 21-12 shows the hypothetical interface for a `ThreadPool`
struct we want to use instead of `thread::spawn`.

We use `ThreadPool::new` to create a new thread pool with a configurable number
of threads, in this case four. Then, in the `for` loop, `pool.execute` has a
similar interface as `thread::spawn` in that it takes a closure that the pool
should run for each stream. We need to implement `pool.execute` so that it
takes the closure and gives it to a thread in the pool to run. This code won’t
yet compile, but we’ll try so that the compiler can guide us in how to fix it.

[Building ](#building-threadpool-using-compiler-driven-development)`ThreadPool` Using Compiler-Driven Development

`ThreadPool` Using Compiler-Driven DevelopmentMake the changes in Listing 21-12 to *src/main.rs*, and then let’s use the
compiler errors from `cargo check` to drive our development. Here is the first
error we get:

```
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0433]: failed to resolve: use of undeclared type `ThreadPool`
  --> src/main.rs:11:16
   |
11 |     let pool = ThreadPool::new(4);
   |                ^^^^^^^^^^ use of undeclared type `ThreadPool`
For more information about this error, try `rustc --explain E0433`.
error: could not compile `hello` (bin "hello") due to 1 previous error
```
Great! This error tells us we need a `ThreadPool` type or module, so we’ll
build one now. Our `ThreadPool` implementation will be independent of the kind
of work our web server is doing. So, let’s switch the `hello` crate from a
binary crate to a library crate to hold our `ThreadPool` implementation. After
we change to a library crate, we could also use the separate thread pool
library for any work we want to do using a thread pool, not just for serving
web requests.

Create a *src/lib.rs* file that contains the following, which is the simplest
definition of a `ThreadPool` struct that we can have for now:

Then, edit the *main.rs* file to bring `ThreadPool` into scope from the library
crate by adding the following code to the top of *src/main.rs*:

This code still won’t work, but let’s check it again to get the next error that we need to address:

```
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no function or associated item named `new` found for struct `ThreadPool` in the current scope
  --> src/main.rs:12:28
   |
12 |     let pool = ThreadPool::new(4);
   |                            ^^^ function or associated item not found in `ThreadPool`
For more information about this error, try `rustc --explain E0599`.
error: could not compile `hello` (bin "hello") due to 1 previous error
```
This error indicates that next we need to create an associated function named
`new` for `ThreadPool`. We also know that `new` needs to have one parameter
that can accept `4` as an argument and should return a `ThreadPool` instance.
Let’s implement the simplest `new` function that will have those
characteristics:

We chose `usize` as the type of the `size` parameter because we know that a
negative number of threads doesn’t make any sense. We also know we’ll use this
`4` as the number of elements in a collection of threads, which is what the
`usize` type is for, as discussed in the [“Integer Types”](ch03-02-data-types.html#integer-types) section in Chapter 3.

Let’s check the code again:

```
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no method named `execute` found for struct `ThreadPool` in the current scope
  --> src/main.rs:17:14
   |
17 |         pool.execute(|| {
   |         -----^^^^^^^ method not found in `ThreadPool`
For more information about this error, try `rustc --explain E0599`.
error: could not compile `hello` (bin "hello") due to 1 previous error
```
Now the error occurs because we don’t have an `execute` method on `ThreadPool`.
Recall from the [“Creating a Finite Number of
Threads”](#creating-a-finite-number-of-threads) section that we
decided our thread pool should have an interface similar to `thread::spawn`. In
addition, we’ll implement the `execute` function so that it takes the closure
it’s given and gives it to an idle thread in the pool to run.

We’ll define the `execute` method on `ThreadPool` to take a closure as a
parameter. Recall from the [“Moving Captured Values Out of
Closures”](ch13-01-closures.html#moving-captured-values-out-of-closures) in Chapter 13 that we can
take closures as parameters with three different traits: `Fn`, `FnMut`, and
`FnOnce`. We need to decide which kind of closure to use here. We know we’ll
end up doing something similar to the standard library `thread::spawn`
implementation, so we can look at what bounds the signature of `thread::spawn`
has on its parameter. The documentation shows us the following:

```
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```
The `F` type parameter is the one we’re concerned with here; the `T` type
parameter is related to the return value, and we’re not concerned with that. We
can see that `spawn` uses `FnOnce` as the trait bound on `F`. This is probably
what we want as well, because we’ll eventually pass the argument we get in
`execute` to `spawn`. We can be further confident that `FnOnce` is the trait we
want to use because the thread for running a request will only execute that
request’s closure one time, which matches the `Once` in `FnOnce`.

The `F` type parameter also has the trait bound `Send` and the lifetime bound
`'static`, which are useful in our situation: We need `Send` to transfer the
closure from one thread to another and `'static` because we don’t know how long
the thread will take to execute. Let’s create an `execute` method on
`ThreadPool` that will take a generic parameter of type `F` with these bounds:

We still use the `()` after `FnOnce` because this `FnOnce` represents a closure
that takes no parameters and returns the unit type `()`. Just like function
definitions, the return type can be omitted from the signature, but even if we
have no parameters, we still need the parentheses.

Again, this is the simplest implementation of the `execute` method: It does
nothing, but we’re only trying to make our code compile. Let’s check it again:

```
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.24s
```
It compiles! But note that if you try `cargo run` and make a request in the
browser, you’ll see the errors in the browser that we saw at the beginning of
the chapter. Our library isn’t actually calling the closure passed to `execute`
yet!

Note: A saying you might hear about languages with strict compilers, such as
Haskell and Rust, is “If the code compiles, it works.” But this saying is not
universally true. Our project compiles, but it does absolutely nothing! If we
were building a real, complete project, this would be a good time to start
writing unit tests to check that the code compiles *and* has the behavior we
want.

Consider: What would be different here if we were going to execute a future instead of a closure?

[Validating the Number of Threads in ](#validating-the-number-of-threads-in-new)`new`

`new`We aren’t doing anything with the parameters to `new` and `execute`. Let’s
implement the bodies of these functions with the behavior we want. To start,
let’s think about `new`. Earlier we chose an unsigned type for the `size`
parameter because a pool with a negative number of threads makes no sense.
However, a pool with zero threads also makes no sense, yet zero is a perfectly
valid `usize`. We’ll add code to check that `size` is greater than zero before
we return a `ThreadPool` instance, and we’ll have the program panic if it
receives a zero by using the `assert!` macro, as shown in Listing 21-13.

We’ve also added some documentation for our `ThreadPool` with doc comments.
Note that we followed good documentation practices by adding a section that
calls out the situations in which our function can panic, as discussed in
Chapter 14. Try running `cargo doc --open` and clicking the `ThreadPool` struct
to see what the generated docs for `new` look like!

Instead of adding the `assert!` macro as we’ve done here, we could change `new`
into `build` and return a `Result` like we did with `Config::build` in the I/O
project in Listing 12-9. But we’ve decided in this case that trying to create a
thread pool without any threads should be an unrecoverable error. If you’re
feeling ambitious, try to write a function named `build` with the following
signature to compare with the `new` function:

`pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {`[Creating Space to Store the Threads](#creating-space-to-store-the-threads)

Now that we have a way to know we have a valid number of threads to store in
the pool, we can create those threads and store them in the `ThreadPool` struct
before returning the struct. But how do we “store” a thread? Let’s take another
look at the `thread::spawn` signature:

```
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```
The `spawn` function returns a `JoinHandle<T>`, where `T` is the type that the
closure returns. Let’s try using `JoinHandle` too and see what happens. In our
case, the closures we’re passing to the thread pool will handle the connection
and not return anything, so `T` will be the unit type `()`.

The code in Listing 21-14 will compile, but it doesn’t create any threads yet.
We’ve changed the definition of `ThreadPool` to hold a vector of
`thread::JoinHandle<()>` instances, initialized the vector with a capacity of
`size`, set up a `for` loop that will run some code to create the threads, and
returned a `ThreadPool` instance containing them.

We’ve brought `std::thread` into scope in the library crate because we’re
using `thread::JoinHandle` as the type of the items in the vector in
`ThreadPool`.

Once a valid size is received, our `ThreadPool` creates a new vector that can
hold `size` items. The `with_capacity` function performs the same task as
`Vec::new` but with an important difference: It pre-allocates space in the
vector. Because we know we need to store `size` elements in the vector, doing
this allocation up front is slightly more efficient than using `Vec::new`,
which resizes itself as elements are inserted.

When you run `cargo check` again, it should succeed.

[Sending Code from the ](#sending-code-from-the-threadpool-to-a-thread)`ThreadPool` to a Thread

`ThreadPool` to a ThreadWe left a comment in the `for` loop in Listing 21-14 regarding the creation of
threads. Here, we’ll look at how we actually create threads. The standard
library provides `thread::spawn` as a way to create threads, and
`thread::spawn` expects to get some code the thread should run as soon as the
thread is created. However, in our case, we want to create the threads and have
them *wait* for code that we’ll send later. The standard library’s
implementation of threads doesn’t include any way to do that; we have to
implement it manually.

We’ll implement this behavior by introducing a new data structure between the
`ThreadPool` and the threads that will manage this new behavior. We’ll call
this data structure *Worker*, which is a common term in pooling
implementations. The `Worker` picks up code that needs to be run and runs the
code in its thread.

Think of people working in the kitchen at a restaurant: The workers wait until orders come in from customers, and then they’re responsible for taking those orders and filling them.

Instead of storing a vector of `JoinHandle<()>` instances in the thread pool,
we’ll store instances of the `Worker` struct. Each `Worker` will store a single
`JoinHandle<()>` instance. Then, we’ll implement a method on `Worker` that will
take a closure of code to run and send it to the already running thread for
execution. We’ll also give each `Worker` an `id` so that we can distinguish
between the different instances of `Worker` in the pool when logging or
debugging.

Here is the new process that will happen when we create a `ThreadPool`. We’ll
implement the code that sends the closure to the thread after we have `Worker`
set up in this way:

- Define a `Worker`struct that holds an`id`and a`JoinHandle<()>`.
- Change `ThreadPool`to hold a vector of`Worker`instances.
- Define a `Worker::new`function that takes an`id`number and returns a`Worker`instance that holds the`id`and a thread spawned with an empty closure.
- In `ThreadPool::new`, use the`for`loop counter to generate an`id`, create a new`Worker`with that`id`, and store the`Worker`in the vector.

If you’re up for a challenge, try implementing these changes on your own before looking at the code in Listing 21-15.

Ready? Here is Listing 21-15 with one way to make the preceding modifications.

We’ve changed the name of the field on `ThreadPool` from `threads` to `workers`
because it’s now holding `Worker` instances instead of `JoinHandle<()>`
instances. We use the counter in the `for` loop as an argument to
`Worker::new`, and we store each new `Worker` in the vector named `workers`.

External code (like our server in *src/main.rs*) doesn’t need to know the
implementation details regarding using a `Worker` struct within `ThreadPool`,
so we make the `Worker` struct and its `new` function private. The
`Worker::new` function uses the `id` we give it and stores a `JoinHandle<()>`
instance that is created by spawning a new thread using an empty closure.

Note: If the operating system can’t create a thread because there aren’t
enough system resources, `thread::spawn` will panic. That will cause our
whole server to panic, even though the creation of some threads might
succeed. For simplicity’s sake, this behavior is fine, but in a production
thread pool implementation, you’d likely want to use
[ std::thread::Builder](../std/thread/struct.Builder.html) and its

[method that returns](../std/thread/struct.Builder.html#method.spawn)

`spawn``Result` instead.This code will compile and will store the number of `Worker` instances we
specified as an argument to `ThreadPool::new`. But we’re *still* not processing
the closure that we get in `execute`. Let’s look at how to do that next.

[Sending Requests to Threads via Channels](#sending-requests-to-threads-via-channels)

The next problem we’ll tackle is that the closures given to `thread::spawn` do
absolutely nothing. Currently, we get the closure we want to execute in the
`execute` method. But we need to give `thread::spawn` a closure to run when we
create each `Worker` during the creation of the `ThreadPool`.

We want the `Worker` structs that we just created to fetch the code to run from
a queue held in the `ThreadPool` and send that code to its thread to run.

The channels we learned about in Chapter 16—a simple way to communicate between
two threads—would be perfect for this use case. We’ll use a channel to function
as the queue of jobs, and `execute` will send a job from the `ThreadPool` to
the `Worker` instances, which will send the job to its thread. Here is the plan:

- The `ThreadPool`will create a channel and hold on to the sender.
- Each `Worker`will hold on to the receiver.
- We’ll create a new `Job`struct that will hold the closures we want to send down the channel.
- The `execute`method will send the job it wants to execute through the sender.
- In its thread, the `Worker`will loop over its receiver and execute the closures of any jobs it receives.

Let’s start by creating a channel in `ThreadPool::new` and holding the sender
in the `ThreadPool` instance, as shown in Listing 21-16. The `Job` struct
doesn’t hold anything for now but will be the type of item we’re sending down
the channel.

In `ThreadPool::new`, we create our new channel and have the pool hold the
sender. This will successfully compile.

Let’s try passing a receiver of the channel into each `Worker` as the thread
pool creates the channel. We know we want to use the receiver in the thread that
the `Worker` instances spawn, so we’ll reference the `receiver` parameter in the
closure. The code in Listing 21-17 won’t quite compile yet.

We’ve made some small and straightforward changes: We pass the receiver into
`Worker::new`, and then we use it inside the closure.

When we try to check this code, we get this error:

```
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0382]: use of moved value: `receiver`
  --> src/lib.rs:26:42
   |
21 |         let (sender, receiver) = mpsc::channel();
   |                      -------- move occurs because `receiver` has type `std::sync::mpsc::Receiver<Job>`, which does not implement the `Copy` trait
...
25 |         for id in 0..size {
   |         ----------------- inside of this loop
26 |             workers.push(Worker::new(id, receiver));
   |                                          ^^^^^^^^ value moved here, in previous iteration of loop
   |
note: consider changing this parameter type in method `new` to borrow instead if owning the value isn't necessary
  --> src/lib.rs:47:33
   |
47 |     fn new(id: usize, receiver: mpsc::Receiver<Job>) -> Worker {
   |        --- in this method       ^^^^^^^^^^^^^^^^^^^ this parameter takes ownership of the value
help: consider moving the expression out of the loop so it is only moved once
   |
25 ~         let mut value = Worker::new(id, receiver);
26 ~         for id in 0..size {
27 ~             workers.push(value);
   |
For more information about this error, try `rustc --explain E0382`.
error: could not compile `hello` (lib) due to 1 previous error
```
The code is trying to pass `receiver` to multiple `Worker` instances. This
won’t work, as you’ll recall from Chapter 16: The channel implementation that
Rust provides is multiple *producer*, single *consumer*. This means we can’t
just clone the consuming end of the channel to fix this code. We also don’t
want to send a message multiple times to multiple consumers; we want one list
of messages with multiple `Worker` instances such that each message gets
processed once.

Additionally, taking a job off the channel queue involves mutating the
`receiver`, so the threads need a safe way to share and modify `receiver`;
otherwise, we might get race conditions (as covered in Chapter 16).

Recall the thread-safe smart pointers discussed in Chapter 16: To share
ownership across multiple threads and allow the threads to mutate the value, we
need to use `Arc<Mutex<T>>`. The `Arc` type will let multiple `Worker` instances
own the receiver, and `Mutex` will ensure that only one `Worker` gets a job from
the receiver at a time. Listing 21-18 shows the changes we need to make.

In `ThreadPool::new`, we put the receiver in an `Arc` and a `Mutex`. For each
new `Worker`, we clone the `Arc` to bump the reference count so that the
`Worker` instances can share ownership of the receiver.

With these changes, the code compiles! We’re getting there!

[Implementing the ](#implementing-the-execute-method)`execute` Method

`execute` MethodLet’s finally implement the `execute` method on `ThreadPool`. We’ll also change
`Job` from a struct to a type alias for a trait object that holds the type of
closure that `execute` receives. As discussed in the [“Type Synonyms and Type
Aliases”](ch20-03-advanced-types.html#type-synonyms-and-type-aliases) section in Chapter 20, type aliases
allow us to make long types shorter for ease of use. Look at Listing 21-19.

After creating a new `Job` instance using the closure we get in `execute`, we
send that job down the sending end of the channel. We’re calling `unwrap` on
`send` for the case that sending fails. This might happen if, for example, we
stop all our threads from executing, meaning the receiving end has stopped
receiving new messages. At the moment, we can’t stop our threads from
executing: Our threads continue executing as long as the pool exists. The
reason we use `unwrap` is that we know the failure case won’t happen, but the
compiler doesn’t know that.

But we’re not quite done yet! In the `Worker`, our closure being passed to
`thread::spawn` still only *references* the receiving end of the channel.
Instead, we need the closure to loop forever, asking the receiving end of the
channel for a job and running the job when it gets one. Let’s make the change
shown in Listing 21-20 to `Worker::new`.

Here, we first call `lock` on the `receiver` to acquire the mutex, and then we
call `unwrap` to panic on any errors. Acquiring a lock might fail if the mutex
is in a *poisoned* state, which can happen if some other thread panicked while
holding the lock rather than releasing the lock. In this situation, calling
`unwrap` to have this thread panic is the correct action to take. Feel free to
change this `unwrap` to an `expect` with an error message that is meaningful to
you.

If we get the lock on the mutex, we call `recv` to receive a `Job` from the
channel. A final `unwrap` moves past any errors here as well, which might occur
if the thread holding the sender has shut down, similar to how the `send`
method returns `Err` if the receiver shuts down.

The call to `recv` blocks, so if there is no job yet, the current thread will
wait until a job becomes available. The `Mutex<T>` ensures that only one
`Worker` thread at a time is trying to request a job.

Our thread pool is now in a working state! Give it a `cargo run` and make some
requests:

```
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field `workers` is never read
 --> src/lib.rs:7:5
  |
6 | pub struct ThreadPool {
  |            ---------- field in this struct
7 |     workers: Vec<Worker>,
  |     ^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default
warning: fields `id` and `thread` are never read
  --> src/lib.rs:48:5
   |
47 | struct Worker {
   |        ------ fields in this struct
48 |     id: usize,
   |     ^^
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^
warning: `hello` (lib) generated 2 warnings
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.91s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```
Success! We now have a thread pool that executes connections asynchronously.
There are never more than four threads created, so our system won’t get
overloaded if the server receives a lot of requests. If we make a request to
*/sleep*, the server will be able to serve other requests by having another
thread run them.

Note: If you open */sleep* in multiple browser windows simultaneously, they
might load one at a time in five-second intervals. Some web browsers execute
multiple instances of the same request sequentially for caching reasons. This
limitation is not caused by our web server.

This is a good time to pause and consider how the code in Listings 21-18, 21-19, and 21-20 would be different if we were using futures instead of a closure for the work to be done. What types would change? How would the method signatures be different, if at all? What parts of the code would stay the same?

After learning about the `while let` loop in Chapter 17 and Chapter 19, you
might be wondering why we didn’t write the `Worker` thread code as shown in
Listing 21-21.

This code compiles and runs but doesn’t result in the desired threading
behavior: A slow request will still cause other requests to wait to be
processed. The reason is somewhat subtle: The `Mutex` struct has no public
`unlock` method because the ownership of the lock is based on the lifetime of
the `MutexGuard<T>` within the `LockResult<MutexGuard<T>>` that the `lock`
method returns. At compile time, the borrow checker can then enforce the rule
that a resource guarded by a `Mutex` cannot be accessed unless we hold the
lock. However, this implementation can also result in the lock being held
longer than intended if we aren’t mindful of the lifetime of the
`MutexGuard<T>`.

The code in Listing 21-20 that uses `let job = receiver.lock().unwrap().recv().unwrap();` works because with `let`, any
temporary values used in the expression on the right-hand side of the equal
sign are immediately dropped when the `let` statement ends. However, `while let` (and `if let` and `match`) does not drop temporary values until the end of
the associated block. In Listing 21-21, the lock remains held for the duration
of the call to `job()`, meaning other `Worker` instances cannot receive jobs.

# Citations

1. Source page: https://doc.rust-lang.org/stable/book/ch21-02-multithreaded.html
