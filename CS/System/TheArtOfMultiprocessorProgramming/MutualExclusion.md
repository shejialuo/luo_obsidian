# Chapter 2 Mutual Exclusion

Mutual exclusion is perhaps the most prevalent form of coordination in multiprocessor programming. This chapter covers classical mutual exclusion algorithms that work by reading and writing shared memory.

## 2.1 Time

Reasoning about concurrent computation is mostly reasoning about time. We need a simple but unambiguous language to talk about events and durations in time.

A thread is a *state machine*, and its state transitions are called *events*.

Events are *instantaneous*: they occur at a single instant of time. It is convenient to require that events are never simultaneous: distinct events occur at distinct times (As a practical matter, if we are unsure about the order of two events that happen very close in time, then any order will do). A thread $A$ produces a sequence of events $a_{0}, a_{1}, \dots$ threads typically contain loops, so a single program statement can produce many events. We denote the $j$-th occurrence of an event $a_{i}$ by $a_{i}^{j}$. One event $a$ *precedes* another event $b$, written $a \rightarrow b$, if $a$ occurs at an earlier time.

Let $a_{0}$ and $a_{1}$ be events such that $a_{0} \rightarrow a_{1}$. An *interval* $(a_{0}, a_{1})$ is the duration between $a_{0}$ and $a_{1}$. Interval $I_{A}= (a_{0}, a_{1})$ *precedes* $_{B} = (b_{0}, b_{1})$, written $I_{A} \rightarrow I_{B}$, if $a_{1} \to b_{0}$. Intervals that are unrelated by $\rightarrow$ are said to be *concurrent*. By analogy with events, we denote the $j$-th execution of interval $I_{A}$ by $I_{A}^{j}$.

## 2.2 Critical Sections

```c++
class Lock {
public:
  virtual void lock() = 0;
  virtual void unlock() = 0;
};
```

We now formalize the properties that a good `Lock` algorithm should satisfy. Let $CS_{A}^{j}$ be the interval during which $A$ executes the critical section for the $j$-th time. Let us assume, for simplicity, that each thread acquires and releases the lock infinitely often, with other work taking place in the meantime.

+ *Mutual Exclusion* Critical sections of different threads do not overlap. For threads $A$ and $B$, and integers $j$ and $k$, either $CS_{A}^{k} \to CS_{B}^{j}$ or $CS_{B}^{j} \to CS_{A}^{k}$.
+ *Freedom from Deadlock* If some thread attempts to acquire the lock, then some thread will succeed in acquiring the lock.
+ *Freedom from Starvation* Every thread that attempts to acquire the lock eventually succeeds.

## 2.3 2-Thread Solutions

We begin with two inadequate but interesting `Lock` algorithms.

### 2.3.1 The LockOne Class

Below shows the `LockOne` algorithm. Our 2-thread lock algorithms follow the following conventions: the threads have ids 0 and 1, the calling thread has $i$, and the other $j = i - 1$. Each thread acquires its index by calling `ThreadID.get()`.

```c++
class ThreadID {
public:
  static int get();
};

class LockOne: public Lock {
private:
  volatile bool flag[2] {};
public:
  virtual void lock() override {
    int i = ThreadID::get();
    int j = 1 - i;
    flag[i] = true;
    while (flag[j]) {}
  }
  virtual void unlock() override {
    int i = ThreadID::get();
    flag[i] = false;
  }
};
```

We use $\text{write}_{A}(x = v)$ to denote the event in which $A$ assigns value $v$ to field $x$, and $\text{read}_{A}(v == x)$ to denote the event in which $A$ reads $v$ from field $x$. Sometimes we omit $v$ when the value is unimportant.

*Lemma* 2.3.1. The `LockOne` algorithm satisfies mutual exclusion.

*Proof*: Suppose not. Then there exist integers $j$ and $k$ such that $CS_{A}^{j} \nrightarrow CS_{B}^{j}$ and $CS_{B}^{k} \nrightarrow CS_{A}^{j}$. Consider each thread's last execution of the `lock()` method before entering its $k$-th ($j$-th) critical section.

Inspecting the code, we see that

$$
\begin{align}
&\text{write}_{A}(\text{flag}[A] = true) \rightarrow \text{read}_{A}(\text{flag}[B] == false) \rightarrow CS_{A} \\
&\text{write}_{B}(\text{flag}[B] = true) \rightarrow \text{read}_{B}(\text{flag}[A] == false) \rightarrow CS_{B} \\
\end{align}
$$

Note that once $\text{flag}[B]$ is set to *true* it remains *true*. If follows the following holds, since otherwise thread $A$ could not have read $\text{flag}[B]$ as *false*.

$$
\text{read}_{A}(\text{flag}[B] == false) \rightarrow \text{write}(\text{flag}[B]=true)
$$

By combine the above two equations, we could get:

$$
\begin{split}
\text{write}_{A}(\text{flag}[A] = true) \rightarrow \text{read}_{A}(\text{flag}[B] == false) \rightarrow  \\
\text{write}_{B}(\text{flag}[B] = true) \rightarrow \text{read}_{B}(\text{flag}[A] == false)
\end{split}
$$

It follows that $\text{write}_{A}(\text{flag}[A] = true) \rightarrow \text{read}_{B}(\text{flag}[A] == false)$ without an intervening write to the $\text{flag}[]$ array, a contradiction.

The `LockOne` algorithm is inadequate because it deadlocks if thread executions are interleaved. If $\text{write}_{A}(\text{flag}[A] = true)$ and $\text{write}_{B}(\text{flag}[B] = true)$ events occur before $\text{read}_{A}(\text{flag}[B])$ and $\text{read}_{B}(\text{flag}[A])$ events, then both threads wait forever. Nevertheless, `LockOne` has an interesting property: if one thread runs before the other, no deadlock occurs, and all is well.
