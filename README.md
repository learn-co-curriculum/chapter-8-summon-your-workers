

# Designing Elixir Systems with OTP


## Chapter 8: Summon Your Workers (Accio Worker)

### Summary			



*    “A worker is process machinery that lets us divide labor for reliability, performance or scalability.”  
*   Workers
    *   Live in boundary. Not quite part of lifecycle layer b/c lives outside of OTP policy. Not exactly boundary b/c it starts and stops work.
    *   Provide concurrency 
*   Motivations
    *   Concurrency
        *   Concurrency has a price. 
        *   Concurrency means doing more than one thing at the same time.
        *   Boundary layer is where concurrency comes into play, file IO and database access, for example. 
    *    Isolation
        *   Processes insulate users from one another. Crashes in one user’s processes won’t affect another.
        *   Isolation limits damage of failure to noncritical subsystem.
    *   Scaling
        *   Elixir allows instance of program to take advantage of all system cores, which makes Elixir scale well. 
        *   Most of the time in Elixir, you don’t need to introduce your own concurrency abstractions. 
        *   Use your tools appropriately and understand what they’re doing for you.
*   Know Your Tools
    *   Worker layer is a concurrency management layer. 
    *   Dependencies:
        *   Phoenix channels use OTP. Phoenix deals with hard parts! 
        *   Adding an OTP app as a dependency gives you workers(sometimes?) p. 153-154
    *   “Generally, avoid naked processes”
        *   Elixir beginners go after processes, but you should use “proven features” instead (hottake)
        *   Use higher abstractions to work with processes. 
    *   Make Serial Code Concurrent with Tasks
        *   Tasks: for executing one-time single-purpose jobs in a process
        *   Use `async_stream` for spreading slow work out and also apply backpressure. The max number of tasks to run at a time defaults usually to the number of cores so you get all the concurrency you can use.
*   Poolboy
    *   Poolboy is an Erlang application for sharing pools of common resources.
    *   Good for programs that need to use processes that take a while to start. Creates a shared _pool_ of processes. Good for throttling. 
    *   Poolboy is an OTP app. Great example of leveraging other frameworks to provide workers.
*   Code Example
    *   That `Enum.split_while` seems really handy
    *   The usage of the optional timeout to get control back and start any quizzes that need to be started is cool.			
    *   “GenServer timeouts are one of the most underused features in OTP. Loosely stated, a timeout says “If nothing is happening in x milliseconds, I’ll make it happen.” More specifically, if no message is received before the timeout occurs, OTP will send the scheduled timeout. That’s ideal for our purposes.”
    *   Hibernate. 


### Questions



*   Can you think of use cases for `Task.async_stream`? Would this be good for processing large data sets or something? (Maybe Timber.io uses it?)
*   I find the cleanup part of shutting down processes hard to visualize. Maybe just a mental block thing. Is this the case for anyone else?
*   Registrar currently has a separate app for running workers. Some respond to webhooks, which are actually messages pulled off a Rabbit queue, while others are scheduled tasks that run at intervals. Are there any other applications or abstractions we should be taking advantage of?
    *   Umbrella apps! 
    *   Processes subscribe to Rabbit queues and call function in core app. Is it a worker???? 
        *   Mirrors same structure in Rails app 
        *   Not necessarily a worker according to book’s definition
        *   We take in a request and process it. Could be more like a boundary, the interface for async requests. 
        *   Boundaries of application. Controller vs message consumer. 
    *   Scheduled / timed tasks 
        *   These are more like the workers the book describes 
        *   “Spawn” usage 
            *   Is a concurrency primitive
            *   Prefer ability to start / stop cleanly 
            *   One-off thing is probably okay to spawn
            *   For everything else, use the GenServer abstraction  
*   Related: The schedule quiz app appears to take the list of scheduled quizzes, get the first one, calculate the timeout based on the difference between the start date of the quiz and the time now. So, if the Proctor doesn’t receive another message between now and the timeout, it’ll send itself a message to start any quizzes that should be started. Is that scheduling logic we could make use of for Registrar’s “timed tasks”? 
*   Phoeyonce uses poolboy. Can we talk a bit more about the implementation and why it seems like a good use case? 
    *   Ecto uses it. Connections to DB. 
    *   IDE v2 => students reconnecting to new server at the same time. 
    *   Poolboy manages # of allowable containers to be created at a time. 
<p class='util--hide'>View <a href='https://learn.co/lessons/chapter-8-summon-your-workers'>Chapter 8 Summon Your Workers</a> on Learn.co and start learning to code for free.</p>
