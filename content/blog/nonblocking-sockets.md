---
external: false
draft: false
title: "Non-Blocking Sockets and I/O Multiplexing with epoll in C"
description: "This guide explains the concept of non-blocking sockets and how to use `epoll` for I/O multiplexing in C to build high-performance network applications."
date: 2025-07-01
---
![cHeaderBanner](/images/nonblock.png "Banner")

This guide explains the concept of non-blocking sockets and how to use `epoll` for I/O multiplexing in C to build high-performance network applications.

## 1. Blocking vs. Non-Blocking Sockets

### Blocking Sockets (The Default)

By default, socket operations in C are **blocking**. This means that when you call a function like `accept()`, `read()`, or `write()`, your program will halt and wait until that operation is complete.

*   `accept(server_fd, ...)`: Blocks until a new client connects.
*   `read(client_fd, ...)`: Blocks until there is data to be read from the client.
*   `write(client_fd, ...)`: Blocks until the data has been sent to the kernel's buffer.

**The Problem:** A server with a blocking socket can only handle one client at a time. If it's stuck waiting for one client to send data, it cannot accept new connections or service other clients.

The traditional solutions are:
1.  **Multi-threading:** Dedicate a thread to each client. This is resource-intensive and can be complex to manage.
2.  **Multi-processing (`fork()`):** Create a new process for each client. This is even more resource-intensive than threading.

### Non-Blocking Sockets

A non-blocking socket will never halt your program. If an operation cannot be completed immediately, it will return an error (`EAGAIN` or `EWOULDBLOCK`) instead of waiting.

*   `accept()`: If no clients are waiting, it returns immediately with an error.
*   `read()`: If there's no data to read, it returns immediately with an error.
*   `write()`: If the kernel's send buffer is full, it returns immediately with an error.

**How to set a socket to non-blocking:**

You can use the `fcntl()` (file control) function to change the properties of a file descriptor.

```c
#include <fcntl.h>
#include <unistd.h>

int set_nonblocking(int sockfd) {
    int flags = fcntl(sockfd, F_GETFL, 0);
    if (flags == -1) {
        perror("fcntl(F_GETFL)");
        return -1;
    }
    if (fcntl(sockfd, F_SETFL, flags | O_NONBLOCK) == -1) {
        perror("fcntl(F_SETFL)");
        return -1;
    }
    return 0;
}
```

**The New Problem:** If you just set a socket to non-blocking and try to read from it in a loop (`while(1) { read(...) }`), you will spin the CPU at 100% usage, constantly getting the `EAGAIN` error. This is called a "busy-wait" and is very inefficient.

This is where I/O multiplexing comes in.

## 2. I/O Multiplexing and `epoll`

I/O multiplexing allows you to monitor multiple file descriptors (sockets) at the same time and get notified when one of them is ready for an I/O operation (e.g., ready to be read from or written to).

`epoll` is a modern and efficient I/O multiplexing API on Linux. It is more scalable than its predecessors (`select()` and `poll()`) because its performance doesn't degrade as the number of monitored file descriptors increases.

### The `epoll` API

The `epoll` API consists of three main functions:

1.  **`epoll_create1(0)`**: Creates a new `epoll` instance. It returns a file descriptor that represents the `epoll` instance. The `0` flag is for future compatibility.

    ```c
    #include <sys/epoll.h>

    int epoll_fd = epoll_create1(0);
    if (epoll_fd == -1) {
        perror("epoll_create1");
        exit(EXIT_FAILURE);
    }
    ```

2.  **`epoll_ctl(epoll_fd, op, target_fd, &event)`**: Adds, modifies, or removes file descriptors from the `epoll` instance's "interest list".

    *   `epoll_fd`: The file descriptor for the `epoll` instance.
    *   `op`: The operation to perform:
        *   `EPOLL_CTL_ADD`: Add `target_fd` to the interest list.
        *   `EPOLL_CTL_MOD`: Modify the events for `target_fd`.
        *   `EPOLL_CTL_DEL`: Remove `target_fd` from the interest list.
    *   `target_fd`: The file descriptor you want to monitor (e.g., your server socket or a client socket).
    *   `&event`: A pointer to a `struct epoll_event`. This structure tells `epoll` what events you are interested in for `target_fd`.

    The `epoll_event` structure looks like this:

    ```c
    struct epoll_event {
        uint32_t     events;      /* Epoll events */
        epoll_data_t data;        /* User data variable */
    };
    ```

    *   `events`: A bitmask of events. Common ones include:
        *   `EPOLLIN`: The associated file is available for `read()` operations.
        *   `EPOLLOUT`: The associated file is available for `write()` operations.
        *   `EPOLLET`: Sets Edge-Triggered behavior (more on this later).
    *   `data`: This is for you to use. You can store anything here. It's common to store the file descriptor itself (`event.data.fd = target_fd;`) or a pointer to a struct containing client state.

