### 0. Process Model의 한계
* 이전까지는 프로세스의 구조, Switching 등에 대해서 배웠고, 프로세스 간 통신을 어떻게 할 것인가에 대한 논의를 해보았다. 그리고 지금까지 살펴본 프로세스 모델은 아래와 같은 두 가지 특징(한계)이 있었다.
  * 1. Cooperating Processes: 어플리케이션은 동시에 요청되는 여러가지 상황에서 fork()를 통해 Child process를 만든다. 예를 들어 웹 서버는 각 클라이언트의 요청에 대해 서버가 child process를 생성하여 클라이언트의 요구를 처리하는 방식이었다. 하지만 요청이 많아질수록 서버 메모리는 이에 따라 증가할 수밖에 없으며 주소공간과 자원 역시 공유되어 비효율적이었다.
  * 2. 전통적인 프로세스 기반 모델은 Multiprocessor architecture의 장점을 살리지 못했다. 왜냐? 각 프로세스는 오직 하나의 프로세서(CPU)에서 밖에 돌지 못했기 때문이다. 따라서 N개의 CPU가 존재한다고 하더라도, 각 프로세스는 오직 1개의 CPU에서밖에 돌지 못하기 때문에, 1/N으로 나누어서 일을 처리할 수 없었다.
* 따라서, 위와 같은 프로세스 기반 모델의 특징 때문에 메모리 공간의 한계나 속도의 비효율성 문제가 생겼고, Multi-threading의 필요성이 제기됐다.


### 1. Multi-Thread란?
* 이전에 배웠던 Process의 Image를 다시 한 번 살펴보자.   
![image](https://user-images.githubusercontent.com/61929745/111899320-3deedf80-8a6f-11eb-8763-485e6a56e5ea.png)  

* 프로세스 이미지는 *크게 세 파트* 로 구성되어 있었다. 
  * 1. Process text : Program context - Data registers, Stack Pointer(SP), Program Counter(PC)
  * 2. Code, Data and stacks - read-only code/data, read/write data, run-time heap, shared Libraries, *user Stack* 등
  * 3. Kernel Context - VM structures, Open files, signal handlers 등

* 멀티스레딩이란 기본적으로 위의 프로세스 이미지를 **공유 가능한 부분** 과 **공유 가능하지 않은 부분** 으로 나눠서 공유 가능한 부분을 **Thread**란 개념으로 나눈 것이다. 다시 말해서,
  * - 공유 가능한 Shared libraries, runtime heap, read/write data, read-only code/data 그리고 Kernel Context는 하나로 두고
  * - 특정 시점의 Program Counter(PC)에 따라 Stack 내용이 달라지기 때문에 개별적으로 가지고 있어야 하는 User stack 부분으로 나누는 것.   
* 아래 그림을 보면 더 쉽게 이해가 될 것 같다.   
![image](https://user-images.githubusercontent.com/61929745/112791025-963f6600-909b-11eb-8e2a-84736774a616.png)   
출처: https://goodgid.github.io/What-is-Multi-Thread/

* 즉, code 부분에서 각 Program counter가 수행하는 부분을 T1, T2, T3로 나눌 수 있고, 각 PC에 필요한 데이터들을 유저 스레드의 Stack에 정의해서 병렬적으로 처리하는 것이다.

### 2. Process VS Thread 
* 웹 서버에서 Process 기반 유저 요청 처리 방식과 Thread 기반 유저 요청 처리 방식을 비교해보자.

```c
### 프로세스 기반 방식 - 클라이언트에게 요청이 오면 child를 fork()해서 요청 처리 ###
while(1){
  int sock = accept();
  if((pid = fork()) == 0){
    /* Handle client request */
  }else{
    /* close socket */
  }
}
```

```c
### 멀티 스레드 기반 방식 - 클라이언트에게 요청이 오면 새로운 Thread를 생성하여 처리
webserver()
{
  while(1) {
    int sock = accept();
     thread_fork(handle_request, sock);
  }
}
handle_request(int sock)
{
  /* Process request */
  close(sock);
}
```

* 멀티스레딩은 시간적으로, 메모리적으로 더 효율적인 방법이다. 하지만 멀티스레딩에는 한 가지 큰 문제가 있다.
  * - 스레드 간 공유를 할 때 Global data(static data) 관리가 어렵다는 것
  * - 스레드 A, B가 어떤 data를 읽고 있었는데, 다른 스레드 C가 이 데이터를 write 하려고 하면?? - Data integrity 문제 발생
- Synchronization 이슈가 발생한다. 이에 대해서는 다음에 다시 다룰 것이다.
