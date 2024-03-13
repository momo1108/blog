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

일반적으로 COM threading 구조를 이해하는 가장 간단한 방법은 모든 COM obejct 를 아파트먼트(apartment)라는 그룹으로 나눈다고 생각하는 것이다. 이에 맞게 COM object 의 메서드는 실행되는 한 apartment 에 소속된 thread 에서만 직접적으로 호출되는데, 이러한 관점으로 보면 COM obejct 는 단 하나의 apartment 에만 속한다. 한 apartment 의 thread 가 다른 apartment 의 object method 를 호출하기 위해서는 proxy[^fn2] 를 통해야한다.

> C 나 C++, C# 에서 다른 apartment 의 COM object method 를 호출할 때 proxy 를 사용하는 부분은 COM 인프라에서 자동으로 처리되고, proxy 를 명시적으로 호출하거나 하진 않는다. Proxy 에 대한 자세한 내용은 해당 [도큐먼트(Proxy)](https://learn.microsoft.com/en-us/windows/win32/com/proxy){: target='_blank' } 를 참조하자.
{: .prompt-tip }

이러한 apartment 에는 2가지 종류가 있는데, *STA(single-threaded apartments)* 와 *MTA(multi-threaded apartments)* 가 있다.

STA(single-threaded apartments)
: 하나의 thread 로 이루어져있다. 따라서 STA 에 속한 모든 COM object 의 method 는 그 하나의 thread 만이 호출할 수 있다. STA 내부의 모든 COM object 의 method 호출은 STA thread 의 windows message queue 를 통해 동기적으로 처리된다. 하나의 실행 thread 를 가진 process 는 이 STA 의 한 케이스이다.

> 마지막 문장이 이해가 잘 안되네?
>
> 여기서 말하는 *하나의 실행 thread 를 가진 process(process with a single thread of execution)* 는 **process** 의 개념으로 단 하나의 thread 만 가지고 있기 때문에 자연스럽게 STA 가 적용되는 특정 케이스이다. STA 는 **apartment** 의 개념으로 COM의 특정 스레딩 모델 중 하나로, 모든 COM object 가 특정 thread 에서만 호출을 받을 수 있는 apartment 를 의미한다.
{: .prompt-info }

MTA(multi-threaded apartments)
: 하나 이상의 thread 로 이루어져있다. 따라서 MTA 에 속한 모든 COM object 들의 method 는 MTA 에 속한 모든 thread 에서 직접적으로 호출할 수 있다. MTA 에 속한 thread 들은 *free-threading* 이라는 모델을 사용한다. free-threading 모델은 MTA 의 다중 thread 환경에서 한 COM object 에 여러 thread 가 동시에 method 를 호출할 때, COM object 내부적으로 안전한 동기화 메커니즘을 구현해 데이터의 안전성을 보장해준다.

> 한 process 의 STA 와 MTA 사이의 통신에 대한 설명을 보고싶으면, 다음 [도큐먼트(Single-Threaded and Multithreaded Communication)](https://learn.microsoft.com/en-us/windows/win32/com/single-threaded-and-multithreaded-communication){: target='_blank' } 를 참조하자.
{: .prompt-tip }

Process 는 **0개 이상의 STA** 와 **0개 혹은 1개의 MTA** 를 가질 수 있다.

Process 내부적으로 main apartment 의 초기 설정이 최우선이다. single-threaded process(STA 아님) 에서는 단 하나의 apartment 만을 가지고, 이게 바로 main apartment 가 된다.

서로 다른 apartment 간에 호출할 때 thread 간에 전달되는 매개변수(데이터)는 marshaling[^fn3] 을 통해 적절한 형식으로 변환되며, COM 은 메세지를 통해 apartment 간의 호출(STA ↔ MTA, STA ↔ STA)을 동기처리한다. Process 내부의 여러개의 thread 를 free-threaded 로 지정하면 해당 thread(free threads) 들은 하나의 apartment 로 속하게 되며(MTA 를 뜻하는듯), 매개변수는 apartment 내의 모든 thread 에 직접적으로 전달되니 반드시 모든 동기화를 직접 처리해야 한다. 

free-threading 과 apartment threading 을 모두 사용하는 process 의 경우, free thread 들은 모두 하나의 apartment(MTA 를 뜻하는듯) 에 속하며, 나머지 apartment 들은 모두 STA 이다. COM 작업을 수행하는 process 는 최대 1개의 MTA 와 여러 STA 들의 모음인 것이다.

COM 내부의 threading model 은 서로 다른 threading 구조를 가진 server-client 간의 작업을 위한 메커니즘을 지원해준다. 다른 process 에서 다른 threading model 을 사용하는 objects 간의 호출은 기본적으로 지원된다. 호출을 요청하는 object 시점에서, 외부 process 의 object 에 대한 모든 호출은 그 object 의 threading model 에 상관없이 동일하게 작동한다. 또 그 반대로 호출을 받는 object 시점에서 외부 process 의 object 가 어떤 threading model 을 사용하던 모두 동일하게 동작한다.

client 와 외부 process 의 object 간의 호출은 다른 threading model 을 사용한다 해도 그렇게 복잡하지 않다. COM 자체적으로 client 와 server 사이의 중재자처럼 상호운용을 위해서 표준 마샬링과 RPC 를 사용한 threading model 코드를 제공해주기 때문이다. 예를 들어, 여러 multiple free-threaded clients 들이 서버의 single-threaded object 를 동시에 호출하면, 서버의 COM 이 여러 호출들에 해당하는 window message 들을 message queue 에 배치하여 동기처리를 한다. 서버에 있는 apartment 는 그 message 들을 하나씩 찾아서 처리하면서 message 안의 호출을 꺼내서 받게된다. in-process servers 가 client 와 적절하게 통신하기 위해서 주의할 점들이 몇가지 있다. 다음 [도큐먼트(In-Process Server Threading Issues)](https://learn.microsoft.com/en-us/windows/win32/com/in-process-server-threading-issues){: target='_blank' } 를 참조하자.

multithreaded model 을 사용한 개발에 가장 중요한 점은 코드를 thread-safe 하게 작성하는 것이다. 다시 말해 특정 thread 를 위한 message 는 반드시 그 thread 로만 전달되도록 하는게 thread 들에 대한 접근을 안전하게 보호하는 것이다.

---

[^fn1]: 다른 독립된 프로세스로서 실행되는 프로그램(서버). COM 에서는 프로세스의 경계를 넘어서 서버-클라이언트 간의 통신이 가능하다. 예를들면 exe 파일이 out-of-process server 이다. 반대로 in-process server 는 다른 프로그램안에서 서비스로서 실행되는 프로그램이다. 대표적으로 COM object 와 DLL 이 있다.
[^fn2]: 여기서 프록시의 역할은 다른 apartment 에 속한 thread 안의 COM object method 를 호출할 수 있고, 그 결과를 다시 호출한 thread 에 반환하는 역할이다.
[^fn3]: 자세한 내용은 다음 링크 참조. <https://devblogs.microsoft.com/oldnewthing/20151020-00/?p=91321>{: target='_blank' }