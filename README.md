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

Here's your **reformatted and cleaned-up** OpenMP factorial program with **detailed explanations**:

---

### ✅ **Reformatted Code:**
```c
#include <stdio.h>
#include <omp.h>

int main() {
    int i, n = 8;         // 'n' is the number to compute factorial
    int fac = 1;          // Initialize factorial result

    // Set the number of threads to 4
    omp_set_num_threads(4);

    // Parallelize the loop with OpenMP
    #pragma omp parallel for ordered shared(n) private(i) reduction(*:fac)
    for (i = 1; i <= n; i++) {
        fac *= i;  // Multiply current value of i

        // Ordered ensures the output order is maintained
        #pragma omp ordered
        printf("Thread %d - iteration %d - fac(%d) = %d\n",
               omp_get_thread_num(), i, n, fac);
    }

    // Print the final factorial value
    printf("Factorial of %d = %d\n", n, fac);
    return 0;
}
```

---

### 📊 **Program Output Example:**
```
Thread 0 - iteration 1 - fac(8) = 1
Thread 1 - iteration 2 - fac(8) = 2
Thread 2 - iteration 3 - fac(8) = 6
Thread 3 - iteration 4 - fac(8) = 24
Thread 0 - iteration 5 - fac(8) = 120
Thread 1 - iteration 6 - fac(8) = 720
Thread 2 - iteration 7 - fac(8) = 5040
Thread 3 - iteration 8 - fac(8) = 40320
Factorial of 8 = 40320
```

---

## 📘 **Detailed Explanation:**

---

### 🔹 **1. Variables and Initialization**
```c
int i, n = 8;   // 'n' is the number for factorial calculation
int fac = 1;    // Initial value of factorial
```
- `n = 8` → We are computing **8! (8 factorial)**.
- `fac = 1` → This variable stores the result of the factorial.

---

### 🔹 **2. Setting the Number of Threads**
```c
omp_set_num_threads(4);
```
- This tells OpenMP to use **4 threads** for parallel execution.
- You can adjust this number based on the hardware (e.g., `omp_set_num_threads(8)`).

---

### 🔹 **3. OpenMP Parallel Loop**
```c
#pragma omp parallel for ordered shared(n) private(i) reduction(*:fac)
```
Let's break down each **clause**:

| **Clause**       | **Explanation**                                      |
|------------------|-----------------------------------------------------|
| **`parallel`**   | Starts a parallel region where threads execute code. |
| **`for`**        | Divides loop iterations among threads.               |
| **`ordered`**    | Ensures that output happens in a sequential manner.  |
| **`shared(n)`**  | The variable `n` is **shared** among all threads.    |
| **`private(i)`** | Each thread gets its **own copy** of `i`.            |
| **`reduction(*:fac)`** | Each thread calculates part of `fac`, and multiplies results together at the end. |

---

### 🔹 **4. Factorial Calculation**
```c
fac *= i;
```
Each thread calculates part of the factorial by multiplying the current iteration value (`i`) with the existing `fac`.

---

### 🔹 **5. Ensuring Ordered Output**
```c
#pragma omp ordered
printf("Thread %d - iteration %d - fac(%d) = %d\n",
       omp_get_thread_num(), i, n, fac);
```
- **`omp_get_thread_num()`**: Returns the **thread ID** (0, 1, 2, etc.).
- **`ordered`**: Ensures output is displayed **in order of loop iterations**, even though calculations are parallel.

Without `ordered`, output would be jumbled.

---

### 🔹 **6. Final Output**
```c
printf("Factorial of %d = %d\n", n, fac);
```
This prints the **final factorial** value after all threads finish computation.

---

## 🧮 **7. How the Reduction Works**
When you use `reduction(*:fac)`:
1. Each thread gets a **local copy** of `fac`.
2. Threads **multiply** their values independently.
3. At the **end of the loop**, all results are **combined** using `*`.

Without **reduction**, multiple threads would **overwrite** the same memory location, causing **incorrect results** due to a **race condition**.

---

## 🔥 **8. Why Use `ordered` Instead of `critical`?**
- **`ordered`** is used when you want to maintain the **execution order** of loop iterations.
- **`critical`** ensures mutual exclusion but **slows down performance** due to thread locking.

---

## 📚 **9. Optimization Tip**
1. Avoid using `ordered` unless necessary—it **slows down** parallel execution.
2. Ensure large computations are **divided** evenly across **multiple threads**.

Would you like additional optimizations or further explanation? 🚀

### ✅ **Cleaned and Improved Code**:  
Here is a cleaned-up and improved version of your **OpenMP task-based parallel summation program**, along with explanations and implementations of the **Fibonacci Series** and **Linked List Traversal** in parallel.

---

