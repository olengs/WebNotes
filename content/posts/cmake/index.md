---
title: "CMake"
summary: "Learnings from using cmake"
date: 2024-06-19
tags: ["c++", "third-party"]
layout: "categories"
author: ["JC"]
draft: true
weight: 5
---


### Cmake Installation

To install cmake, go to their [link](https://cmake.org/download/) and download the respective files for your device.

---

### Cmake Setup

To use cmake, create a CMakelists.txt into your project folder.

<!-- >Add image of directory for cmakelist.txt here</!-->

---

### Cmakelists.txt Code

Your cmake lists is what cmake will use to compile your project. This is how cmake knows where to find your files to build, project, project settings and dependencies.


Firstly, we declare the minimum version required of cmake

```cmake {linenos=true}
cmake_minimum_required(VERSION "3.19.0")
```

Then we have to create a project to build, set the project c++ version

```cmake {linenos=true}
cmake_minimum_required(VERSION "3.19.0")


project(BaseServer)

#set(CMAKE_CXX_STANDARD 14)
#set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD 20)
```

Next we set the runtime out directory to a file location, for this project would be $(CMAKE_CURRENT_SOURCE_DIR) which is the directory of the cmakelists.txt file, take not that the syntax for cmake functions to separate arguments is a space(" ").

```cmake {linenos=true}
cmake_minimum_required(VERSION "3.19.0")


project(BaseServer)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/BaseServer)
```

And add the files required to build the executable, take not that the syntax for cmake functions to separate arguments is a space(" ").
```cmake {linenos=true}
cmake_minimum_required(VERSION "3.19.0")


project(BaseServer)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/BaseServer)
add_executable(BaseServer "BaseServer/main.cpp")
```

---

### Adding a library
To add a library, you first have to create a cmake object for the library.

```cmake {lineos=true}
cmake_minimum_required(VERSION "3.19.0")


set(myLib ${CMAKE_CURRENT_SOURCE_DIR}/lib/myLib.lib)

project(BaseServer)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/BaseServer)
add_executable(BaseServer "BaseServer/main.cpp")
```

and then link the library to your project after adding the executable
```cmake {lineos=true}
cmake_minimum_required(VERSION "3.19.0")


set(BlankNetLib ${CMAKE_CURRENT_SOURCE_DIR}/lib/BlankNet.lib)

project(BaseServer)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/BaseServer)
add_executable(BaseServer "BaseServer/main.cpp")
target_link_libraries(BaseServer BlankNetLib)
```


```cmake {linenos=true}
cmake_minimum_required(VERSION "3.19.0")

set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreadedDebug$<$<CONFIG:DEBUG>:DEBUG>")
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:RELEASE>:RELEASE>")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)

add_subdirectory(BlankNetLib)

project(BaseServer)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/BaseServer)
add_executable(BaseServer "BaseServer/main.cpp")
target_link_libraries(BaseServer BlankNetLib)

project(BaseClient)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/BaseClient)
add_executable(BaseClient "BaseClient/main.cpp")
target_link_libraries(BaseClient BlankNetLib)

project(Networking)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/Networking)
add_executable(Networking 
"main.cpp" 
"Testing/Task.hpp"
)
set_target_properties(Networking PROPERTIES CXX_STANDARD 17
                                       CXX_STANDARD_REQUIRED ON
                                       CXX_EXTENSIONS ON)
target_link_libraries(Networking BlankNetLib)
```

---

![regular](images/imgtest.jpg)

---






