

Пример:

```CMake
cmake_minimum_required(VERSION 3.13)  # CMake version check
project(ncursesExample)               # Create project "simple_example"
set(CMAKE_CXX_STANDARD 14)            # Enable c++14 standard

# Add main.cpp file of project root directory as source file
set(SOURCE_FILES src/main.cpp)

# Add executable target with source files listed in SOURCE_FILES variable
add_executable(ncursesExample ${SOURCE_FILES})
find_package(Curses REQUIRED)
target_link_libraries(ncursesExample ${CURSES_LIBRARY})
```

Сборка проекта:

```sh
mkdir build
cd build
cmake ..
cmake --build .
```