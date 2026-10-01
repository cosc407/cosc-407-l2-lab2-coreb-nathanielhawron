# Lab 2 results — sealed core

Name:  Nathaniel Hawron
Student number:  14455729
Lab section:  L02
Core:  B — the letter on BRIEF.md
Machine:  MSI katana GF66, Docker (6p+8e cores)
Cores:  20 — an integer

## Tools and sources

Tools and sources: vim

> Mandatory, even if it says "none". **No AI in the lab, at all** — see the
> README. Missing declaration: zero until you supply one. False one: misconduct.

## S2 — the defect · 40 marks

Three or more runs of `./bar given`, including one thread:

```
./bar given 1 2000
mode=given threads=1 rounds=2000 bad=0 firstbad=-1 checksum=ok correct=yes deadlock=no time=0.0050 cpu=0.0051

./bar given 2 2000
mode=given threads=2 rounds=2000 bad=0 firstbad=-1 checksum=ok correct=yes deadlock=no time=0.0201 cpu=0.0211

./bar given 8 2000
mode=given threads=8 rounds=2000 bad=-1 firstbad=-1 checksum=unknown correct=no deadlock=yes time=5.0205 cpu=0.0055
given: no progress after 5.0 s -- giving up. This is a result, not a crash: paste the line above.
```

**S2.1** Name the mechanism: which claim in `given.c`'s header is false, and
what is actually happening? State the barrier's invariant and say which half of
it this code does not keep.

The second claim is false. Every thread is not released, since only one thread is signaled.
The invariants are:
    value >= 0
    value = initial + posts - completes
The error in this code is not caused by an invariant issue, its caused by an implementation error.

**S2.2** Prove it, in the form your `BRIEF.md` requires.

The code works with 1 or 2 threads. This is because one thread doesn't block in the wait function, and with two threads, signal can wake all the other 1 threads. Testing it here doesn't expose the issue, we have to test it outside of the scope of the signal function (i.e. broadcast). However, as soon as a third thread is added, signal is no longer able to wake all the other threads. We can see through the cpu time that none of the threads are actually running (5 seconds, with only 6ms of CPU usage), meaning they are asleep.

**S2.3** Minimality: what breaks if you do less, what it costs if you do more.

My fix will notify all threads that they are to wake up. If we don't notify, all threads, we get a deadlock with more than 2. The alternative is to loop through the number of threads -1 and signal all of them individually, but this costs an extra for loop (extra branches and operations).

## S3 — the measurement · 30 marks

`./bar all <t> <rounds>` at 1, 2, 4 and 8 threads. Pasted, not retyped. If a
mode stops, `all` stops with it — run the modes one at a time and paste those.

```
./bar given 1 2000
mode=given threads=1 rounds=2000 bad=0 firstbad=-1 checksum=ok correct=yes deadlock=no time=0.0118 cpu=0.0116
./bar given 2 2000
mode=given threads=2 rounds=2000 bad=0 firstbad=-1 checksum=ok correct=yes deadlock=no time=0.0269 cpu=0.0302
./bar given 4 2000
mode=given threads=4 rounds=2000 bad=-1 firstbad=-1 checksum=unknown correct=no deadlock=yes time=5.0227 cpu=0.0056
given: no progress after 5.0 s -- giving up. This is a result, not a crash: paste the line above.
./bar given 8 2000
mode=given threads=8 rounds=2000 bad=-1 firstbad=-1 checksum=unknown correct=no deadlock=yes time=5.0266 cpu=0.0066
given: no progress after 5.0 s -- giving up. This is a result, not a crash: paste the line above.

/bar fixed 1 2000
mode=fixed threads=1 rounds=2000 bad=0 firstbad=-1 checksum=ok correct=yes deadlock=no time=0.0067 cpu=0.0066
./bar fixed 2 2000
mode=fixed threads=2 rounds=2000 bad=0 firstbad=-1 checksum=ok correct=yes deadlock=no time=0.0282 cpu=0.0319
./bar fixed 4 2000
mode=fixed threads=4 rounds=2000 bad=0 firstbad=-1 checksum=ok correct=yes deadlock=no time=0.0246 cpu=0.0506
./bar fixed 8 2000
mode=fixed threads=8 rounds=2000 bad=0 firstbad=-1 checksum=ok correct=yes deadlock=no time=0.0556 cpu=0.2124


```

| threads | given: correct? | given: time | given: cpu | fixed: time | fixed: cpu | alt: time | alt: cpu |
|---|---|---|---|---|---|---|---|
| 1 |yes| 0.0118| 0.0116| 0.0067| 0.0066| | |
| 2 |yes| 0.0269| 0.0302| 0.0282| 0.0319| | |
| 4 | no| 5.0277| 0.0056| 0.0246| 0.0506| | |
| 8 | no| 5.0266| 0.0066| 0.0556| 0.2124| | |

**S3.1** Reconcile with `PREDICTION.md`: quote what you predicted, say what
happened, account for the difference. If you were right, say what would have
made you wrong.

"Yes. With one thread, there can't be any contention. Also, b->count++ will go straight to b->n (0 to 1), so it will go straigt to the if statement, signal noone (since no threads are waiting), then exit the wait."
Correct, the single thread didnt have to signal itself.

"It stops. pthread_cond_signal only notifies a single thread that it can wait, while we want to broadcast to all threads. The first two threads to finish will stop at the second wait, the rest will be stuck at the first."
Correct, changing it to broadcast fixed the issue.

"Each run should give the same numbers, since it is running the same operation (as long as the workloads are truly independant in each round)."
Correct, all the checksums were correct.

"Fastest will be fixed, slowest given. Alt will slow down as more threads are added, given will take infinite time since it stops. Alt takes slightly longer because it has to do an additional loop to add to post to the semaphore." 


**S3.2** Which would you ship on this machine, **and what measurement would
change your mind?**

I would ship fixed on this machine. Both fixed and alt work, but fixed is slightly faster.

## S4 — explain-back · 15 marks

> Two or three sentences, your own words: someone who has not seen this code
> asks *what was wrong with it, and what did fixing it cost?*

The problem was that only one thread was being notified by a signal, while we need to notify all with a broadcast. Fixing it didnt cost much, since broadcast and signal have similar performance.

## Anything you got stuck on

Optional. One or two lines.
