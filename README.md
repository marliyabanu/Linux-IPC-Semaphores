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
```
// producer_consumer.c

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/sem.h>
#include <sys/types.h>
#include <sys/wait.h>

#define BUFFER_SIZE 5

struct shared_data {
    int buffer[BUFFER_SIZE];
    int in;
    int out;
};

union semun {
    int val;
};

int main() {

    key_t key = ftok("semfile", 65);

    // Shared memory
    int shmid = shmget(key, sizeof(struct shared_data), 0666 | IPC_CREAT);
    struct shared_data *data = (struct shared_data *)shmat(shmid, NULL, 0);

    data->in = 0;
    data->out = 0;

    // Semaphores
    int mutex = semget(key + 1, 1, 0666 | IPC_CREAT);
    int empty = semget(key + 2, 1, 0666 | IPC_CREAT);
    int full  = semget(key + 3, 1, 0666 | IPC_CREAT);

    union semun arg;

    arg.val = 1;
    semctl(mutex, 0, SETVAL, arg);

    arg.val = BUFFER_SIZE;
    semctl(empty, 0, SETVAL, arg);

    arg.val = 0;
    semctl(full, 0, SETVAL, arg);

    struct sembuf wait_op = {0, -1, 0};
    struct sembuf signal_op = {0, 1, 0};

    pid_t pid = fork();

    if (pid == 0) {

        // Consumer
        for (int i = 0; i < 5; i++) {

            semop(full, &wait_op, 1);
            semop(mutex, &wait_op, 1);

            int item = data->buffer[data->out];
            printf("Consumer consumed: %d\n", item);

            data->out = (data->out + 1) % BUFFER_SIZE;

            semop(mutex, &signal_op, 1);
            semop(empty, &signal_op, 1);

            sleep(1);
        }

    } else {

        // Producer
        for (int i = 1; i <= 5; i++) {

            semop(empty, &wait_op, 1);
            semop(mutex, &wait_op, 1);

            data->buffer[data->in] = i;
            printf("Producer produced: %d\n", i);

            data->in = (data->in + 1) % BUFFER_SIZE;

            semop(mutex, &signal_op, 1);
            semop(full, &signal_op, 1);

            sleep(1);
        }

        wait(NULL);

        // Cleanup
        shmdt(data);
        shmctl(shmid, IPC_RMID, NULL);

        semctl(mutex, 0, IPC_RMID);
        semctl(empty, 0, IPC_RMID);
        semctl(full, 0, IPC_RMID);
    }

    return 0;
}
```




## OUTPUT

<img width="365" height="360" alt="image" src="https://github.com/user-attachments/assets/81422865-c094-485f-beb2-f8ee6dd7405b" />

<img width="705" height="489" alt="image" src="https://github.com/user-attachments/assets/c12706ad-c311-4e2b-96ed-98337d670944" />







# RESULT:
The program is executed successfully.
