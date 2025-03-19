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



## 🔍 **1. What is OpenMP?**
OpenMP (**Open Multi-Processing**) is an API for writing **parallel programs** in C, C++, and Fortran. It supports **shared-memory parallelism** using compiler directives, runtime routines, and environment variables.

---

## 📌 **2. Setting Up OpenMP**
1. **Compiling an OpenMP Program**
```bash
gcc -fopenmp program.c -o output
```

2. **Controlling the Number of Threads**
```bash
export OMP_NUM_THREADS=4  # Use 4 threads
./output
```

---

## 📘 **3. Basic OpenMP Program Structure**
```c
#include <stdio.h>
#include <omp.h>

int main() {
    #pragma omp parallel
    {
        int id = omp_get_thread_num();
        printf("Hello from thread %d\n", id);
    }
    return 0;
}
```

✔️ This will output "Hello from thread X" for each thread.

---

## 📚 **4. OpenMP Directives and Examples**

---

### 📌 **4.1 `#pragma omp parallel`**
Used to define a **parallel region** where multiple threads execute concurrently.

**Example:**
```c
#include <stdio.h>
#include <omp.h>

int main() {
    #pragma omp parallel
    {
        printf("Thread %d is working\n", omp_get_thread_num());
    }
    return 0;
}
```
**Output (order may vary):**
```
Thread 0 is working
Thread 1 is working
Thread 2 is working
Thread 3 is working
```

---

### 📌 **4.2 `#pragma omp for`**
Divides **loop iterations** among multiple threads.

**Example:**
```c
#include <stdio.h>
#include <omp.h>

int main() {
    int sum = 0;
    #pragma omp parallel for
    for (int i = 0; i < 10; i++) {
        printf("Thread %d: i = %d\n", omp_get_thread_num(), i);
    }
    return 0;
}
```

✅ **Note:** Ensure the loop variable is **private** by default.

---

### 📌 **4.3 `#pragma omp sections`**
Executes **different code blocks** in parallel.

**Example:**
```c
#include <stdio.h>
#include <omp.h>

int main() {
    #pragma omp parallel sections
    {
        #pragma omp section
        printf("Task 1 by thread %d\n", omp_get_thread_num());

        #pragma omp section
        printf("Task 2 by thread %d\n", omp_get_thread_num());
    }
    return 0;
}
```

---

### 📌 **4.4 `#pragma omp single`**
Ensures **only one** thread executes a block.

**Example:**
```c
#include <stdio.h>
#include <omp.h>

int main() {
    #pragma omp parallel
    {
        #pragma omp single
        printf("Only one thread prints this.\n");

        printf("All threads execute this part.\n");
    }
    return 0;
}
```

---

### 📌 **4.5 `#pragma omp master`**
Only the **master** thread (thread 0) executes the block.

**Example:**
```c
#include <stdio.h>
#include <omp.h>

int main() {
    #pragma omp parallel
    {
        #pragma omp master
        printf("This is the master thread: %d\n", omp_get_thread_num());
    }
    return 0;
}
```

---

### 📌 **4.6 `#pragma omp critical`**
Allows **only one thread** at a time to execute a block (prevents race conditions).

**Example:**
```c
#include <stdio.h>
#include <omp.h>

int main() {
    int sum = 0;

    #pragma omp parallel
    {
        #pragma omp critical
        sum += 1;
    }
    printf("Sum: %d\n", sum);
    return 0;
}
```

---

### 📌 **4.7 `#pragma omp atomic`**
Performs a **single atomic operation** (faster than `#pragma omp critical`).

**Example:**
```c
#include <stdio.h>
#include <omp.h>

int main() {
    int sum = 0;
    #pragma omp parallel
    {
        #pragma omp atomic
        sum++;
    }
    printf("Sum: %d\n", sum);
    return 0;
}
```

---

### 📌 **4.8 `#pragma omp barrier`**
Forces all threads to **wait** at a point before continuing.

**Example:**
```c
#include <stdio.h>
#include <omp.h>

int main() {
    #pragma omp parallel
    {
        printf("Before barrier: %d\n", omp_get_thread_num());

        #pragma omp barrier

        printf("After barrier: %d\n", omp_get_thread_num());
    }
    return 0;
}
```

---

### 📌 **4.9 `#pragma omp task`**
Creates **asynchronous tasks**.

**Example:**
```c
#include <stdio.h>
#include <omp.h>

int main() {
    #pragma omp parallel
    {
        #pragma omp single
        {
            for (int i = 0; i < 5; i++) {
                #pragma omp task
                printf("Task %d executed by thread %d\n", i, omp_get_thread_num());
            }
        }
    }
    return 0;
}
```

---

### 📌 **4.10 `#pragma omp reduction`**
Combines values using **reduction operators** across threads.

**Example:**
```c
#include <stdio.h>
#include <omp.h>

int main() {
    int sum = 0;
    #pragma omp parallel for reduction(+:sum)
    for (int i = 1; i <= 10; i++) {
        sum += i;
    }
    printf("Sum: %d\n", sum);  // Outputs 55
    return 0;
}
```

✅ **Common Reduction Operators:**
- `+:` Sum
- `*: ` Product
- `&:` Bitwise AND
- `|:` Bitwise OR
- `^:` Bitwise XOR

---

## 📊 **5. OpenMP Environment Variables**
| Variable               | Description                       |
|------------------------|-----------------------------------|
| `OMP_NUM_THREADS`      | Number of threads to use          |
| `OMP_DYNAMIC`          | Enable/disable dynamic threads    |
| `OMP_SCHEDULE`         | Control scheduling policy         |
| `OMP_STACKSIZE`        | Set thread stack size             |

Example:
```bash
export OMP_NUM_THREADS=8
```

---

## 📚 **6. Common OpenMP Functions**
| Function                   | Description                           |
|----------------------------|---------------------------------------|
| `omp_get_thread_num()`     | Returns the thread's ID (0 to N-1)    |
| `omp_get_num_threads()`    | Returns the number of threads         |
| `omp_set_num_threads(n)`   | Sets the number of threads            |
| `omp_get_wtime()`          | Returns the wall-clock time           |
| `omp_in_parallel()`        | Returns `1` if in parallel, `0` otherwise |

Example:
```c
printf("Time: %f seconds\n", omp_get_wtime());
```

---

## 📈 **7. Performance Tips**
1. Use **`reduction`** for thread-safe accumulations.
2. Avoid overusing **`critical`** and **`atomic`** (they slow down performance).
3. Tune **scheduling**:
   - `static` – Equal block distribution.
   - `dynamic` – Assigns chunks as threads finish.
4. Minimize thread communication.

---
