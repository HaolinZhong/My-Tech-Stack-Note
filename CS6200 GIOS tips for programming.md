# Pthreads

Pthreads 是 POSIX 线程库的缩写，它是在 UNIX 和 Linux 操作系统上进行多线程编程的标准方法。

要使用 Pthreads，您需要在项目中包含 <pthread.h> 头文件。

创建线程的基本步骤如下：

1. 定义线程函数，该函数是线程的主体。
2. 创建 pthread_t 变量来存储线程的标识符。
3. 使用 pthread_create() 函数创建新线程，并传递线程函数和参数。
4. 使用 pthread_join() 函数等待线程结束。

例如:

```c
#include <pthread.h>
#include <stdio.h>

void *thread_function(void *arg) {
    printf("Thread says: %s\n", (char *)arg);
    return NULL;
}

int main() {
    pthread_t thread;
    char *message = "Hello, World!";
    pthread_create(&thread, NULL, thread_function, message);
    pthread_join(thread, NULL);
    return 0;
}

```

这是一个简单的Pthread示例，它创建了一个名为 thread_function 的线程，并在线程中打印一条消息。

注意:

- pthread_create() 第一个参数是指向 pthread_t 类型的指针，用于存储新线程的标识符。
- pthread_join() 接受两个参数，第一个是要等待的线程的标识符，第二个是用于存储线程返回值的指针。
- pthread_create()和pthread_join() 的返回值是错误代码，如果返回值为0，则表示成功，否则表示失败
- Pthreads 还提供了其他有用的函数，例如：
  - pthread_exit()：终止当前线程。
  - pthread_cancel()：取消其他线程。
  - pthread_detach()：将线程设置为分离状态，这样线程结束时会自动释放所有资源。
  - pthread_self()：获取当前线程的标识符。
- Pthreads 还提供了互斥量、条件变量和读写锁等同步机制，用于在多线程环境中管理共享资源的访问。





# Sockets

sockets是网络编程的基础，它允许程序通过网络进行通信。在 C 语言中，sockets 使用 <sys/socket.h> 头文件。

创建 socket 的步骤如下：

1. 使用 socket() 函数创建 socket。
2. 使用 bind() 函数将 socket 绑定到本地地址和端口。
3. 使用 listen() 函数将 socket 设置为监听状态。
4. 使用 accept() 函数接受客户端连接。
5. 使用 recv() 或 send() 函数进行数据传输。
6. 使用 close() 函数关闭 socket。



例如，这是一个简单的 TCP server 示例：

```c
#include <sys/socket.h>
#include <netinet/in.h>
#include <stdio.h>

int main() {
    int server_socket;
    struct sockaddr_in server_address;
    int connection;
    char buffer[256];
    int n;

    //create socket
    server_socket = socket(AF_INET, SOCK_STREAM, 0);

    //specify server address
    server_address.sin_family = AF_INET;
    server_address.sin_addr.s_addr = INADDR_ANY;
    server_address.sin_port = htons(8080);

    //bind the socket to specified IP and port
    bind(server_socket, (struct sockaddr *) &server_address, sizeof(server_address));

    //listen for connections
    listen(server_socket, 5);

    //accept a connection
    connection = accept(server_socket, (struct sockaddr *) NULL, NULL);

    //receive data from the client
    n = recv(connection, buffer, sizeof(buffer), 0);

    printf("Data received: %s\n", buffer);

    //send data to the client
    send(connection, "Hello, World!", 14, 0);

    //close the socket
    close(connection);
    close(server_socket);

    return 0;
}
```





这是一个简单的TCP client示例:

```c
#include <sys/socket.h>
#include <netinet/in.h>
#include <stdio.h>
#include <string.h>
#include <arpa/inet.h>

int main() {
    int client_socket;
    struct sockaddr_in server_address;
    char buffer[256];

    //create socket
	client_socket = socket(AF_INET, SOCK_STREAM, 0);
    //specify server address
    server_address.sin_family = AF_INET;
    server_address.sin_port = htons(8080);
    inet_pton(AF_INET, "127.0.0.1", &server_address.sin_addr);

    //connect to the server
    connect(client_socket, (struct sockaddr *) &server_address, sizeof(server_address));

    //send data to the server
    send(client_socket, "Hello, Server!", 15, 0);

    //receive data from the server
    recv(client_socket, buffer, sizeof(buffer), 0);

    printf("Data received: %s\n", buffer);

    //close the socket
    close(client_socket);

	return 0;
}
```



这只是简单的例子，在实际使用过程中还需要考虑错误处理、超时处理等等。

实际使用中建议使用封装好的库，例如 Berkeley sockets 或者 Winsock ，这样可以使编程更加简单和可维护。





# 多线程server/client

## server

- 原本的server同一时间内只能处理一个请求
- 需要重新设计`handler.c`和修改`gfserver_main.c`
- 采用boss-worker设计
  - boss `accept` client, 然后把`connection`交给底下的worker thread
  - worker thread会设计成一个线程池, pool size由外部传入的参数决定
  - 程序会处理一定数量(由外部传入的参数决定)的请求并分配给线程池中的thread. 当请求全部完成后, boss要关闭所有的worker并退出.
  - 具体的数据结构要求:
    - 要有一个work queue (from `steque.ch`)
    - 至少一个mutex
    - 至少一个condition variable
- 可能需要在handler.c中额外创建一些函数, 这些函数会被`gfserver_main.c`调用 (因此会用到extern keyword)



## client

- modify `gfclient_download.c` 来使其能够同时下载多个文件