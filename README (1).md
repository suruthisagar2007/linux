# Parallel Processing using MPI and OpenMP

# Commands

## MPI setup

```bash
sudo apt install python3
sudo apt install -y -qq mpich
sudo apt install python3-pip
pip install mpi4py
sudo apt-get update
sudo apt-get install libopenmpi-dev
sudo apt-get install openmpi-bin
```

## MPI run

```bash
gedit mpi_example.py
mpirun -np 2 python3 mpi_example.py   # fails: not enough slots
mpirun -np 1 python3 mpi_example.py
```

## OpenMP setup

```bash
sudo apt-get install libomp-dev
```

## OpenMP compile and run

```bash
gedit hello_omp.cpp
g++ -fopenmp hello_omp.cpp -o hello_omp.out
./hello_omp.out

gedit loop_parallel.cpp
g++ -fopenmp loop_parallel.cpp -o loop_parallel.out
./loop_parallel.out

gedit private.cpp
g++ -fopenmp private.cpp -o private.out
./private.out

gedit first_private.cpp
g++ -fopenmp first_private.cpp -o first_private.out
./first_private.out

gedit last_private.cpp
g++ -fopenmp last_private.cpp -o last_private.out
./last_private.out

gedit critical.cpp
g++ -fopenmp critical.cpp -o critical.out
./critical.out
```

---

# Code

## mpi_example.py

```python
from mpi4py import MPI

comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

print(f"Hello from process {rank} out of {size}")
```

## hello_omp.cpp

```cpp
#include <omp.h>
#include <iostream>
int main(){
    omp_set_num_threads(4);
    #pragma omp parallel
    {
        int thread_id = omp_get_thread_num();
        int total_threads = omp_get_num_threads();
        std::cout << "Hello world from thread " << thread_id
                  << " out of " << total_threads << " thread.\n";
    }
    return 0;
}
```

## loop_parallel.cpp

```cpp
#include <omp.h>
#include <iostream>
#include <chrono>
#define THREAD_COUNT 5
using namespace std;

long long serial_sum(int n){
    long long sum = 0;
    for(int i = 1; i <= n; i++){
        sum += i;
    }
    return sum;
}

long long parallel_sum(int n){
    long long sum = 0;
    #pragma omp parallel for reduction(+:sum)
    for(int i = 1; i <= n; i++){
        sum += i;
    }
    return sum;
}

int main(int argc, char** argv){
    omp_set_num_threads(THREAD_COUNT);
    int n = 100000000;

    auto start_time1 = chrono::high_resolution_clock::now();
    long long ser_sum = serial_sum(n);
    auto end_time1 = chrono::high_resolution_clock::now();
    chrono::duration<double> duration1 = end_time1 - start_time1;

    auto start_time2 = chrono::high_resolution_clock::now();
    long long par_sum = parallel_sum(n);
    auto end_time2 = chrono::high_resolution_clock::now();
    chrono::duration<double> duration2 = end_time2 - start_time2;

    cout << "Serial Result: " << ser_sum << endl;
    cout << "Time taken for serial execution: " << duration1.count() << endl;
    cout << "Parallel Result: " << par_sum << endl;
    cout << "Time Taken for Parallel Execution: " << duration2.count() << endl;
    return 0;
}
```

## private.cpp

```cpp
#include <omp.h>
#include <iostream>
using namespace std;
int main(){
    int x = 10;
    #pragma omp parallel private(x)
    {
        cout << "Thread " << omp_get_thread_num() << " at starting has x = " << x << endl;
        x = omp_get_thread_num();
        cout << "Thread " << omp_get_thread_num() << " after initialization has x = " << x << endl;
    }
    cout << "outside parallel region, x = " << x << endl;
    return 0;
}
```

## first_private.cpp

```cpp
#include <omp.h>
#include <iostream>
using namespace std;
int main(){
    int x = 10;
    #pragma omp parallel firstprivate(x)
    {
        cout << "Thread " << omp_get_thread_num() << " start with x = " << x << endl;
        x += omp_get_thread_num();
        cout << "Thread " << omp_get_thread_num() << " end with x = " << x << endl;
    }
    cout << "outside parallel region, x = " << x << endl;
    return 0;
}
```

## last_private.cpp

```cpp
#include <omp.h>
#include <iostream>
using namespace std;
int main(){
    int x = 10;
    #pragma omp parallel for lastprivate(x)
    for(int i = 0; i < 4; i++)
    {
        x = i;
        cout << "Thread " << omp_get_thread_num() << " working on iteration "
             << i << " set x = " << x << endl;
    }
    cout << "outside parallel region, x = " << x << endl;
    return 0;
}
```

## critical.cpp

```cpp
#include <omp.h>
#include <iostream>
using namespace std;
int main(){
    int total_sum = 0;
    #pragma omp parallel for
    for(int i = 0; i < 1000; i++)
    {
        int local_sum = i;
        #pragma omp critical
        {
            total_sum += local_sum;
        }
    }
    cout << "Total sum : " << total_sum << endl;
    return 0;
}
```
