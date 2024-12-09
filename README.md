# Linux-IPC-Semaphores
Ex05-Linux IPC-Semaphores

# AIM:
To Write a C program that implements a producer-consumer system with two processes using Semaphores.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Sempahores

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that implements a producer-consumer system with two processes using Semaphores.

Simplified:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/sem.h>
#include <time.h>

#define NUM_LOOPS 10

union semun {
    int val;
    struct semid_ds *buf;
    unsigned short *array;
};

int main() {
    int sem_set_id;
    union semun sem_val;
    struct sembuf sem_op;
    pid_t child_pid;

    // Create semaphore
    sem_set_id = semget(IPC_PRIVATE, 1, 0600);
    if (sem_set_id == -1) {
        perror("semget");
        exit(1);
    }

    // Initialize semaphore to 0
    sem_val.val = 0;
    if (semctl(sem_set_id, 0, SETVAL, sem_val) == -1) {
        perror("semctl");
        exit(1);
    }

    // Fork to create producer and consumer
    child_pid = fork();
    if (child_pid == -1) {
        perror("fork");
        exit(1);
    }

    if (child_pid == 0) {  // Child process (Consumer)
        for (int i = 0; i < NUM_LOOPS; i++) {
            sem_op.sem_num = 0;
            sem_op.sem_op = -1;  // Wait on semaphore
            sem_op.sem_flg = 0;

            if (semop(sem_set_id, &sem_op, 1) == -1) {
                perror("semop");
                exit(1);
            }

            printf("Consumer: '%d'\n", i);
            fflush(stdout);
        }
        exit(0);
    } else {  // Parent process (Producer)
        for (int i = 0; i < NUM_LOOPS; i++) {
            printf("Producer: '%d'\n", i);
            fflush(stdout);

            sem_op.sem_num = 0;
            sem_op.sem_op = 1;  // Signal semaphore
            sem_op.sem_flg = 0;

            if (semop(sem_set_id, &sem_op, 1) == -1) {
                perror("semop");
                exit(1);
            }

            // Simulate some delay
            usleep(100000);
        }

        // Wait for child process to finish
        wait(NULL);

        // Remove semaphore
        if (semctl(sem_set_id, 0, IPC_RMID, sem_val) == -1) {
            perror("semctl (IPC_RMID)");
            exit(1);
        }
    }

    return 0;
}
```

Not so simplified:

```c
/*
 * sem-producer-consumer.c  - demonstrates a basic producer-consumer
 *                            implementation.
 */
#include <stdio.h>	 /* standard I/O routines.              */
#include <stdlib.h>      /* rand() and srand() functions        */
#include <unistd.h>	 /* fork(), etc.                        */
#include <time.h>	 /* nanosleep(), etc.                   */
#include <sys/types.h>   /* various type definitions.           */
#include <sys/ipc.h>     /* general SysV IPC structures         */
#include <sys/sem.h>	 /* semaphore functions and structs.    */
#define NUM_LOOPS	20	 /* number of loops to perform. */
#if defined(__GNU_LIBRARY__) && !defined(_SEM_SEMUN_UNDEFINED)
/* union semun is defined by including <sys/sem.h> */
#else
/* according to X/OPEN we have to define it ourselves */
union semun {
        int val;                    /* value for SETVAL */
        struct semid_ds *buf;       /* buffer for IPC_STAT, IPC_SET */
        unsigned short int *array;  /* array for GETALL, SETALL */
        struct seminfo *__buf;      /* buffer for IPC_INFO */
};
#endif
int main(int argc, char* argv[])
{
    int sem_set_id;	      /* ID of the semaphore set.       */
    union semun sem_val;      /* semaphore value, for semctl(). */
    int child_pid;	      /* PID of our child process.      */
    int i;		      /* counter for loop operation.    */
    struct sembuf sem_op;     /* structure for semaphore ops.   */
    int rc;		      /* return value of system calls.  */
    struct timespec delay;    /* used for wasting time.         */
	/* create a private semaphore set with one semaphore in it, */
    /* with access only to the owner.                           */
    sem_set_id = semget(IPC_PRIVATE, 1, 0600);
    if (sem_set_id == -1) {
	   perror("main: semget");
	   exit(1);
    }
    printf("semaphore set created, semaphore set id '%d'.\n", sem_set_id);
    /* intialize the first (and single) semaphore in our set to '0'. */
    sem_val.val = 0;
    rc = semctl(sem_set_id, 0, SETVAL, sem_val);
    /* fork-off a child process, and start a producer/consumer job. */
    child_pid = fork();
    switch (child_pid) {
	  case -1:	/* fork() failed */
	    perror("fork");
	    exit(1);
	 case 0:		/* child process here */
	    for (i=0; i<NUM_LOOPS; i++) {
		/* block on the semaphore, unless it's value is non-negative. */
		sem_op.sem_num = 0;
		sem_op.sem_op = -1;
		sem_op.sem_flg = 0;
		semop(sem_set_id, &sem_op, 1);
		printf("consumer: '%d'\n", i);
		fflush(stdout);
	    }
	    break;
		default:	/* parent process here */
			for (i=0; i<NUM_LOOPS; i++) {
			printf("producer: '%d'\n", i);
			fflush(stdout);
			/* increase the value of the semaphore by 1. */
			sem_op.sem_num = 0;
	sem_op.sem_op = 1;
			sem_op.sem_flg = 0;
			semop(sem_set_id, &sem_op, 1);
		/* pause execution for a little bit, to allow the */
		/* child process to run and handle some requests. */
		/* this is done about 25% of the time.            */
		if (rand() > 3*(RAND_MAX/4)) {
	    	    delay.tv_sec = 0;
	    	    delay.tv_nsec = 10;
	    	    //nanosleep(&delay, NULL);
		                      sleep(10); }	
if(NUM_LOOPS>=10)    {
	    semctl(sem_set_id, 0, IPC_RMID, sem_val) ; // Remove the sem_set_id
	    break;
		}
	}}
    return 0;
}
````



## OUTPUT
#### $ ./sem.o 
![image](https://github.com/user-attachments/assets/10b09d08-85d4-480c-8ebd-2bd7bb6d1c98)


#### $ ipcs
![image](https://github.com/user-attachments/assets/80aa2017-fb6f-4c3b-b247-53298cc16131)





# RESULT:
The program is executed successfully.
