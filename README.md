Download Link: https://assignmentchef.com/product/solved-csc360-assignment-2-synchronization
<br>
In this assignment, you will solve two well-known concurrency problems using two different thread libraries: <em>pthreads</em> and <em>uthreads</em> (uthreads are a simple user level thread library, which you will be provided with and can see all of the guts of!).  While these problems are toys, they were designed to model specific types of real-world synchronization problems that show up in real concurrent systems such as operating systems and video games etc.

The first problem you’ll tackle is the classic Producer Consumer problem.  You’ll solve this problem three ways:

<ul>

 <li>using blocking in pthreads</li>

 <li>using spinlocks and polling in uthreads</li>

 <li>using blocking in uthreads</li>

</ul>

The second problem is the Smokers Problem.  You will solve this in two ways:

<ul>

 <li>using blocking in pthreads</li>

 <li>using blocking in uthreads</li>

</ul>

In each case your program will consist of a solution to the synchronization challenge and a test harness that creates a set of threads to exercise your code and instruments your code to collect information that you can use to convince yourself (and us!) that you have implemented the problem correctly.  Bugs will either be in the form of incorrect results (i.e., violating the stated constraints) or deadlock (i.e., your program hangs).

The code you need for the uthreads portion this assignment will be provided in <em>a2code.zip</em>.  There you will find the complete uthread package, including the implementation of monitors and condition variables and semaphores.   You will also find a Makefile that you will use to build the various parts of this assignment and skeleton files for each of the questions with some parts implemented for you and some parts, listed as TODO, left for you.

<h1>Question 1: The Producer-Consumer Problem</h1>

The producer-consumer problem is a classic.  This problem uses a set of threads that add and remove things from a shared, bounded-size resource pool.  Some threads are producers that add items to the pool and some are consumers that remove items.

Video streaming applications, for example, typically consist of two processes connected by a shared buffer.  The producer fetches video frames from a file or the network, decodes them and adds them to the buffer.  The consumer fetches the decoded frames from the buffer at a designated rate (e.g., 60 frames per second) and delivers them to the graphics system to be displayed.  The buffer is needed because these two processes do not necessarily run at the same rate.  The producer will sometimes be fast and sometimes slow (depending on network performance or video-scene complexity).  On average it is faster than the consumer, but sometimes its slower.

There are two synchronization issues.  First, the resource pool is a shared resource access by multiple threads and thus the producer and consumer code that accesses it are critical sections.  Synchronization is needed to ensure mutual exclusion for these critical sections.

The second type of synchronization is between producers and consumers.  The resource pool has finite size and so producers must sometimes wait for a consumer to free space in the pool, before adding new items.  Similarly, consumers may sometimes find the pool empty and thus have to wait for producers to replenish the pool.

<h2>What to do</h2>

First, model this problem/solution using <em>pthreads </em>with mutexes and condition variables.  Your solution must use at least four threads (two producers and two consumers).  You will find TONS of examples of this out there in the wild… Feel free to share what you find on the forum!

Second, take it on in uthreads, using the provided code in two different ways.  First investigating spinlocks, then blocking.

The file pc_spinlock.c contains a very simple outline of the problem. Modify this file to add threads and synchronization according to the following requirements.  Use <em>uthreads</em> initialized to four processors (i.e., uthread_init(4)).

To keep things simple, the shared resource pool is just a single integer called items.   Set the initial value of items to 0.  To add an item to the pool, increment items by 1.  To remove an item, decrement items by 1.

Producer threads should loop, repeatedly attempting to add items to the pool one at a time and consumer threads should loop removing them, one at a time.  Ensure that each of these add-one or remove-one operations can interleave arbitrarily when the program executes.

Use <em>spinlocks</em> to guarantee mutual exclusion.  To use a spinlock, you must first allocate and initialize (i.e., create) one:

spinlock_t lock; spinlock_create (&amp;lock);

Then you lock and unlock like this:

spinlock_lock (&amp;lock);

…

spinlock_unlock (&amp;lock);

