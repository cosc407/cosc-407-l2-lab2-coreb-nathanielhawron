# Prediction sheet — push by 0:20, before you compile

> Marked on **having predicted** and on reconciling it in S3.1 — **not on being
> right.** A confident wrong prediction you then explain is full marks. A blank
> page is none. A page timestamped after your first run is worse than none.
>
> Read `src/given.c` and `BRIEF.md`. Run nothing.

Cores:  20 — from PREP.md
Lab 0 spread:  39.5% — the percentage, from PREP.md

> **P1.** `./bar given` on **one** thread — does it come out right? Yes/no, one
> sentence why.

Yes. With one thread, there can't be any contention. Also, b->count++ will go straight to b->n (0 to 1), so it will go straigt to the if statement, signal noone (since no threads are waiting), then exit the wait.

> **P2.** On **8** threads, pick one and commit to it: right answer / wrong
> answer / it stops. If wrong, roughly how big is `bad`? If it stops, say at
> which of the two waits in a round.

It stops. pthread_cond_signal only notifies a single thread that it can wait, while we want to broadcast to all threads. The first two threads to finish will stop at the second wait, the rest will be stuck at the first.

> **P3.** Three runs at 8 threads — **identical** numbers, or different? Think
> about this one before you write it; it is the most useful line on the page.

Each run should give the same numbers, since it is running the same operation (as long as the workloads are truly independant in each round).

> **P4.** Seconds, before measuring. Orders of magnitude are what matter. `cpu`
> is process CPU time over all threads, so `cpu`/`time` is how many cores were
> busy — one number per box.

| | 1 thread: time | 8 threads: time | 8 threads: cpu/time |
|---|---|---|---|
| `given` | inf| inf| inf|
| `fixed` | 10ms| 15ms| 7|
| `alt` | 10ms| 20ms| 6|

> **P5.** Fastest and slowest at 8 threads? Name anything you expect to get
> **slower** as threads are added, and anything you expect to stop altogether.

Fastest will be fixed, slowest given. Alt will slow down as more threads are added, given will take infinite time since it stops. Alt takes slightly longer because it has to do an additional loop to add to post to the semaphore.