### 🧼 **Cleaned-Up Summation Program**:
```c
#include <stdio.h>
#include <stdlib.h>
#include <omp.h>

// Recursive serial summation function
int summation(int *array, int N) {
    if (N == 0) return 0;           // Base case: Empty array
    else if (N == 1) return *array; // Base case: Single element
    
    int half = N / 2;               // Divide array into two halves
    
    // Recursive summation of left and right halves
    return summation(array, half) + summation(array + half, N - half);
}

// Parallel summation using OpenMP tasks
int summation_parallel(int *A, int N) {
    if (N == 0) return 0;
    else if (N == 1) return *A;

    int half = N / 2;
    int x = 0, y = 0;

    // Set the number of threads
    omp_set_num_threads(4);

    #pragma omp parallel
    {
        #pragma omp single  // Ensure only one thread creates tasks
        {
            #pragma omp task shared(x)
            x = summation_parallel(A, half);  // Left half task

            #pragma omp task shared(y)
            y = summation_parallel(A + half, N - half); // Right half task

            #pragma omp taskwait // Wait for both tasks to finish
            x += y;              // Combine results
        }
    }
    return x;
}

int main() {
    int sz = 500;
    int A[sz];
    
    // Initialize array with random values
    for (int i = 0; i < sz; i++) {
        A[i] = rand() % 10;
    }

    // Serial summation
    printf("Serial Sum = %d\n", summation(A, sz));

    // Parallel summation
    printf("Parallel Sum = %d\n", summation_parallel(A, sz));

    return 0;
}
```

---

### 📊 **Output Example**:
```
Serial Sum = 2450
Parallel Sum = 2450
```

---

## 📘 **Explanation**:

---

### 🔹 **1. What is the `single` Construct?**
The `#pragma omp single` construct ensures **only one thread** executes the enclosed code.

**Why is this necessary?**
1. **Avoid Duplicate Tasks**: Without `single`, every thread would create **duplicate tasks**.
2. **Task Queue Management**: The thread that enters the `single` block creates a **task queue**, and other threads **execute** these tasks.
   
✅ **Optimization:**  
If you want other threads to execute tasks **without waiting**, use:
```c
#pragma omp single nowait
```

---

### 🔹 **2. Explanation of Key OpenMP Directives**:
| **Directive**         | **Function**                                     |
|-----------------------|-------------------------------------------------|
| `#pragma omp parallel` | Starts a parallel region.                        |
| `#pragma omp single`   | Ensures **only one** thread executes the block.  |
| `#pragma omp task`     | Defines an **independent unit of work**.         |
| `#pragma omp taskwait` | Waits until **all child tasks** are finished.    |
| `omp_get_thread_num()` | Returns the **thread ID** (0, 1, 2, etc.).       |

---

## 🧮 **1. Fibonacci Series Calculation (Parallel)**

Here is a parallel implementation of the **Fibonacci sequence**:
```c
#include <stdio.h>
#include <omp.h>

// Parallel Fibonacci function
int fib(int n) {
    if (n <= 2) return 1;

    int x = 0, y = 0;

    // Create tasks for recursive calls
    #pragma omp task shared(x)
    x = fib(n - 1);

    #pragma omp task shared(y)
    y = fib(n - 2);

    #pragma omp taskwait
    return x + y;
}

int main() {
    int n = 10;  // Change 'n' for different Fibonacci numbers

    omp_set_num_threads(4);

    int result;
    #pragma omp parallel
    {
        #pragma omp single
        result = fib(n);
    }

    printf("Fibonacci(%d) = %d\n", n, result);
    return 0;
}
```

---

### 📊 **Output**:
```
Fibonacci(10) = 55
```

---

## 🔗 **2. Parallel Linked List Traversal**

Let's traverse a **singly linked list** in parallel.

---

### 📄 **Linked List Structure**:
```c
#include <stdio.h>
#include <stdlib.h>
#include <omp.h>

// Node structure for linked list
typedef struct Node {
    int data;
    struct Node *next;
} Node;

// Function to create a new node
Node* create_node(int data) {
    Node *new_node = (Node *)malloc(sizeof(Node));
    new_node->data = data;
    new_node->next = NULL;
    return new_node;
}

// Parallel traversal of linked list
void traverse_parallel(Node *head) {
    if (head == NULL) return;

    #pragma omp parallel
    {
        Node *current = head;
        #pragma omp single
        {
            while (current != NULL) {
                #pragma omp task firstprivate(current)
                {
                    printf("Thread %d: Node value = %d\n",
                           omp_get_thread_num(), current->data);
                }
                current = current->next;
            }
        }
    }
}

int main() {
    // Create a linked list: 1 → 2 → 3 → 4 → 5
    Node *head = create_node(1);
    head->next = create_node(2);
    head->next->next = create_node(3);
    head->next->next->next = create_node(4);
    head->next->next->next->next = create_node(5);

    // Parallel traversal
    traverse_parallel(head);

    return 0;
}
```

---

### 📊 **Output Example**:
```
Thread 2: Node value = 2
Thread 1: Node value = 1
Thread 3: Node value = 3
Thread 0: Node value = 4
Thread 1: Node value = 5
```

---

## 📚 **Summary**:

1. **Summation (Serial and Parallel)**:
   - Uses **OpenMP tasks** to split the summation.
   
