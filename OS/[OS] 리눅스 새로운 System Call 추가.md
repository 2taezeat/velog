## # Intro
- `우분투 리눅스에 새로운 Kernel을 컴파일하고 수정해서 새로운 System Call을 추가`하는 수업 프로젝트의 제가 수행한 내용을 소개합니다.
- `기존의 Kernel에 새로운 System Call을 추가한 후에 이를 호출할 수 있는
User-Application 까지를 구현` 하는 것이 목표입니다.
- 새로 추가한 system call은 내부적으로 `Queue 자료구조`를 가지고, 이 Queue에 integer값을
`Enqueue`하거나 `Dequeue`하는 역할을 수행합니다.


## # 리눅스의 시스템 콜
![참고 사진](https://images.velog.io/images/ba159sal/post/fd062636-53ef-4e39-8a73-7a913a6ea765/fork-system-call.png)

`시스템 콜은 CPU가 프로세스를 실행할 때, User mode에서 실행중인 프로세스가 Kernel mode의 권한이 필요한 서비스를 받기 위한 방법을 말합니다. User mode에서 구동되는 프로세스는 CPU에 직접적으로 접근할 수 없는데, CPU에 간접적 접근을 가능케 하기 위해 시스템 콜은 Kernel mode로의 실행 모드를 전환하도록 해줍니다.`

`CPU가 실행 중인 어플리케이션(프로세스)에서, system_call()을 호출하게 되면, 라이브러리에서 해당 System call의 번호를 MOVE 명령어를 통해레지스터에 저장하고, 인터럽트를 발생시킵니다. (즉, 커널은 Wrapper Function을 이용해 0x80의 Trap Instruction이 발생합니다.) User space에서 Kernel Space으로 Trap이 발생하면 IDT를 확인하며, Trap을 처리하기 위한 Handler는 system_call_table의 번호를 확인해 해당 번호에 맞는 system call을 table에서 찾아 호출하고, 특정 값을 반환하여 system call 함수 실행이 종료되었음을 알립니다. 이 루틴을 모두 수행하면, Kernel mode에서 다시 User mode로 돌아 오게 됩니다. 특정 값을 반환 하기 전까지 어플리케이션은 대기 상태에 있습니다.`



## # 과정
### 1. syscall_64.tbl
![](https://images.velog.io/images/ba159sal/post/b782f8fe-3572-4ce8-bf9b-d61cf8a13bde/image.png)

`/usr/src/linux-4.20.11/arch/x86/entry/syscalls/syscall_64.tbl` 파일에 위에 내용을 작성하였습니다. 시스템 콜 루틴에서 `sys_call_table`을 참조하여 시스템 콜을 호출하기에, 64bit 시스템에서 `syscall_64.tbl` 파일에 새롭게 추가하는 `enqueue와 dequeue` 시스템 콜에 해당하는 함수 이름과 번호를 지정해야 하고, `sys_oslab_enqueue`에는 335번을 `sys_oslab_dequeue`에는 336번을 할당하였습니다.


### 2. syscalls.h
![](https://images.velog.io/images/ba159sal/post/fe05fa9b-e761-4148-8d47-fb2a07524276/image.png)
`/usr/src/linux-4.20.11/include/linux/syscalls.h` 파일에 위에 내용을 추가하였습니다.
시스템 콜 함수들의 prototype을 정의하여야만, 헤더파일을 읽고 참조가 가능하게 됩니다. 또한 함수 선언문 앞에 asmlinkage를 붙여서, 시스템 콜 호출은 int 80 인터럽트 핸들러에서 호출하는데 이 핸들러는 어셈블리 코드로 작성되기에 어셈블리 코드에서도 C함수 호출이 가능해지게 하였습니다.
`enqueue 함수`는 큐에 int형 변수를 받아서 큐에 추가하고,
`dequeue 함수`는 큐의 index가 가장 낮은 원소를 제거 하고 그 값을 `return` 하기에 
다음과 같이 함수의 input, output 타입을 작성하였습니다.

### 3. my_queue_syscall.c


`/usr/src/linux-4.20.11/kernel/my_queue_syscall.c` 파일에 새로 추가할 시스템 콜을 코딩 하였고, 코드에 대한 설명은 주석에 작성하였습니다.
`SYSCALL_DEFINEx`: 파라미터의 개수가 x개 인 시스템 콜 구현을 위한 매크로를 사용하기 위해 `#include<linux/syscalls.h>`을 추가하였습니다.

`[oslab_enqueue]`
1. rear 인덱스 변수와 MAXSIZE 변수를 비교하여, queue가 full인지 비교합니다.
2. input인 a가 queue에 이미 존재하는지 체크 합니다.
3. 중복되지 않았다면, queue에 원소를 삽입합니다.
4. 그 후 queue의 front ~ rear 까지 의 원소를 출력합니다.

`[oslab_dequeue]`
1. rear와 front 인덱스 변수를 비교하여 queue가 empty인지 확인합니다.
2. res에 queue에서 가장 처음에 들어온 값을 할당하고, 그 이후 res을 return 합니다.
3. queue의 front ~ rear 까지 의 원소를 출력. 이때 queue에서 i 번째 원소가 i – 1번째 원소로 한 칸씩 밀리는 과정을 거친후, front ~ rear까지 원소를 출력하게 됩니다.

** * `dequeue와 enqueue` 기능을 하는 함수를 짜는 방법은 이것 외에 여러가지 있으며, 요구조건을 만족하는 최소한의 기능을 수행하도록 저는 구현하였습니다.**

```c
#include<linux/syscalls.h>
#include<linux/kernel.h>
#include<linux/linkage.h>
#define MAXSIZE 500

int queue[MAXSIZE];
int front = 0; // queue의 맨 앞index을 나타내는 변수
int rear = 0; //  queue의 맨 뒤index를 나타내는 변수
int i, j, res = 0;


SYSCALL_DEFINE1(oslab_enqueue, int, a ){
	
	if (rear >= MAXSIZE - 1) { // if queue is full 
		printk(KERN_INFO "[Error] - QUEUE IS FULL------------------\n");
		return -2;
	}
	
	for ( i = 0; i<=rear ;i++) { // input a 가 queue에 이미 존재하는지 체크
		if (a == queue[i])
		{	
			printk(KERN_INFO "[Error] - Already existing value \n");
			return a;
		}
	}
	
	queue[rear] = a ; // 중복되지 않았다면, queue에 원소 삽입
	
	printk(KERN_INFO "[System call] oslab_enqueue(); ------\n");
	printk("Queue Front--------------------\n");
	for (i = front; i <= rear ; i ++)
		printk("%d\n", queue[i]); // queue의 front부터 시작해서 rear까지의 원소를 출력함.
	printk("Queue Rear--------------------\n");
	
	rear = rear + 1;

	return a;
}

SYSCALL_DEFINE0(oslab_dequeue){
	
	if (rear == front) { // if queue is empty
		printk(KERN_INFO "[Error] - EMPTY QUEUE------------------\n");
		return -2;
	}
	res = queue[front]; // 변수 res에 queue에서 가장 처음에 들어온 값을 할당, 그 후 return res
	rear = rear - 1;
	
	printk(KERN_INFO "[System call] oslab_dequeue(); ------\n");
	printk("Queue Front--------------------\n");
	
	for(i = front + 1; i < MAXSIZE ; i++)
		queue[i-1] = queue[i]; // queue에서 i번째 원소가 i-1번째 원소로 한 칸씩 밀림.

	
	for (j = front; j < rear ; j ++)
		printk("%d\n", queue[j]); // queue의 front부터 시작해서 rear까지의 원소를 출력함.
	
	printk("Queue Rear--------------------\n");

	return res;
}
```

### 4. Makefile
![](https://images.velog.io/images/ba159sal/post/1029389d-7876-43bc-b6e8-f0860ddc8fac/image.png)
`/usr/src/linux-4.20.11/kernel/Makefile` 파일의 `obj-y` 부분에 `my_queue_syscall.o`를 추가하였습니다. 커널을 Compile 할 때, 추가한 시스템 콜을 포함 시켜야 하기 때문에 추가하였고, 이로서 Compile 할 시 `my_queue_syscall.c` 파일도 컴파일 됩니다.


### 5. call_my_queue.c (User Application)
```c
#include<unistd.h>
#include<stdio.h>

#define my_queue_enqueue 335 // enqueue 에 대한 시스템 콜을 고유번호로 매핑하는 define
#define my_queue_dequeue 336 // dequeue 에 대한 시스템 콜을 고유번호로 매핑하는 define

int main(){
	int a = 0;
	
	a = syscall(my_queue_enqueue, 1); // 매크로 함수 syscall를 사용하여 시스템 콜을 호출함.
	printf("Enqueue : ");			//큐에 숫자 1을 넣음. 
	printf("%d\n", a);
	
	a = syscall(my_queue_enqueue, 2); //매크로 함수 syscall를 사용하여 시스템 콜을 호출함.
	printf("Enqueue : ");			//큐에 숫자 2을 넣음. 
	printf("%d\n", a);
	
	a = syscall(my_queue_enqueue, 3); //매크로 함수 syscall를 사용하여 시스템 콜을 호출함.
	printf("Enqueue : ");			//큐에 숫자 3을 넣음. 
	printf("%d\n", a);
	
	a = syscall(my_queue_enqueue, 2); // 매크로 함수 syscall를 사용하여 시스템 콜을 호출함.
	printf("Enqueue : ");			//큐에 숫자 2을 넣음. (기존 큐에 있던 2와 중복됨). 
	printf("%d\n", a);
	
	a = syscall(my_queue_dequeue); //매크로 함수 syscall를 사용하여 시스템 콜을 호출함.
	printf("Dequeue : ");		// 큐에서 index가 가장 낮은 원소를 빼내고 반환함. 
	printf("%d\n", a);
	
	a = syscall(my_queue_dequeue); //매크로 함수 syscall를 사용하여 시스템 콜을 호출함.
	printf("Dequeue : ");		// 큐에서 index가 가장 낮은 원소를 빼내고 반환함. 
	printf("%d\n", a);
	
	a = syscall(my_queue_dequeue); //매크로 함수 syscall를 사용하여 시스템 콜을 호출함.
	printf("Dequeue : ");		// 큐에서 index가 가장 낮은 원소를 빼내고 반환함. 
	printf("%d\n", a);
	
	return 0;
}
```

`unistd.h`의 syscall 함수를 통해 추가한 시스템 콜을 호출하고, 
`enqueue와 dequeue` 에 대한 시스템 콜의 고유번호를 `define`문을 통해 정의합니다.
`syscall(my_queue_enqueue, number)` : `sys_oslab_enqueue` 시스템 콜을 호출 합니다.
`syscall(my_queue_dequeue)` : `sys_oslab_dequeue` 시스템 콜을 호출 합니다.
시스템 콜의 반환 값을 a 라는 변수에 할당하여 출력합니다.
유저 application(`call_my_queue.c`)은 큐에 1,2,3,2을 순서대로 enqueue하고(큐 원소의 중복체크도 함.), 3번 dequeue를 합니다. 

**User Application 실행 결과**
![](https://images.velog.io/images/ba159sal/post/0233a602-2c50-408d-8179-862d8a374dd8/2021-05-19%2020;09;11.PNG)

## # `Kernel Log` 출력 결과
![](https://images.velog.io/images/ba159sal/post/69c4c3e9-da7e-41b4-8330-d8e1e284e9fd/%EC%BB%A4%EB%84%90%20%EB%A1%9C%EA%B7%B8%20%EC%B6%9C%EB%A0%A5%201.PNG)
![](https://images.velog.io/images/ba159sal/post/219aca60-5eb6-4026-b742-3041864acf86/%EC%BB%A4%EB%84%90%20%EB%A1%9C%EA%B7%B8%20%EC%B6%9C%EB%A0%A5%202.PNG)