Your code must ensure that items is never less than 0 nor more than MAX_ITEMS (which you can set to 10).  Consumers may have to wait until there is an item to consume and producers may have to wait until there is room for a new item.  In both cases, implement this waiting by <strong><em>spinning</em></strong> on a read of the items variable; consumers waiting for it to be non-zero and producers waiting for it to be less than MAX_ITEMS.  Be sure not to spin while holding the spinlock, because doing so will cause a deadlock (i.e., your program will hang).  And be sure to doublecheck the value of items once you do hold the spinlock to handle a possible race condition between two consumers or two producers.  This code will look similar to the final spinlock implementation shown in class, though in C:

ld   $lock, r1   # r1 = &amp;lock loop: ld   (r1), r0    # r0 = lock

beq  r0, try     # goto try if lock==0 (available)       br   loop        # goto loop if lock!=0 (held) try:  ld   $1, r0      # r0 = 1

xchg (r1), r0    # atomically swap r0 and lock       beq  r0, held    # goto held lock was 0 before swap       br   loop        # try again if another thread holds lock held:                  # we now hold the lock

First spin on the condition without hold the lock, then acquire the lock and re-check the condition.  If the condition no longer holds, release the lock and go back to the first spinning step.

Use the included <em>makefile</em> to compile your uthread program.

<h2>Testing</h2>

To test your solution, run a large number of iterations of each thread.  Add assert statement(s) to ensure that the constraint 0 &lt;= items &lt;= MAX_ITEMS is never violated.  Count the

number of times that producer or consumer threads have to wait by using two global variables called producer_wait_count and consumer_wait_count.  In addition, maintain a

histogram of the values that items takes on and print it out at the end of the program.  Use the histogram to ensure that the total number of changes to items is correct (i.e., equal to the total number of iterations of consumers and producers).  Print the values of the counters and the histogram when the program terminates.  The histogram would look something like this.

int histogram [MAX_ITEMS + 1];

…

histogram [items] ++;  // do this when items changes value

Ensure that your program prints these values when it terminates exactly as is done in the provided code.

Since concurrency bugs are non-deterministic — they only show up some of the time — be sure to run your program several times.  This repeated execution is particularly important to ensure that your program is deadlock-free.

One you have all that going—it’s time to improve it and remove the spinlocks!!!  Make a copy of pc_spinlock_uthread.c and call it pc_mutex_cond_uthread.c.  Modify this file to

replace spinlocks with blocking mutexes and spinning with condition variables so that all waiting is now blocking. Test your program the same way you tested pc_spinlock_uthread.c.

<h1>Question 2: The Smokers Problem</h1>

The smokers problem is a classic synchronization problem, posed by Suhas Patil in 1971.  In this problem there are four actors, each represented by a thread, and three resources required to construct and smoke a cigarette: tobacco(?!), paper, and matches.  One of the actors is the agent and the other three are smokers.  The agent has an infinite supply of all of the resources.  Each smoker has an infinite supply of one resource and nothing else; each smoker possesses a different resource.

The three smoker threads loop attempting to smoke, which requires that they obtain one unit of both of the resources they do not possess.  The agent loops repeatedly, randomly choosing two ingredients to make available to smokers.  Each time the agent does this, one of the three smokers should be able to achieve its heath-destroying goal.  For example, if the agent chose paper and matches, then the tobacco-possessing smoker can consume these two items, combined with its own supply of tobacco, to smoke.

This is a simple model of a general resource-management problem that operating systems deal with in many forms.  To ensure that it captures that real problem correctly, the agent has a few additional constraints placed on it.

The agent is only allowed to communicate by signaling the availability of a resource using a condition variable.  It is not permitted to disclose resource availability in any other way; i.e., smokers cannot ask the agent what is available.  In addition, the agent is not permitted to know anything about the resource needs of smokers; i.e., the agent cannot wakeup a smoker directly.  Finally, each time the agent makes two resources available, it must wait on a condition variable for a smoker to smoke before it can make any additional resources available.

The problem is tricky because when the agent makes two items available, every smoker thread can use one of them, but only one can use both.  If you aren’t careful, you might create a solution that results in deadlock.  For example, if the agent makes paper and matches available, both the paper and the matches smokers want one of these, but neither will be able to smoke because neither has tobacco.  But, if either of them does wake up and consume a resource, that will prevent the tobacco thread from begin able to smoke and thus also prevent the agent from waking up to deliver additional resources.  If this happens, the system is deadlocked; no thread will be able to make further progress.

<h2>What to do</h2>

