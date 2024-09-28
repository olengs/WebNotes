---
title: "Creating a OpenGL instance"
summary: "My experience with creating OpenGL instance on both windows and mac OS"
date: 2024-09-28
tags: ["c++", "OpenGL", "Graphics", "GLAD", "GLFW"]
author: ["JC"]
draft: false
weight: 0
ShowToc: true
---

This blog will showcase how to create an openGL instance in both windows and mac OS

### OpenGL
I will be talking about creating an OpenGL instance, and the dependencies needed for this project.

1. Dependencies for OpenGL
What you will need:
- Download [GLFW](https://www.glfw.org/download.html) and [GLAD](https://glad.dav1d.de). 
Ensure the GLFW and GLAD that has been downloaded supports your current GL version. For OpenGL, the choose the GLAD gl API and not gles.
After you download your files, place them into your c++ projects. My files structure looks like this:
note that the libglfw3.a file is used for mac OS
```
└── Engine
    ├── Includes
    │   ├── glad
    │   │   └── glad.h
    │   ├── GLFW
    │   │   ├── glfw3.h
    │   │   └── glfw3native.h
    │   └── KHR
    │       └── khrplatform.h
    ├── Libs
    │   ├── glfw3.lib
    │   └── libglfw3.a
    └── Vendor
        └── glad.c
```
2. Create the OpenGL instance

First we will create the GL window
```c++{linenos=true}
    //only win32 features
	#ifdef WIN32
	//memory_leaks check
	_CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF);

	//hide debug_window
	::ShowWindow(::GetConsoleWindow(), SW_SHOWMINIMIZED);
	(void)SetLayeredWindowAttributes(GetConsoleWindow(), NULL, 230, LWA_ALPHA);
	(void)SetWindowPos(
		GetConsoleWindow(),
		0,
		0,
		0,
		GetSystemMetrics(SM_CXFULLSCREEN),
		GetSystemMetrics(SM_CYFULLSCREEN),
		0
	);
	#endif
    //init glfw
    glfwInit();
    //set openGL version
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
	glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
#ifdef __APPLE__
	glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
#endif
    //create glfw window
	window = glfwCreateWindow(width, height, m_WindowName, NULL, NULL);
	if (window == NULL)
	{
		std::cout << "Failed to create GLFW window" << std::endl;
		glfwTerminate();
		return false;
	}
    //focus on created window
	glfwMakeContextCurrent(window);
    //initialise GLAD
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
    {
        std::cout << "Failed to initialize GLAD" << std::endl;
        return false;
    }
    std::cout << "GL Version: " << glGetString(GL_VERSION) << std::endl;
    //set window size
    glViewport(0, 0, width, height);
    //add callback for window size adjustment
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
```

Then add an update loop
```c++{linenos=true}
while (!glfwWindowShouldClose(window))
{
    //input
    ProcessInput();

    glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT);

    //check and call event and swap the buffers
    glfwPollEvents();
    glfwSwapBuffers(window);
}

void ProcessInput()
{
	if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
		glfwSetWindowShouldClose(window, true);
}
```

### Building with cmake
This assumes you already know how to use cmake, if not you can look at my [cmake] guide.

```cmake{linenos=true}
cmake_minimum_required(VERSION 3.20.0)

if(MSVC)
	set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreadedDebug$<$<CONFIG:DEBUG>:DEBUG>")
	set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:RELEASE>:RELEASE>")
endif()

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin)

#Get GLM from git
include(FetchContent)

FetchContent_Declare(
	glm
	GIT_REPOSITORY	https://github.com/g-truc/glm.git
	GIT_TAG 	bf71a834948186f4097caa076cd2663c69a10e1e #refs/tags/1.0.1
)

FetchContent_MakeAvailable(glm)

#declare APPLE definition
if(APPLE)
add_compile_definitions(__APPLE__)
endif()

#create GLFW cmake var and include the respective windows/mac files
project(GLFW)
add_library(GLFW STATIC IMPORTED)
if(WIN32)
set_property(TARGET GLFW
PROPERTY IMPORTED_LOCATION "${CMAKE_CURRENT_SOURCE_DIR}/Libs/glfw3.lib")
elseif(APPLE)
set_property(TARGET GLFW
PROPERTY IMPORTED_LOCATION "${CMAKE_CURRENT_SOURCE_DIR}/Libs/libglfw3.a")
endif()

project(Engine)
set(CMAKE_CXX_STANDARD 20)
add_executable(Engine STATIC 
#add all to build files here
)
#link macOS additional libraries
if(APPLE)
target_link_libraries(Engine "-framework QuartzCore" "-framework Cocoa" "-framework OpenGL" "-framework IOKit")
endif()
#link GLFW
target_link_libraries(Engine GLFW)

target_include_directories(Engine 
PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}
PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/Includes)
target_link_libraries(Engine glm::glm)
```

Once you finish building. You should see a blue screen and that is all for this blog.

Any comments or questions can be directed to me via my socials on the main page.