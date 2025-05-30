# Async Scheduler

**Async Scheduler** is a lightweight asynchronous task scheduler and runtime written in C.

Inspired by Rust's `std::future`, `async/await`, and `tokio`-style runtimes, it demonstrates how to implement efficient asynchronous programming in a garbage collector–free environment.

## Features

- **Manual implementation of Future and async/await**: Recreates Rust's Future trait and async/await semantics in C.
- **Coroutine-based task system**: Stackless coroutine model for task scheduling.
- **epoll-based asynchronous I/O**: Efficient I/O event handling using Linux's epoll.
- **Graceful shutdown**: Cleans up all resources safely before server termination.

## System Design

![system architecture](docs/images/architecture.png)

---

## Build

### Quick Build

```bash
source BUILD
```

### Manual Build

Configure:

```bash
export CC=clang
export CFLAGS="-g -O2 -Wall -Wextra"

autoreconf -i
mkdir -p build
cd build
../configure
```

Build:

```bash
make
# or, with compilation database
bear -- make
```

### Clean Build

```bash
cd build
make clean
make distclean
```

---

## Run

### Server

```bash
./build/server
```

Example output:

```bash
Welcome to the Echo Server.
Usage: telnet localhost 10000
Press Ctrl+C to stop the Server.
Accept (6): 127.0.0.1
Echo (6): Hello?
Accept (7): 127.0.0.1
Echo (7): Good evening!
Echo (7): Goodbye.
Close (7)
Echo (6): Bye.
Close (6)
^C[SIG: 2]
Executor Stop.
Clean (5)
Goodbye.
```

### Client

#### Telnet

```bash
telnet localhost 10000
```

```bash
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
Hello?
Hello?
Bye.
Bye.
^]
telnet> quit
Connection closed.
```

#### Netcat

Disconnect immediately after sending data to the server.
In this case, the connection is terminated before the client's fd is registered with the server.

```bash
echo "Hello?" | nc -q 0 localhost 10000
```

---

## Valgrind

### Compile for Debugging

```bash
export CC=clang
export CFLAGS="-gdwarf-4 -O0"
autoreconf -i
mkdir -p build && cd build
../configure
bear -- make
```

### Run with Valgrind

```bash
valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes ./build/server
```

### Example Output with Valgrind

```log
==49402== Memcheck, a memory error detector
==49402== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==49402== Using Valgrind-3.18.1 and LibVEX; rerun with -h for copyright info
==49402== Command: ./build/server
==49402==
Welcome to the Echo Server.
Usage: telnet localhost 10000
Press Ctrl+C to stop the Server.
Accept (6): 127.0.0.1
Echo (6): hi
Close (6)
^C[SIG: 2]
Executor Stop.
Clean (5)
Goodbye.
==49402==
==49402== HEAP SUMMARY:
==49402==     in use at exit: 0 bytes in 0 blocks
==49402==   total heap usage: 57 allocs, 57 frees, 11,194 bytes allocated
==49402==
==49402== All heap blocks were freed -- no leaks are possible
==49402==
==49402== For lists of detected and suppressed errors, rerun with: -s
==49402== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

---

## Code Formatting

- torvalds/linux: [.clang-format](https://github.com/torvalds/linux/blob/master/.clang-format)

```bash
find . -iname '*.h' -o -iname '*.c' | xargs clang-format -i
```