3.  **`epoll_wait(epoll_fd, events, max_events, timeout)`**: This is the core of the event loop. It waits for events on the file descriptors in the interest list.

    *   `epoll_fd`: The `epoll` instance file descriptor.
    *   `events`: A pointer to an array of `struct epoll_event` that will be filled with information about the events that have occurred.
    *   `max_events`: The maximum number of events `epoll_wait` should return in a single call.
    *   `timeout`: The maximum time to wait in milliseconds. A value of `-1` means wait indefinitely. A value of `0` means return immediately, even if there are no events.

    `epoll_wait` returns the number of file descriptors ready for the requested I/O, or `0` if it timed out, or `-1` on error.

## 3. Tutorial: Building a Non-Blocking Server with `epoll`

Here is a step-by-step guide to the logic of an `epoll`-based server.

### Step 1: Setup the Listening Socket

This part is standard socket programming, with one addition: setting the socket to non-blocking.

```c
// 1. Create socket
int server_fd = socket(AF_INET, SOCK_STREAM, 0);
// ... error checking ...

// 2. Set socket options (e.g., SO_REUSEADDR)
// ...

// 3. Bind to an address and port
// ...

// 4. Set the socket to be non-blocking
set_nonblocking(server_fd);

// 5. Listen for connections
listen(server_fd, SOMAXCONN);
```

### Step 2: Create `epoll` Instance and Add the Server Socket

Now, create the `epoll` instance and tell it you are interested in `read` events (`EPOLLIN`) on your listening socket. A "read" event on a listening socket means a new client is trying to connect.

```c
int epoll_fd = epoll_create1(0);
// ... error checking ...

struct epoll_event event;
event.events = EPOLLIN; // We are interested in read events
event.data.fd = server_fd;

if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, server_fd, &event) == -1) {
    perror("epoll_ctl: server_fd");
    exit(EXIT_FAILURE);
}
```

### Step 3: The Event Loop

This is the heart of your server.

```c
#define MAX_EVENTS 10
struct epoll_event events[MAX_EVENTS];

while (1) {
    int n_events = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
    if (n_events == -1) {
        perror("epoll_wait");
        break;
    }

    // Loop through all the events that have occurred
    for (int i = 0; i < n_events; i++) {
        // Event handling logic goes here...
    }
}
```

### Step 4: Handling Events

Inside the `for` loop, you need to figure out what caused the event.

#### Case 1: New Client Connection

If the event's file descriptor is your `server_fd`, it means a new client is connecting. You should `accept()` it.

**Crucially**, because the listening socket is non-blocking, you should call `accept` in a loop to accept all pending connections for that one event.

```c
if (events[i].data.fd == server_fd) {
    // New connection
    while (1) {
        struct sockaddr_in client_addr;
        socklen_t client_len = sizeof(client_addr);
        int client_fd = accept(server_fd, (struct sockaddr *)&client_addr, &client_len);

        if (client_fd == -1) {
            if (errno == EAGAIN || errno == EWOULDBLOCK) {
                // We have processed all incoming connections.
                break;
            } else {
                perror("accept");
                break;
            }
        }

        // Set the new client socket to non-blocking
        set_nonblocking(client_fd);

        // Add the new client socket to the epoll interest list
        event.events = EPOLLIN | EPOLLET; // Monitor for read events, edge-triggered
        event.data.fd = client_fd;
        if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, client_fd, &event) == -1) {
            perror("epoll_ctl: client_fd");
            close(client_fd);
        }
    }
}
```

#### Case 2: Data from a Client

If the event is not for the `server_fd`, it must be from a client. The `EPOLLIN` flag means the client has sent data.

