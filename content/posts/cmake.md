---
title: "CMake"
summary: "Learnings from using cmake"
date: 2024-06-19
tags: ["c++", "third-party", "cmake", "compile"]
author: ["JC"]
draft: false
weight: 0
ShowToc: true
---


### Cmake Installation

To install cmake, go to their [link](https://cmake.org/download/) and download the respective files for your device.

---

### Cmake Setup

To use cmake, create a CMakelists.txt into your project folder.

![](../images/directory.jpg)

<!-- >Add image of directory for cmakelist.txt here</!-->

---

### Building the project

Cmake is a command line tool. The following commands are used to build the project.

```cmake
#to cache build
cmake .

#to build current folder (after baking)
#debug mode
cmake --build . --config Debug

#release mode
cmake --build . --config Release
```

There is also a MSVC tool for cmake that is to be downloaded via the MSVC installer
![](../images/msvc_cmake_install.jpg)

The tool allows you to adjust configuration and build without caching via the UI.
![](../images/msvc_cmake_config.jpg)


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
There are 2 ways of adding a library. First way is to directly link the library to the target

```cmake {linenos=true}
cmake_minimum_required(VERSION "3.19.0")

project(BaseServer)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/BaseServer)
add_executable(BaseServer "BaseServer/main.cpp")

#linking custom .lib library
target_link_libraries(BaseServer ${CMAKE_CURRENT_SOURCE_DIR}/lib/BlankNetLib.lib)
```

The other way is to set the library as a cmake object. The cmake object can be either created via CXX scripts or an imported .lib file.
The following shows adding an imported library file.

```cmake {linenos=true}
cmake_minimum_required(VERSION "3.19.0")

#adding imported lib
add_library(BlankNetLib STATIC IMPORTED)
set_property(TARGET BlankNetLib PROPERTY
             IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/lib/BlankNetLib.lib)

project(BaseServer)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/BaseServer)
add_executable(BaseServer "BaseServer/main.cpp")
target_link_libraries(BaseServer BlankNetLib)
```

The following shows adding a self created library.

```cmake {linenos=true}
cmake_minimum_required(VERSION "3.19.0")

set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreadedDebug$<$<CONFIG:DEBUG>:DEBUG>")
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:RELEASE>:RELEASE>")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)

#creating new lib
project(BlankNetLib)
set(CMAKE_CXX_STANDARD 17)
add_library(BlankNetLib STATIC 
${CMAKE_CURRENT_SOURCE_DIR}/BlankNetLib/Client.cpp
${CMAKE_CURRENT_SOURCE_DIR}/BlankNetLib/Client.h
${CMAKE_CURRENT_SOURCE_DIR}/BlankNetLib/Server.cpp
${CMAKE_CURRENT_SOURCE_DIR}/BlankNetLib/Server.h
${CMAKE_CURRENT_SOURCE_DIR}/BlankNetLib/ThreadPool.cpp
${CMAKE_CURRENT_SOURCE_DIR}/BlankNetLib/ThreadPool.h
${CMAKE_CURRENT_SOURCE_DIR}/BlankNetLib/Common.h
${CMAKE_CURRENT_SOURCE_DIR}/JCCommon/Task.hpp
${CMAKE_CURRENT_SOURCE_DIR}/JCCommon/TaskPool.hpp
)

project(BaseServer)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/BaseServer)
add_executable(BaseServer "BaseServer/main.cpp")

target_link_libraries(BaseServer BlankNetLib)
```

### Adding subdirectories

Remember the other cmakelists.txt you saw in my directory folder?
You can create subdirectories with cmake

subdirectories is basically a #include which copies all the subdirectory to the compiled cmakelists.txt

Ensure that the input of the subdirectory is the folder name of the subdirectory 1 folder layer in.

```cmake {linenos=true}
cmake_minimum_required(VERSION "3.19.0")

add_subdirectory(BlankNetLib)

project(BaseServer)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/BaseServer)
add_executable(BaseServer "BaseServer/main.cpp")
target_link_libraries(BaseServer BlankNetLib)
```


### Additional options (MSVC)

Some additioal options that can be added:


1. multithreaded flag for static or dynamic builds
``` cmake{linenos=true}
#compiles with multithreading flag for debug mode statically-linked runtime library.
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreadedDebug$<$<CONFIG:DEBUG>:DEBUG>")

#compiles with multithreading flag for debug mode statically-linked runtime library.
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:RELEASE>:RELEASE>")

#compiles with multithreading flag for debug mode dynamically-linked runtime library.
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreadedDebugDLL$<$<CONFIG:DEBUG>:DEBUG>")

#compiles with multithreading flag for debug mode dynamically-linked runtime library.
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreadedDLL$<$<CONFIG:RELEASE>:RELEASE>")
```


### Additional options (Project)

Some additional options for the project that can be added


1. setting output directory for library output and exe output
``` cmake{linenos=true}
#where to put your static libraries
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY <dir>)

#where to put your shared libraries
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY <dir>)

#where to put your executables
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY <dir>)
```


---






