### **1. Include Necessary Headers**
```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
```

---

### **2. Creating and Joining Threads**
```c
void *print_message(void *arg) {
    printf("Hello from thread!\n");
    return NULL;
}

int main() {
    pthread_t thread;
    
    // Create thread
    pthread_create(&thread, NULL, print_message, NULL);
    
    // Wait for thread to finish
    pthread_join(thread, NULL);
    
    return 0;
}
```

- **`pthread_create(&thread, NULL, function, arg)`** → Creates a new thread.
- **`pthread_join(thread, NULL)`** → Waits for the thread to finish.

---

### **3. Using Mutex Locks (Prevent Race Conditions)**
```c
pthread_mutex_t lock;  // Declare mutex

void *critical_section(void *arg) {
    pthread_mutex_lock(&lock);  // Lock before critical section
    printf("Thread %d in critical section\n", *(int *)arg);
    pthread_mutex_unlock(&lock);  // Unlock after critical section
    return NULL;
}

int main() {
    pthread_t threads[2];
    int thread_id[2] = {0, 1};

    pthread_mutex_init(&lock, NULL);  // Initialize mutex

    for (int i = 0; i < 2; i++)
        pthread_create(&threads[i], NULL, critical_section, &thread_id[i]);

    for (int i = 0; i < 2; i++)
        pthread_join(threads[i], NULL);

    pthread_mutex_destroy(&lock);  // Destroy mutex
    return 0;
}
```

- **`pthread_mutex_init(&lock, NULL)`** → Initialize mutex.
- **`pthread_mutex_lock(&lock)`** → Acquire lock.
- **`pthread_mutex_unlock(&lock)`** → Release lock.
- **`pthread_mutex_destroy(&lock)`** → Clean up.

---

### **4. Using Condition Variables (Producer-Consumer)**
```c
pthread_mutex_t lock;
pthread_cond_t cond;
int ready = 0;

void *producer(void *arg) {
    pthread_mutex_lock(&lock);
    ready = 1;
    pthread_cond_signal(&cond);  // Signal consumer
    pthread_mutex_unlock(&lock);
    return NULL;
}

void *consumer(void *arg) {
    pthread_mutex_lock(&lock);
    while (!ready)
        pthread_cond_wait(&cond, &lock);  // Wait for producer
    printf("Consumed data\n");
    pthread_mutex_unlock(&lock);
    return NULL;
}

int main() {
    pthread_t t1, t2;

    pthread_mutex_init(&lock, NULL);
    pthread_cond_init(&cond, NULL);

    pthread_create(&t1, NULL, producer, NULL);
    pthread_create(&t2, NULL, consumer, NULL);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    pthread_mutex_destroy(&lock);
    pthread_cond_destroy(&cond);
    return 0;
}
```

- **`pthread_cond_wait(&cond, &lock)`** → Consumer waits for signal.
- **`pthread_cond_signal(&cond)`** → Producer signals consumer.

---

### **5. Parallel Matrix Multiplication**
```c
#define ROWS 3
#define COLS 3
#define NUM_THREADS 3

int A[ROWS][COLS] = {{1, 2, 3}, {4, 5, 6}, {7, 8, 9}};
int B[COLS][ROWS] = {{9, 8, 7}, {6, 5, 4}, {3, 2, 1}};
int C[ROWS][ROWS];

void *multiply(void *arg) {
    int row = *(int *)arg;
    for (int j = 0; j < ROWS; j++) {
        C[row][j] = 0;
        for (int k = 0; k < COLS; k++)
            C[row][j] += A[row][k] * B[k][j];
    }
    return NULL;
}

int main() {
    pthread_t threads[NUM_THREADS];
    int indices[NUM_THREADS];

    for (int i = 0; i < NUM_THREADS; i++) {
        indices[i] = i;
        pthread_create(&threads[i], NULL, multiply, &indices[i]);
    }

    for (int i = 0; i < NUM_THREADS; i++)
        pthread_join(threads[i], NULL);

    return 0;
}
```

---

### **6. Compilation & Execution**
```bash
gcc program.c -o program -lpthread
./program
```