```c
else {
    // Data from an existing client
    int client_fd = events[i].data.fd;
    char buffer[1024];
    ssize_t count;

    // Read data from the client
    while ((count = read(client_fd, buffer, sizeof(buffer))) > 0) {
        // Process the data (e.g., echo it back, parse an FTP command)
        write(STDOUT_FILENO, buffer, count);
    }

    if (count == 0) {
        // Client closed the connection
        printf("Client %d disconnected\n", client_fd);
        close(client_fd); // This automatically removes it from epoll
    } else if (count == -1) {
        // If errno is EAGAIN, that means we have read all data.
        // So we can continue to the next event.
        if (errno != EAGAIN) {
            perror("read");
            close(client_fd);
        }
    }
}
```

## 4. Edge-Triggered (ET) vs. Level-Triggered (LT)

`epoll` has two modes for delivering events:

*   **Level-Triggered (LT) - The Default:** `epoll_wait` will continuously report an event as long as the condition holds. For example, if a socket has data to be read, `epoll_wait` will *always* return that socket as ready-to-read until you have read *all* the data from its buffer. This is simpler to use but can be less performant.

*   **Edge-Triggered (ET) - `EPOLLET`:** `epoll_wait` will only report an event *once* when the state changes. For example, it will tell you a socket is ready-to-read only when new data *first arrives*. It will not notify you again until more new data arrives.

**Why use Edge-Triggered?**
ET is more efficient. It prevents `epoll_wait` from constantly reminding you about an event you haven't fully handled yet.

**The Catch with ET:**
When you get an ET notification, you **must** process the file descriptor until it would block (i.e., `read()` or `write()` returns `EAGAIN`). If you only read part of the data, `epoll` will not notify you again, and the remaining data will sit in the buffer forever. This is why the `while` loops around `accept()` and `read()` in the examples above are so important when using `EPOLLET`.

## Summary

1.  **Server-Side:**
    *   Create a listening socket, make it non-blocking.
    *   Create an `epoll` instance.
    *   Add the listening socket to `epoll` (`EPOLLIN`).
    *   Loop with `epoll_wait()`.
    *   If `epoll_wait()` returns your listening socket, `accept()` all new clients, make their sockets non-blocking, and add them to `epoll` (`EPOLLIN | EPOLLET`).
    *   If `epoll_wait()` returns a client socket, `read()` all data from it until you get `EAGAIN`. Parse the FTP command and prepare a response.
    *   To send a response (or a file), you might need to watch for `EPOOLOUT` events if the initial `write()` call doesn't send all the data at once.

2.  **Client-Side:**
    *   The logic is similar but simpler.
    *   Create a socket, make it non-blocking.
    *   `connect()` to the server. For a non-blocking socket, `connect` will likely return immediately with `EINPROGRESS`.
    *   Create an `epoll` instance. Add your socket to it, watching for `EPOLLOUT` events. An `EPOLLOUT` event on a connecting socket indicates that the connection has been established.
    *   Once connected, you can use `epoll` to manage reading responses from the server (`EPOLLIN`) and writing commands to the server (`EPOLLOUT`).

This approach is a significant conceptual shift from a simple blocking model but is the foundation for all modern, high-performance network servers.

# A Deep Dive into Non-Blocking Sockets and `epoll`

## Visualizing the `epoll` Workflow

**Application (Your Program)** and the **Kernel (Operating System)**.

1.  **`epoll_create()`**: The Application sends a request to the Kernel, which creates a special `epoll` instance. Think of this as an empty "watch list" inside the Kernel. The Kernel gives back a file descriptor (`epoll_fd`) to the Application as a handle to this list.

2.  **`epoll_ctl(ADD, ...)`**: The Application tells the Kernel to add sockets to the watch list using `epoll_ctl`. For example, it adds the main `server_fd` to watch for new connections. Each time a new client connects, the new `client_fd` is also added to this list.

3.  **`epoll_wait()`**: The Application calls `epoll_wait()` and goes to sleep. It's now waiting for the Kernel to wake it up.

4.  **An Event Occurs**: A new client sends data. The data arrives at the network card and is processed by the Kernel. The Kernel sees that the data is for one of the sockets in the `epoll` watch list.

5.  **The Wake-Up**: The Kernel "wakes up" the Application from its `epoll_wait()` sleep and tells it which socket(s) are ready for I/O.

6.  **Processing**: The Application, now awake, loops through the ready sockets. It reads the data from the client socket, processes the FTP command, and can then continue its work.

This model allows a single-threaded server to efficiently handle many clients at once without getting stuck waiting on any single one.

---
