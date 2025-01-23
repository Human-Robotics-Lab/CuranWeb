---
layout: "default"
permalink : "/utilities/"
---

# Utilities

The utilities library is contained in the Curan API is located in the library folders in the utils folder. In CMAKE the target of the library is 'utils' and tu use it you can define a CMakeLists.txt with the following content 

```cmake
add_exectutable(myexample main.cpp)

target_link_libraries(myexample PUBLIC
utils
)
```

This code signals to CMake that our target depends on utils and when we compile it we must have the relative paths to both the include directories and the library files of the utils target. Now we will introduce a bit of the library for you to get a better graps of when and where to use it.

## Index

* ThreadPool and Jobs : [ThreadPool and Jobs](#threadpool-and-jobs)
* SafeQueue : [SafeQueue](#safequeue) 
* MemoryUtils : [MemoryUtils](#memoryutils)
* DateManipulation : [DateManipulation](#datemanipulation)
* FileStructures : [FileStructures](#filestructures)
* Logger : [Logger](#logger)
* Reader : [Reader](#reader)
* StringManipulation : [StringManipulation](#stringmanipulation)

# Tutorials

## ThreadPool and Jobs

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp
#include <iostream>
#include "utils/TheadPool.h"

void thread_pool_tutorial(){
    using namespace curan::utilities;
    std::atomic<size_t> value = 0;
    Job job{"increment value",[&](){++value;}};
    std::cout << "the description should be the same: (expected \"increment value\")" << job.description() << std::endl ;
    std::cout << "the value should be 0: "<< value << std::endl ;
    job();
    std::cout << "the value should be 1: "<< value << std::endl ;
    job();
    std::cout << "the value should be 2: "<< value << std::endl ;
    job();
    std::cout << "the value should be 3: "<< value << std::endl ;
    job();
    std::cout << "the value should be 4: "<< value << std::endl ;
    {
        value = 0;
        auto pool = ThreadPool::create(1,TERMINATE_ALL_PENDING_TASKS);
        for(size_t i = 0; i < 100; ++i)
            pool->submit(job);
    } // ~pool() is called here
    std::cout << "the value should be 99: "<< value << std::endl ;
    {
        value = 0;
        auto pool = ThreadPool::create(1,RETURN_AS_FAST_AS_POSSIBLE);
        for(size_t i = 0; i < 100; ++i)
            pool->submit("increment value",[&](){std::this_thread::sleep_for(std::chrono::microseconds(1));++value;});
        std::this_thread::sleep_for(std::chrono::microseconds(50));
    } // ~pool() is called here
    std::cout << "the value should be smaller than 99: "<< value << std::endl ;
}
```

We start by providing the necessary include directories 

```cpp
#include <iostream>
#include "utils/TheadPool.h"
```

we also employ the using directive to reduce the typing required 

```cpp
using namespace curan::utilities;
```

the manipulated variable with the Jobs class will be an atomic variable which we will increment. It should be atomic so as to guarantee that 
data is always in a consistent state, e.g., we avoid race conditions. 

```cpp
std::atomic<size_t> value = 0;
```

because jobs are running on a thread pool (more on that later) it is useful to attach string descriptors to each job.
We can hence query for the description of the task for a particular job. 

```cpp
Job job{"increment value",[&](){++value;}};
std::cout << "the description should be the same: (expected \"increment value\")" << job.description() << std::endl ;
```

note that the job takes a lambda that increments, by reference, the values that begins at zero. This means that we can 
increment it everytime the operator() is called.

```cpp
std::cout << "the value should be 0: "<< value << std::endl ;
job();
std::cout << "the value should be 1: "<< value << std::endl ;
job();
std::cout << "the value should be 2: "<< value << std::endl ;
job();
std::cout << "the value should be 3: "<< value << std::endl ;
job();
std::cout << "the value should be 4: "<< value << std::endl ;
```

now we finaly focus our attention on the most important concept, ThreadPools. A thread pool is 
an aglomeration of threads that can execute jobs anytime, anywhere. As a design choise we allow
the developer to specify how many threads a particular ThreadPool allocated. Once the ThreadPool
destructor is called there are two possible customizable behaviors. Assume that your ThreadPool
has a single thread. If 100 jobs are submited, as shown next

```cpp
{
    value = 0;
    auto pool = ThreadPool::create(1,TERMINATE_ALL_PENDING_TASKS);
    for(size_t i = 0; i < 100; ++i)
        pool->submit(job);
} // ~pool() is called here
std::cout << "the value should be 99: "<< value << std::endl ;
```

the destructor of the pool will be called before the ThreadPool executes all submited jobs. Thus we are faced with 
a choice, we either block on the destructor untill all jobs have been executed, or we return as soon as we can. In the 
previous code we requested from the constructor to terminate all pending tasks, thus the line "// ~pool() is called here"
will be blocking until value is 99. On the other hand, we can request to return as fast as possible through the following code listing

```cpp
{
    value = 0;
    auto pool = ThreadPool::create(1,RETURN_AS_FAST_AS_POSSIBLE);
    for(size_t i = 0; i < 100; ++i)
        pool->submit("increment value",[&](){std::this_thread::sleep_for(std::chrono::microseconds(1));++value;});
    std::this_thread::sleep_for(std::chrono::microseconds(50));
} // ~pool() is called here
std::cout << "the value should be smaller than 99: "<< value << std::endl ;
```

note that in this case line "// ~pool() is called here" is blocking until the current jobs, in this case one because there is a single 
thread, terminate their task, and we ignore all pending work. 

## SafeQueue

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp
#include <iostream>
#include "utils/SafeQueue.h"

void safe_queue_tutorial(){
    using namespace curan::utilities;
    SafeQueue<std::string> queue;
    auto pool = ThreadPool::create(1,TERMINATE_ALL_PENDING_TASKS);
    pool->submit("string submission",[&](){std::this_thread::sleep_for(std::chrono::milliseconds(50)); queue.push("osifhm9xq904rsdfsdcvw4tererge55gdfgx0");});
    auto value = queue.wait_and_pop(std::chrono::milliseconds(100));
    std::cout << ( value ? "the string (expected \"osifhm9xq904rsdfsdcvw4tererge55gdfgx0\")in the queue is:" + *value :  "no string was received (unexpected)" ) << std::endl;
    pool->submit("string submission",[&](){std::this_thread::sleep_for(std::chrono::milliseconds(500)); queue.push("osifhm9xq904rsdfsdcvw4tererge55gdfgx0");});
    value = queue.wait_and_pop(std::chrono::milliseconds(100));
    std::cout << ( value ? "this case should never happen" + *value :  "expected no value due to timing" ) << std::endl;
    value = queue.try_pop();
    std::cout << ( value ? "this case should never happen" + *value :  "expected no value due to timing" ) << std::endl;
    queue.emplace("osifhm9xq904rsdfsdcvw4tererge55gdfgx0");
    value = queue.try_pop();
    std::cout << ( value ? "the string (expected \"osifhm9xq904rsdfsdcvw4tererge55gdfgx0\")in the queue is:" + *value :  "no string was received (unexpected)" ) << std::endl;
}   
```

We start by providing the necessary include directories 

```cpp
#include <iostream>
#include "utils/SafeQueue.h"
```

we also employ the using directive to reduce the typing required 

```cpp
using namespace curan::utilities;
```

now we create the safe queue and a threadpool. We then commit a function that first waits 50 milliseconds and then pushes a string unto the queue.

```cpp
SafeQueue<std::string> queue;
auto pool = ThreadPool::create(1,TERMINATE_ALL_PENDING_TASKS);
pool->submit("string submission",[&](){std::this_thread::sleep_for(std::chrono::milliseconds(50)); queue.push("osifhm9xq904rsdfsdcvw4tererge55gdfgx0");});
```

we then submit a blocking request where we wait at a maximum 100 milliseconds for someone to submit a string to our queue. Notice that the function will only block 
for 50 milliseconds and not 100 milliseconds because someone submits a string 50 ms later. 

```cpp
auto value = queue.wait_and_pop(std::chrono::milliseconds(100));
std::cout << ( value ? "the string (expected \"osifhm9xq904rsdfsdcvw4tererge55gdfgx0\")in the queue is:" + *value :  "no string was received (unexpected)" ) << std::endl;
```

on the other hand, if the submission takes longer than 100 ms, in the following case 500 ms, then the wait_and_pop function returns a null optional. 

```cpp
pool->submit("string submission",[&](){std::this_thread::sleep_for(std::chrono::milliseconds(500)); queue.push("osifhm9xq904rsdfsdcvw4tererge55gdfgx0");});
value = queue.wait_and_pop(std::chrono::milliseconds(100));
std::cout << ( value ? "this case should never happen" + *value :  "expected no value due to timing" ) << std::endl;
```

on certain applications, we might not wish to wait for any time, either because waiting adds latency or due to some other requirement. In this case we can
try to get an instantaneous snapshot that either returns a string or a empty optional. Because we always remove a string when we read from the queue the 
previous function calls, the queue is currently empty at this point in the code, thus the code will return an empty optional

```cpp
value = queue.try_pop();
std::cout << ( value ? "this case should never happen" + *value :  "expected no value due to timing" ) << std::endl;
```

on the following snippet of code we demonstrate that try pop also returns a string 

```cpp
queue.emplace("osifhm9xq904rsdfsdcvw4tererge55gdfgx0");
value = queue.try_pop();
std::cout << ( value ? "the string (expected \"osifhm9xq904rsdfsdcvw4tererge55gdfgx0\")in the queue is:" + *value :  "no string was received (unexpected)" ) << std::endl;
```

## MemoryUtils

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp
#include <iostream>
#include "utils/MemoryUtils.h"

bool comparator(const std::string &value_to_control, const std::shared_ptr<curan::utilities::MemoryBuffer> &buffer){
  size_t address = 0;
  for (auto begin = buffer->begin(); begin != buffer->end(); ++begin)
    for (size_t j = 0; j < begin->size(); ++j, ++address)
      if (value_to_control.at(address) != *(((const char *)begin->data()) + j))
        return false;
  return true;
};

void memory_utils_tutorial(){
using namespace curan::utilities;
{
  std::string mem = "osifhm9xq904rsdfsdcvw4tererge55gdfgx0";
  // the copy buffer receives an pointer and the size of data to be copied and does so promply
  auto buffer = CopyBuffer::make_shared(mem.data(), mem.size());
  std::cout << "the buffers should be the same: " << comparator(mem,buffer) << std::endl;
  mem[0] = 'a';
  std::cout << "the buffers should not be the same: " << comparator(mem,buffer) << std::endl;
}

{
  auto mem = std::make_shared<std::string>("osifhm9xq904rsdfsdcvw4tererge55gdfgx0");
  // the copy buffer receives an pointer and the size of data to be copied and does so promply
  auto buffer = CaptureBuffer::make_shared(mem->data(), mem->size(),mem);
  std::cout << "the buffers should be the same: " << comparator(*mem,buffer) << std::endl;
  mem->data()[0] = 'a';
  std::cout << "the buffers should still be the same: " << comparator(*mem,buffer) << std::endl;
}
}
```

We start by providing the necessary include directories 

```cpp
#include <iostream>
#include "utils/MemoryUtils.h"
```

first we define a function that takes a string and a curan memory buffer and returns the comparison result byte by byte. This is 
useful for demonstrative purpouses. 

```cpp
bool comparator(const std::string &value_to_control, const std::shared_ptr<curan::utilities::MemoryBuffer> &buffer){
  size_t address = 0;
  for (auto begin = buffer->begin(); begin != buffer->end(); ++begin)
    for (size_t j = 0; j < begin->size(); ++j, ++address)
      if (value_to_control.at(address) != *(((const char *)begin->data()) + j))
        return false;
  return true;
};
```

we also employ the using directive to reduce the typing required 

```cpp
using namespace curan::utilities;
```

now we first create a blob of memory [0 37] bytes

```cpp
std::string mem = "osifhm9xq904rsdfsdcvw4tererge55gdfgx0";
```

and we wish to pass this blob of memory from thread 1 to thread 2. 
Note that if after line "std::cout << "the buffers should not be the same: " << comparator(mem,buffer) << std::endl;"
the mem string is destroyed, its usefull to decouple the lifetime of the blob of memory from our buffer. For this purpouse 
we use a CopyBuffer. This class takes a const char* pointer and allocates internally a new buffer [38 75] bytes in lenght
and copy the data from mem into buff. 

```cpp
// the copy buffer receives an pointer and the size of data to be copied and does so promply
auto buffer = CopyBuffer::make_shared(mem.data(), mem.size());
```

note that if we compare the memory contents of both buffers they are equivalent, because they were just copied from one to the other

```cpp
std::cout << "the buffers should be the same: " << comparator(mem,buffer) << std::endl;
```

now because these buffers are independent, if we change mem, then the comparison will no longer hold true

```cpp
mem[0] = 'a';
std::cout << "the buffers should not be the same: " << comparator(mem,buffer) << std::endl;
```

## DateManipulation

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp
#include <iostream>
#include "utils/DateManipulation.h"

void date_manipulation_tutorial(){
    using namespace curan::utilities;
    auto date = formated_date<std::chrono::system_clock>(std::chrono::system_clock::time_point(std::chrono::system_clock::duration(0)));
    std::cout << "the computed date with the provided time point is (expected \"1970-01-01 00:00:00\"):" << date  << std::endl;
}
```

We start by providing the necessary include directories 

```cpp
#include <iostream>
#include "utils/TheadPool.h"
```

we also employ the using directive to reduce the typing required 

```cpp
using namespace curan::utilities;
```

the formated_date is a templated function that receives the type of the clock we wish to use associated with a timepoint 
measured in the templated type of clock. The function returns a string with this formated type

```cpp
auto date = formated_date<std::chrono::system_clock>(std::chrono::system_clock::time_point(std::chrono::system_clock::duration(0)));
```

for sanity sake, we print the date, which is associated with the UNIX begining of time 1970-01-01 00:00:00

```cpp
std::cout << "the computed date with the provided time point is (expected \"1970-01-01 00:00:00\"):" << date  << std::endl;
```

## FileStructures

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 

```cpp
#include <iostream>
#include "utils/FileStructures.h"

template <typename T>
T parse_from_file(std::istream &instream)
{
    T file_encoding{instream};
    return file_encoding;
}

std::istream print_to_file(auto type)
{
    std::stringstream mock_file_in_disk;
    mock_file_in_disk << type;
    return mock_file_in_disk;
}

void file_structure_tutorial()
{
    using namespace curan::utilities;
    auto creation_data = formated_date<std::chrono::system_clock>(std::chrono::system_clock::now());

    {
        UltrasoundCalibrationData data{creation_data, Eigen::Matrix<double, 4, 4>::Identity(), 0.0};
        std::cout << "(original  ) ultrasound calibration data: \n"
                  << data << std::endl;
        auto mock_file_in_disk = print_to_file(data);
        auto copydata = parse_from_file<UltrasoundCalibrationData>(mock_file_in_disk);
        std::cout << "(replication) ultrasound calibration data: \n"
                  << copydata << std::endl;
    }

    {
        NeedleCalibrationData data{creation_data, Eigen::Matrix<double, 4, 4>::Identity(), 0.0};
        std::cout << "(original  ) needle calibration data: \n"
                  << data << std::endl;
        auto mock_file_in_disk = print_to_file(data);
        auto copydata = parse_from_file<NeedleCalibrationData>(mock_file_in_disk);
        std::cout << "(replication) needle calibration data: \n"
                  << copydata << std::endl;
    }

    {
        RegistrationData data{creation_data, Eigen::Matrix<double, 4, 4>::Identity(), 0.0, Type::VOLUME};
        std::cout << "(original  ) registration data: \n"
                  << data << std::endl;
        auto mock_file_in_disk = print_to_file(data);
        auto copydata = parse_from_file<RegistrationData>(mock_file_in_disk);
        std::cout << "(replication) registration data: \n"
                  << copydata << std::endl;
    }

    {
        TrajectorySpecificationData data{creation_data, Eigen::Matrix<double, 3, 1>::Ones(), Eigen::Matrix<double, 3, 1>::Ones(), Eigen::Matrix<double, 3, 3>::Identity(), "path_to_moving_image"};
        std::cout << "(original  ) trajectory specification data: \n"
                  << data << std::endl;
        auto mock_file_in_disk = print_to_file(data);
        auto copydata = parse_from_file<TrajectorySpecificationData>(mock_file_in_disk);
        std::cout << "(replication) trajectory specification data: \n"
                  << copydata << std::endl;
    }
}
```

We start by providing the necessary include directories 

```cpp
#include <iostream>
#include "utils/FileStructures.h"
```

we also define two helper functions, one that takes an input stream, file, stringstream etc... that is templated for 
our file encoding types

```cpp
template <typename T>
T parse_from_file(std::istream &instream)
{
    T file_encoding{instream};
    return file_encoding;
}

std::stringstream print_to_file(auto type)
{
    std::stringstream mock_file_in_disk;
    mock_file_in_disk << type;
    return mock_file_in_disk;
}
```

now we focus on the serialization and deserialization types. Note that these types are supposed to propagate
data between executables we force data to always be in a consistent state. The data can be created from, from two sources
from files or from constructors. Any other manipulation is disallowed. Take for example UltrasoundCalibrationData that encodes
the date at which the ultrasound calibration was created, the calibration matrix (in this case we mock it with an identity matrix)
and the error of the calibration. Note that the type can be serializated into a std::ostream 

```cpp
UltrasoundCalibrationData data{creation_data, Eigen::Matrix<double, 4, 4>::Identity(), 0.0};
std::cout << "(original  ) ultrasound calibration data: \n" << data << std::endl;

```

now we mimic what would happen is a normal executable, where we have created the calibration data and we print it to a file, in this
case mocked by a std::stringstream.

```cpp
auto mock_file_in_disk = print_to_file(data);

```
On another executable we would read this into a istream of some type and recreate UltrasoundCalibrationData. Once 
its created the structure can no longer be modified

```cpp
auto copydata = parse_from_file<UltrasoundCalibrationData>(mock_file_in_disk);
std::cout << "(replication) ultrasound calibration data: \n" << copydata << std::endl;
```

Note that the same logic applies for the remaining structures. In this case NeedleCalibrationData takes the date at which the needle was calibrated
the matrix that defines the pose relative to the end-effector of the robot and the calibratione error

```cpp
NeedleCalibrationData data{creation_data, Eigen::Matrix<double, 4, 4>::Identity(), 0.0};
```

this case RegistrationData takes the date at which the registration was defined, the transformation from moving to fixed image,
the registration error and the type of registration used to create the registration type

```cpp
RegistrationData data{creation_data, Eigen::Matrix<double, 4, 4>::Identity(), 0.0, Type::VOLUME};
```

this case TrajectorySpecificationData takes the date at which the needle was calibrated
the vector that defines the target, the entry point and the orientation desired as well as the path to the moving image which is usefull for 
intraoperative navigation

```cpp
TrajectorySpecificationData data{creation_data, Eigen::Matrix<double, 3, 1>::Ones(), Eigen::Matrix<double, 3, 1>::Ones(), Eigen::Matrix<double, 3, 3>::Identity(), "path_to_moving_image"};
```

## Logger

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 

```cpp
#include <iostream>
#include "utils/Logger.h"

void logger_tutorial(){
    using namespace curan::utilities;
    Logger logger{};
    print<Severity::info>("data to print to word{0}\n",1);
    print<Severity::debug>("data to print to word{0}\n",2);
    print<Severity::major_failure>("data to print to word{0}\n",3);
    print<Severity::minor_failure>("data to print to word{0}\n",4);
    print<Severity::warning>("data to print to word{0}\n",5);
    for(int i = 0; i < 5 && logger; ++i)
        logger.processing_function();
}
```

We start by providing the necessary include directories 

```cpp
#include <iostream>
#include "utils/Logger.h"
```

we also employ the using directive to reduce the typing required 

```cpp
using namespace curan::utilities;
```

its important that the code is proliferated with comments, because it allows us to query and understand the code 
at runtime. It is also important that these strings can be turned off or on depending on the criticallity of the code. 
Usually one could define a macro as such


```cpp
//THIS IS NOT PART OF THE TUTORIAL
#define PRINT_INFO(x) std::cout << x << std::endl; 
```

but we avoid this strategy because it is pure text replacement, without string formating options. Instead what we propose
is a templated print function that depending on the level will compile into a print statement or not. internally a macro is defined, 
which can be set at the CMake level informing that particlar executable of the severity level that is desired to be printed. Even if 
we set the print level to the lowest, e.g., print everything, the program will not print to anything because we have not created a logger. 

```cpp
Logger logger{};
```

now lets consider the print statements that ensue

```cpp
print<Severity::info>("data to print to word{0}\n",1);
print<Severity::debug>("data to print to word{0}\n",2);
print<Severity::major_failure>("data to print to word{0}\n",3);
print<Severity::minor_failure>("data to print to word{0}\n",4);
print<Severity::warning>("data to print to word{0}\n",5); 
```

if the macro to be defined at the cmake level is CURAN_WARNING_LEVEL major_failure then once compiled the previous statements will compile to

```cpp
// all other calls are eliminated at compile time
print<Severity::major_failure>("data to print to word{0}\n",3);
```

notice that no string allocation stakes place with the previous statements. Once the logger is created, internally a global pointer is filled that informs all compiled print statements of where to print to.
As soon as this is done, each print statement will lock a mutex of the logger and add this string to the internal list of strings recorded
in the logger. So far nothing is priting anywhere. The last piece of the puzzle is that *somewhere* in the codebase
a thread must consume the strings added internally to the queue of strings inside the logger

```cpp
while(logger)
    logger.processing_function();
```

notice that the there is no danger of continually adding too many strings to the queue because internally it contains a maximum upper bound
that will discard previous strings if achieved

## Reader

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 

```cpp
#include <iostream>
#include "utils/Reader.h"

void reader_tutorial(){
    using namespace curan::utilities;
    char matrix_correct[] = R"(11.14285714 , 12.14285714
                               13.14285714 , 14.14285714)";
    std::stringstream datastream;
    datastream << matrix_correct;
    Eigen::MatrixXd matrix = convert_matrix(datastream,',');
    std::cout << "parsed matrix:\n" << matrix << std::endl;
}
```

We start by providing the necessary include directories 

```cpp
#include <iostream>
#include "utils/Reader.h"
```

we also employ the using directive to reduce the typing required 

```cpp
using namespace curan::utilities;
```

the following is a simple piece of code, we take a stringstream that is filled with a string representing an array

```cpp
std::stringstream datastream;
datastream << matrix_correct;
```

the function convert matrix takes this stream and an optional argument that defines how are number separated between each other

```cpp
Eigen::MatrixXd matrix = convert_matrix(datastream,',');
std::cout << "parsed matrix:\n" << matrix << std::endl;
```

## StringManipulation

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 

```cpp
#include <iostream>
#include "utils/StringManipulation.h"

void string_manipulation_tutorial(){
    using namespace curan::utilities;
    std::cout << "data to output is: " << to_string_with_precision(1324.9123491,1) << std::endl;
    std::cout << "data to output is: " << to_string_with_precision(1324.9123491,2) << std::endl;
    std::cout << "data to output is: " << to_string_with_precision(1324.9123491,3) << std::endl;
    std::cout << "data to output is: " << to_string_with_precision(1324.9123491,4) << std::endl;
    std::cout << "data to output is: " << to_string_with_precision(1324.9123491,5) << std::endl;
    std::cout << "data to output is: " << to_string_with_precision(1324.9123491,6) << std::endl;
}
```

We start by providing the necessary include directories 

```cpp
#include <iostream>
#include "utils/StringManipulation.h"
```

we also employ the using directive to reduce the typing required 

```cpp
using namespace curan::utilities;
```

the to_string_with_precision converts a double presentation and prints the number of chars after the comma. This is usefull when we desire consistency while printing data 

```cpp
std::cout << "data to output is: " << to_string_with_precision(1324.9123491,1) << std::endl;
```