Implement a deadlock-free solution to the smokers problem in two different C programs using the different thread libraries (name them smoke_pthreads.c,and another one smoke_uthreads.c).

Create four threads: one for the agent and one for each type of smoker.  The agent thread should loop through a set of iterations.  In each iteration it chooses two resources randomly, signals their condition variables, and then waits on a condition variable that smokers signal when they are able to smoke.  When smoker threads are unable to run they must be waiting on a condition variable. When a smoker wakes up to find both of the resources it needs, it signals the agent and goes back to waiting for the next chance to smoke.

The agent must use exactly four condition variables: one for each resource and one to wait for smokers.  The agent must indicate that a resource is available by calling signal on that resource’s condition variables exactly once.  There is no other way for any other part of the system to know which resources are currently available.  The code for the agent in uthreads is provided for you.  You do not need to change this code, but you can.  Just be sure you follow the rules we have just outlined.

<em>You may find it useful to create other threads and add additional condition variables.   It is perfectly fine to do so as long as you follow the constraints imposed on the agent thread.  For example, notice that we have not said how the smokers wait other than to say that they wait on some condition variable.  </em>

To generate a random number in C you can use the procedure random() that is declared in &lt;stdlib.h&gt;.  It gives you a random integer.  You if want a random number between 0 and N, one way to do that is to use the modulus operator; i.e., random() % N.  This procedure returns random numbers starting of a seed value.  Every time you run your program it will by default use the same seed and so calls to random() will produce the same sequence of random numbers.  That is fine.

<h2>Testing</h2>

The most common problem with attempts to solve this problem is deadlock.  The simplest way to diagnose this problem initially is to use printf statements in the agent and smokers that tell you what each is doing.  A printf just before and just after every statement that could block (e.g., every wait) is probably a good idea.  If the printing stops before the program does, you have a deadlock and the last few strings printed should tell you where.   Start with one iteration of the agent.  Get that to work, then try more than one.

Be sure that the strings you print with printf end with a new line character (i.e., “
”), because printf does not actually print until it sees this character or the program terminates.  If you print without the newline and then your program deadlocks, you will not see the string printed and you will be confused about where the program deadlocked.

Once you think you’ve got this working, you’ll want to remove the printf’s so that you can drive the problem through a large number of iterations without being bombarded with output.  One way to do this is to use the C Preprocessor to surround each of your printf statements with a #ifdef directive like this:

#ifdef VERBOSE

printf (“Tobacco smoker is smoking.
”); #endif

A better way — though the more you do with macros the trickier it can get — is to define a macro called VERBOSE_PRINT that is  a printf if VERBOSE is defined and the empty statement otherwise.  To do this, include the following macro definition at the beginning of your program.

#ifdef VERBOSE

#define VERBOSE_PRINT(S, …) printf (S, ##__VA_ARGS__); #else

#define VERBOSE_PRINT(S, …) ;

#endif

And then use the macro instead of printf for debugging statements, like this:

VERBOSE_PRINT (“Tobacco smoker is smoking.
”);

In either case you can now selectively define the VERBOSE macro when you compile your program to turn diagnostic printf’s on or off.

To turn them on (the program now prints all of the debugging statements)):

gcc -D VERBOSE -std=gnu11 -o smoke smoke.c uthread.c uthread_mutex_cond.c -pthread To turn them off (the program now prints no debugging statements):

gcc -std=gnu11 -o smoke smoke.c uthread.c uthread_mutex_cond.c -pthread

<h2>Testing</h2>

Test your programs by driving the agent through a large set of iterations.  Instrument the agent to count the expected times each smokers should smoke and instrument each smoker to count the number of times that each does smoke.  Compare these to ensure they match and print them when the program terminates.

<h1>Challenge 3: Producer Consumer with Semaphores</h1>

Re-implement Question 1 using <em>semaphores</em> in both pthreads and uthreads.  Put your solution in the file called pc_sem_thread.c; start from the provided file in uthreads.  Note that you cannot get wait counts with semaphores because there is no way to know whether a call to sem_wait() will actually wait (remember you can’t check the value of a semaphore).  So, please leave these out of your implementation.

The only synchronization primitives are you permitted to use are wait and signal (in uthreads this is uthread_sem_wait and uthread_sem_signal).  <strong>Use semaphores to replace both the mutexes and condition variables. </strong>