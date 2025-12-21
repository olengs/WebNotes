---
title: "gRPC between c++ and python"
summary: "Using gRPC for messaging between a c++ and python program"
date: 2025-11-20
tags: ["gRPC"]
author: ["JC"]
draft: true
weight: 0
ShowToc: true
---

# gRPC installation for C++
Follow the following post: [https://grpc.io/docs/languages/cpp/quickstart/](https://grpc.io/docs/languages/cpp/quickstart/)
Important!!!! ensure the cmake prefix is set up correct, else risking your files installed into your local folder, and might be hard to remove.
IF you are unable to get 
```-DCMAKE_INSTALL_PREFIX=$MY_INSTALL_DIR``` 
to work

use this instead: 
```cmake --install <\path to build> --prefix <\local path to install>```
of
```make install```

you might need to run this command as administrator, just be wary.

# gRPC installation for python
```pip install grpcio```

# gRPC install as part of dockerfile
This part is optional, only if the c++ server is built via docker
```Dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y cmake build-essential git

WORKDIR /deps

#taken from gRPC page
RUN git clone --recurse-submodules -b v1.76.0 --depth 1 --shallow-submodules https://github.com/grpc/grpc

RUN mkdir -p /deps/grpc/build && cd /deps/grpc/build && \
    cmake -DgRPC_INSTALL=ON \
    -DgRPC_BUILD_TESTS=OFF \
    .. && \
    make -j8 install
```

```cmd
docker build -t <tag> .
```

# gRPC basics
