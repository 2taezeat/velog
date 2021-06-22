## # Intro
수업 프로젝트 중 직접 5가지의 간단한`Process`를 만들고,
`Scheduling Algorithm Evaluation` (1. `FCFS`, 2. `Nonpreemptive SJF`, 3. `RR` ) 
3가지 알고리즘 각각에 대해 5가지 Process 들을 스케쥴링 한 결과와 
`Gantt chart` / `프로세스별 waiting time` / `평균 waiting time` 을 구하는 프로젝트를 수행하였습니다.


## # Process 설명

### 1. Simple Loop
```c
#include <stdio.h>
#include <time.h>

int main(void)
{	
	int i = 0;
	while (i < 10000000)
	{
		i = i + 1;
	}

	printf("P1 : Simple loop\n");
	printf("Process result : %d\n", i);
	

	return 0;
}
```
### 2. Hanoi Top
```c
#include <stdio.h>
#include <time.h>

void Hanoi(int n, const char* src, const char* use, const char* dest);

int main(void)
{
	printf("P2 : Hanoi Top\n");
	printf("Process result : \n");
    Hanoi(6, "A", "B", "C");


	return 0;
}

void Hanoi(int n, const char* src, const char* use, const char* dest)
{
    if (n <= 0)
    {
        return;
    }
    Hanoi(n - 1, src, dest, use);
    printf("%s → %s\n", src, dest);
    Hanoi(n - 1, use, src, dest);
}
```
### 3. Fibonacci
```c
#include <stdio.h>
#include <time.h>

int fibo(int num);

int main(void) {

	int num = 27;
	int i;
	printf("P3 : Fibonacci\n");
	printf("Process result :\n");

	for (i = 0; i < num; i++)
	{
		printf("%d ", fibo(i));
	}

	printf("\n");
    
	return 0;
}

int fibo(int num) {

	if (num == 0) return 0;
	else if (num == 1) return 1;
	else return fibo(num - 1) + fibo(num - 2);
}
```
### 4. Double Loop
```c
#include <stdio.h>
#include <time.h>
//double loop
int main(void) {

	printf("P4 : Double Loop\n");
	int sum1 = 0;
	for (int i = 0; i < 5000; i++)
	{
		for (int j = 0; j < 5000; j++)
		{
			sum1 = sum1 + i + j;
		}
	}
	printf("Process result : %d \n", sum1);

	return 0;
}
```
### 5. Factorial
```c
#include <stdio.h>
#include <time.h>
// factorial

int factorial(int n)
{
	if (n == 1)      // n이 1일 때
		return 1;    // 1을 반환하고 재귀호출을 끝냄

	return n * factorial(n - 1);    // n과 factorial 함수에 n - 1을 넣어서 반환된 값을 곱함
}

int main(void) {
	printf("P5 : Factorial\n");
	int result = factorial(30);
	printf("Process result : %d \n", result);
	return 0;
}
```


**전체 프로세스 실제 수행 결과**

|Process|Burst Time|Description|
|------|---|---|
|P1|34 ms|Simple Loop|
|P2|58 ms|Hanoi Top|
|P3|42 ms|Fibonacci|
|P4|74 ms|Double Loop|
|P5|13 ms|Factorial|

## # Process 실행
```c
#include <stdio.h>
#include <time.h>
#include <stdlib.h>
#include <Windows.h>

typedef struct process {
	int PID;
	int cpu_burst;
	int waiting_time;
}Process;

int main() {
	STARTUPINFO si; //new process의 특성
	PROCESS_INFORMATION pi; // new process/thread에 대한 핸들/식별자

	/*allocate memory*/
	ZeroMemory(&si, sizeof(si));
	si.cb = sizeof(si);
	ZeroMemory(&pi, sizeof(pi));
	/*create child process*/

	clock_t start1 = clock();
	CreateProcess(NULL, "C:\\Users\\admin\\Desktop\\process\\process1.exe", NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi); //process1 = Simple loop
	WaitForSingleObject(pi.hProcess, INFINITE);
	clock_t end1 = clock();
	double p1_burst_time = (end1 - start1);
	printf("P1_burst_time : %f\n", p1_burst_time);
	printf("------------------------------------------------\n");

	clock_t start2 = clock();
	CreateProcess(NULL, "C:\\Users\\admin\\Desktop\\process\\process2.exe", NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi); //process2 = Hanoi Top
	WaitForSingleObject(pi.hProcess, INFINITE);
	clock_t end2 = clock(); // 끝난 시간;
	double p2_burst_time = (end2 - start2);
	printf("P2_burst_time : %f\n", p2_burst_time);
	printf("------------------------------------------------\n");

	clock_t start3 = clock();
	CreateProcess(NULL, "C:\\Users\\admin\\Desktop\\process\\process3.exe", NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi); //process3 = Fibonacci
	WaitForSingleObject(pi.hProcess, INFINITE);
	clock_t end3 = clock(); // 끝난 시간;
	double p3_burst_time = (end3 - start3);
	printf("P3_burst_time : %f\n", p3_burst_time);
	printf("------------------------------------------------\n");

	clock_t start4 = clock();
	CreateProcess(NULL, "C:\\Users\\admin\\Desktop\\process\\process4.exe", NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi); //process4 = Double loop
	WaitForSingleObject(pi.hProcess, INFINITE);
	clock_t end4 = clock(); // 끝난 시간;
	double p4_burst_time = (end4 - start4);
	printf("P4_burst_time : %f\n", p4_burst_time);
	printf("------------------------------------------------\n");

	clock_t start5 = clock();
	CreateProcess(NULL, "C:\\Users\\admin\\Desktop\\process\\process5.exe", NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi); //process5 = Factorial
	WaitForSingleObject(pi.hProcess, INFINITE);
	clock_t end5 = clock(); // 끝난 시간;
	double p5_burst_time = (end5 - start5);
	printf("P5_burst_time : %f\n", p5_burst_time);
	printf("------------------------------------------------\n");

	CloseHandle(pi.hProcess); // new process에 대한 핸들 닫기
	CloseHandle(pi.hThread); // new threead에 대한 핸들 닫기

	Process ps[5];
	Process ready_queue[5];

	ps[0].PID = 1;
	ps[0].cpu_burst = p1_burst_time;
	ps[1].PID = 2;
	ps[1].cpu_burst = p2_burst_time;
	ps[2].PID = 3;
	ps[2].cpu_burst = p3_burst_time;
	ps[3].PID = 4;
	ps[3].cpu_burst = p4_burst_time;
	ps[4].PID = 5;
	ps[4].cpu_burst = p5_burst_time;


	return 0;
}
```


## # Scheduling Result
### 1. FCFS
![](https://images.velog.io/images/ba159sal/post/4b6d538f-f929-4e97-a330-384475766682/2021-05-19%2021;12;22.PNG)
### 2. Nonpreemptive SJF
![](https://images.velog.io/images/ba159sal/post/9a187210-c345-4668-979c-2e661006e554/2021-05-19%2021;12;36.PNG)
### 3. RR 
( * time slice = 50ms )
![](https://images.velog.io/images/ba159sal/post/4797d951-aa4f-4cb4-aa5d-52f4995a27a1/2021-05-19%2021;12;41.PNG)



> **Scheduling Result 요약본**
![](https://images.velog.io/images/ba159sal/post/52410933-7293-44a2-bada-25e468a67911/2021-05-19%2021;14;26.PNG)

