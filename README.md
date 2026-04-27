*This project has been created as part of the 42 curriculum by mcuello.*
# 🍝 Philosophers

> *"One cannot think well, love well, sleep well, if one has not dined well."* — Virginia Woolf

![42 Badge](https://img.shields.io/badge/42-Philosophers-blue?style=for-the-badge&logo=42&logoColor=white)
![Language](https://img.shields.io/badge/Language-C-lightgrey?style=for-the-badge&logo=c)
![Threads](https://img.shields.io/badge/Concurrency-Threads-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Finished-brightgreen?style=for-the-badge)

---

## Table of Contents

- [Philosophers](#-philosophers)
	- [About](#-about)
	- [The Problem](#-the-problem)
	- [How It Works](#️-how-it-works)
		- [Thread Safety](#thread-safety)
	- [Features](#-features)
	- [Usage](#-usage)
		- [Compilation](#compilation)
		- [Running](#running)
		- [Arguments](#arguments)
		- [Examples](#examples)
		- [Output Format](#output-format)
	- [Implementation Details](#-implementation-details)
		- [Project Structure](#project-structure)
	- [Testing Tips](#-testing-tips)

---

## About

This project is a simulation of the classic **Dining Philosophers Problem**, a famous concurrency problem formulated by Edsger W. Dijkstra. The goal is to learn about threading, mutexes, and the challenges of concurrent programming — particularly **race conditions**, **deadlocks**, and **data races**.

---

## The Problem

A group of philosophers sit around a circular table. Between each pair of adjacent philosophers lies a fork. A philosopher can only eat if they hold **both** the left and right forks. Philosophers alternate between three states:

```
🤔 THINKING  →  🍝 EATING  →  😴 SLEEPING  →  🔁 (repeat)
```

- There is one fork per philosopher on the table.
- A philosopher must grab both adjacent forks to eat.
- After eating, they put the forks back and sleep, then think again.
- If a philosopher doesn't eat within `time_to_die` milliseconds, they die.
- The simulation ends when a philosopher dies, **or** when all philosophers have eaten `number_of_times_each_philosopher_must_eat` times (if specified).

---

## How It Works

Each philosopher runs in its own **thread**. Each fork is protected by a **mutex** to prevent simultaneous access. A monitor thread checks for deaths and meal completion conditions.

```
┌─────────────────────────────────────────────────────────┐
│                   Main Thread                           │
│  - Creates N philosopher threads                        │
│  - Creates N mutexes (one per fork)                     │
│  - Starts monitor loop                                  │
└──────────────────────┬──────────────────────────────────┘
                       │ spawns
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
  [Philosopher 1] [Philosopher 2] [Philosopher N]
   think→eat→sleep  think→eat→sleep  think→eat→sleep
```

### Thread Safety

- All shared state is protected with `pthread_mutex_t`
- A dedicated mutex guards the print function to avoid interleaved output
- Death detection uses a shared flag to stop all threads cleanly

---

##  Features

- ✅ No data races (verified with `valgrind --tool=helgrind` and `thread sanitizer`)
- ✅ No deadlocks — even/odd fork-picking strategy prevents circular wait
- ✅ Accurate millisecond timing using `gettimeofday`
- ✅ Clean thread termination — no zombie threads
- ✅ Works correctly with a single philosopher edge case

---

##  Usage

### Compilation

```bash
git clone https://github.com/mancuello/philosophers.git
cd philosophers/philo
make
```

### Running

```bash
./philo number_of_philosophers time_to_die time_to_eat time_to_sleep [number_of_times_each_philosopher_must_eat]
```

### Arguments

| Argument | Description |
|---|---|
| `number_of_philosophers` | Number of philosophers (and forks) |
| `time_to_die` | Time in ms before a philosopher dies without eating |
| `time_to_eat` | Time in ms a philosopher takes to eat |
| `time_to_sleep` | Time in ms a philosopher spends sleeping |
| `[number_of_times_each_philosopher_must_eat]` | *(Optional)* Stop after each philosopher eats this many times |

### Examples

```bash
# 5 philosophers, no one should die
./philo 5 800 200 200

# 4 philosophers, each must eat 7 times
./philo 4 410 200 200 7

# 1 philosopher — will always die (only one fork)
./philo 1 800 200 200
```

### Output Format

```
timestamp_in_ms X has taken a fork
timestamp_in_ms X is eating
timestamp_in_ms X is sleeping
timestamp_in_ms X is thinking
timestamp_in_ms X died
```

---

##  Implementation Details

### Project Structure

```
philosophers/
├── Makefile
├── main.c               # Entry point
├── include/
│   └── philo.h          # Structs, defines, prototypes
└── src/
    ├── aux.c            # Helper functions
    ├── ft_atoi.c        # Custom atoi implementation
    ├── init.c           # Parsing and struct initialization
    ├── monitor.c        # Death and meal-count checking
    ├── philo.c          # Core philosopher logic (eat/sleep/think)
    ├── routine.c        # Philosopher thread routine
    └── threads.c        # Thread creation and management
``` 

---

##  Allowed Functions

`memset` · `printf` · `malloc` · `free` · `write` · `usleep` · `gettimeofday` · `pthread_create` · `pthread_detach` · `pthread_join` · `pthread_mutex_init` · `pthread_mutex_destroy` · `pthread_mutex_lock` · `pthread_mutex_unlock`

---

##  Testing Tips

```bash
# Check for data races (Linux)
gcc -fsanitize=thread -g *.c && ./a.out 5 800 200 200

# Check with helgrind
valgrind --tool=helgrind ./philo 5 800 200 200

# Stress test — should never die
./philo 5 800 200 200

# Should die immediately
./philo 4 310 200 100
```

---