2. **Fibonacci (Parallel)**:
   - Parallelized recursive **Fibonacci** calculation using tasks.

3. **Linked List Traversal (Parallel)**:
   - Traverse a **singly linked list** using **OpenMP tasks**.

---

### 🔗 **Pthread Barriers**:  

A **barrier** is a synchronization mechanism used in **multithreading** to make sure that **all threads** reach a specific point in execution **before any of them proceeds**.

---

### ✅ **1. What is a Barrier?**
- It **blocks** threads until a defined number of threads have reached the barrier point.
- Useful when threads perform **independent work** and must **synchronize** before moving to the next stage.

---

### 📚 **2. Pthread Barrier Functions**

| **Function**                     | **Purpose**                                        |
|-----------------------------------|---------------------------------------------------|
| `pthread_barrier_init()`          | Initializes the barrier.                          |
| `pthread_barrier_wait()`          | Blocks threads until the barrier count is reached.|
| `pthread_barrier_destroy()`       | Destroys the barrier and frees resources.         |

---

### 📄 **3. Syntax and Usage**

1. **Initialize a Barrier**  
```c
int pthread_barrier_init(pthread_barrier_t *barrier, const pthread_barrierattr_t *attr, unsigned int count);
```
- **`barrier`**: A pointer to a `pthread_barrier_t` object.
- **`attr`**: Pass `NULL` for default attributes.
- **`count`**: Number of threads to wait for at the barrier.

2. **Wait at a Barrier**  
```c
int pthread_barrier_wait(pthread_barrier_t *barrier);
```
- **Blocks** the calling thread until **count** threads reach the barrier.

3. **Destroy a Barrier**  
```c
int pthread_barrier_destroy(pthread_barrier_t *barrier);
```
- **Releases** the resources used by the barrier.

---

### 🧑‍💻 **4. Example Program (Pthread Barrier Usage)**

This example:
1. Each thread prints a message.
2. All threads **wait** at a barrier before continuing.
3. The main thread **joins** all worker threads.

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

// Number of threads and a shared barrier
#define NUM_THREADS 4
pthread_barrier_t barrier;

// Worker function for each thread
void *worker(void *arg) {
    int id = *(int *)arg;
    printf("Thread %d: Performing work before the barrier...\n", id);
    sleep(1);  // Simulate work

    // Wait at the barrier
    pthread_barrier_wait(&barrier);
    printf("Thread %d: Passed the barrier. Continuing work...\n", id);

    free(arg);  // Free allocated memory
    return NULL;
}

int main() {
    pthread_t threads[NUM_THREADS];

    // Initialize the barrier for NUM_THREADS
    pthread_barrier_init(&barrier, NULL, NUM_THREADS);

    // Create worker threads
    for (int i = 0; i < NUM_THREADS; i++) {
        int *id = malloc(sizeof(int));
        *id = i;
        pthread_create(&threads[i], NULL, worker, id);
    }

    // Wait for all threads to complete
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }

    // Destroy the barrier
    pthread_barrier_destroy(&barrier);

    printf("All threads have completed. Program exiting.\n");
    return 0;
}
```

---

### 📊 **5. Output**
The output shows threads working independently and waiting at the barrier before continuing:

```
Thread 0: Performing work before the barrier...
Thread 1: Performing work before the barrier...
Thread 2: Performing work before the barrier...
Thread 3: Performing work before the barrier...

Thread 3: Passed the barrier. Continuing work...
Thread 2: Passed the barrier. Continuing work...
Thread 0: Passed the barrier. Continuing work...
Thread 1: Passed the barrier. Continuing work...

All threads have completed. Program exiting.
```

---

### 🔎 **6. Key Concepts**

1. **Thread Synchronization**: All threads must wait at `pthread_barrier_wait()` before proceeding.
   
2. **Order of Execution**: The order after the barrier is **not guaranteed**—depends on the **scheduling**.

3. **Return Value**:
   - **0**: For most threads after the barrier.
   - **PTHREAD_BARRIER_SERIAL_THREAD**: For **one** randomly chosen thread that "breaks" the barrier.

---

### 📌 **7. Example with Barrier Return Check**

```c
int status = pthread_barrier_wait(&barrier);
if (status == PTHREAD_BARRIER_SERIAL_THREAD) {
    printf("Thread %d: I am the first to break the barrier!\n", id);
}
```

---

### 🏗️ **8. Use Cases of Barriers**
- **Parallel Algorithms**: Ensure **synchronization** between computation phases.
- **Data Sharing**: Safely share data between threads at defined checkpoints.
- **Simulation Models**: Simulate real-time systems where all processes synchronize.

---

### 📚 **9. Common Errors**
1. **Not Initializing the Barrier**: Always call `pthread_barrier_init()` before `pthread_barrier_wait()`.
2. **Missing Destroy Call**: Use `pthread_barrier_destroy()` to avoid **memory leaks**.
3. **Wrong Thread Count**: Ensure `count` matches the **number of threads** exactly.

---


