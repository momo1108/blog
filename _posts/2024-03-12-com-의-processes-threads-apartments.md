---
layout: post
title: COM 의 Processes, Threads, Apartments
date: '2024-03-12 16:58:48 +0900'
category: [Windows, COM]
tags: [process, thread, apartment]
---

# COM 의 Processes, Threads, and Apartments
## Process 와 Thread
*process* 란 가상 메모리 공간, 코드, 데이터, 시스템 리소스 들의 모음이다. *thread* 는 프로세스 내에서 순차적으로 실행되는 코드이다. 따라서 processor(CPU) 는 process 가 아닌 thread 를 실행시키는 것이다.

따라서 하나의 Application 은 최소 1개의 process 를 가지고, 하나의 process 는 최소 1개의 thread(primary thread) 부터 추가적인 여러개의 thread 를 가질 수 있다.

Process 들 간에는 메세지를 통해 통신을 하는데, 마이크로소프트의 RPC(Remote Procedure Call) 기술을 사용해 정보를 전달한다. 이때, 다른 process 가 어느 Machine(Local, Remote) 에서 호출하던간에 딱히 다를것 없다.

Thread 가 실행되고 나면, 해당 thread 가 kill 되거나 더 우선순위가 높은 thread(user action 혹은 kernel 의 thread scheduler 로 인한) 가 interrupt 하기 전까지 계속 실행된다. 각각의 thread 가 서로 다른 부분의 코드를 실행할 수도 있고, 여러 thread 가 하나의 코드를 동시에 실행할 수도 있다. 하나의 코드를 동시에 실행하는 경우 각 thread 는 각자 자기만의 stack 을 따로 가지기 때문에 자체적인 실행 흐름을 갖는다. 같은 process 안에 있는 thread 들은 해당 process 의 global variable 이나 resource 를 공유한다.

thread scheduler 는 언제, 얼마나 thread 를 실행할지 결정하는데, process 의 priority class 속성과 thread 의 base priority 를 조합하여 결정한다. process 의 priority class attribute 는 [SetPriorityClass](https://learn.microsoft.com/en-us/windows/desktop/api/processthreadsapi/nf-processthreadsapi-setpriorityclass){: target='_blank' } 함수를 호출해 설정할 수 있고, thread 의 base priority 는 [SetThreadPriority](https://learn.microsoft.com/en-us/windows/desktop/api/processthreadsapi/nf-processthreadsapi-setthreadpriority){: target='_blank' } 함수를 호출해 설정할 수 있다.

multi thread 를 사용하는 application 이 주의할 두가지 threading problem 이 있다.

deadlock
: 각각의 thread 가 다른 thread 의 동작을 기다리느라 대기하는 상태. *COM call control* 이 두 object 사이의 call 에서 발생하는 deadlock 을 막아준다.

race condition
: 한 thread 가 값을 참조하는 다른 thread  보다 먼저 완료됐을 때, 해당 thread 는 유효한 값을 제공하지 못하는 시점이기 때문에 제대로 초기화되지 않은 값을 사용하게 된다. COM 에서는 out-of-process servers[^fn1] 안의 race condition 을 피하기 위해 특별히 디자인된 함수를 지원한다. 구현을 위한 [도큐먼트(Out-of-Process Server Implementation Helpers)](https://learn.microsoft.com/en-us/windows/win32/com/out-of-process-server-implementation-helpers){: target='_blank' } 도 있다.

---

## Apartment 와 COM Threading Architecture
COM 의 초기에는 single-thread-per-process(process 당 하나의 thread) 모델이 보편적이었다. 이후 Multiple thread 동시 실행이 발표되었고, multiple threads 의 이점을 살려 다른 thread 가 긴 시간이 걸리는 작업을 완료하는 동안 동시에 thread 를 실행할 수 있게 되었다. 이로인해 효율적인 application 을 개발할 수 있게 되었다.

> multiple threads 를 사용하는것이 반드시 더 나은 성능을 보장하지는 않는다. thread factoring(쓰레드 관리) 는 어려운 문제이기 때문에, multiple threads 를 사용하는것은 대개 성능 문제를 유발할 수 있다. 그러니 내가 하는 작업을 정확히 알고 사용하는 것이 중요하다.
{: .prompt-warning }



---

[^fn1]: 다른 독립된 프로세스로서 실행되는 프로그램(서버). COM 에서는 프로세스의 경계를 넘어서 서버-클라이언트 간의 통신이 가능하다. 예를들면 exe 파일이 out-of-process server 이다. 반대로 in-process server 는 다른 프로그램안에서 서비스로서 실행되는 프로그램이다. 대표적으로 COM object 와 DLL 이 있다.