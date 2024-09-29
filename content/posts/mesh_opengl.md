---
title: "Buffers and Rendering in OpenGL"
summary: "Notes on the different types of buffers, and how to render in OpenGL"
date: 2024-09-28
tags: ["c++", "OpenGL", "Graphics", "GLAD", "GLFW", "Mesh"]
author: ["JC"]
draft: false
weight: 0
ShowToc: true
---

This blog will talk about vertex buffers, index buffers, vertex array objects.

### Vertex
A vertex is a point of any mesh, it can include properties eg. position, UV, normal, color etc.

```c++{linenos=true}
struct Position
{
	float x, y, z;
	Position(float x = 0, float y = 0, float z = 0) { Set(x, y, z); }
	void Set(float x, float y, float z) { this->x = x; this->y = y; this->z = z; }
	glm::vec3 tovec3() {
		return glm::vec3(this->x, this->y, this->z);
	}

	friend Position operator*(const glm::mat4x4& lhs, const Position& rhs)
	{
		float b[4];
		for (int i = 0; i < 4; i++)
			//b[i] = lhs.mat[0 * 4 + i] * rhs.x + lhs.mat[1 * 4 + i] * rhs.y + lhs.mat[2 * 4 + i] * rhs.z + lhs.mat[3 * 4 + i] * 1;
			b[i] = lhs[0][i] * rhs.x + lhs[1][i] * rhs.y + lhs[2][i] * rhs.z + lhs[3][i] * 1;
		return Position(b[0], b[1], b[2]);
	}
};

struct TexCoord
{
	float u, v;
	TexCoord(float u = 0, float v = 0) { Set(u, v); }
	void Set(float u, float v) { this->u = u; this->v = v; }
};

struct Vertex
{
	Position pos;
	Color color;
	glm::vec3 normal;
	TexCoord texCoord;
	Vertex() {}
};
```

### Vertex buffers
A vertex buffer object or VBO is an array of vertex data stored in the VRAM. 
Extra: OpenGL will assign you a serial number of type unsigned int which will track the address of the data in the VRAM through the driver.

```c++{linenos=true}
class VertexBuffer final{
public:

	VertexBuffer() : m_VertexBuffer(0) {}

	~VertexBuffer() {
		glDeleteBuffers(1, &m_VertexBuffer);
	}

	void Load(const void* data, unsigned int size) {
        //generate buffer of size 1, retrieving the signature via m_VertexBuffer
		glGenBuffers(1, &m_VertexBuffer);
		glBindBuffer(GL_ARRAY_BUFFER, m_VertexBuffer);
        //copies the data to currently bound buffer, stating size (in bytes) of data and the data address
		glBufferData(GL_ARRAY_BUFFER, size, data, GL_STATIC_DRAW);
	}
	void Bind() const {
		glBindBuffer(GL_ARRAY_BUFFER, m_VertexBuffer);
	}
	void Unbind() const {
		glBindBuffer(GL_ARRAY_BUFFER, 0);
	}

private:
	unsigned int m_VertexBuffer;
};

```

### Index buffers
A Index buffer object or IBO is an array of index data stored in the VRAM. It is almost the same as a vertex buffer, just that it stores only indices. An Index is used to state the order of drawing triangles in relation to the vertex buffer.

```c++{linenos=true}
#ifndef INDEXBUFFER_H
#define INDEXBUFFER_H

#include "Common.h"

class IndexBuffer final{
public:
	IndexBuffer() : m_Count(0), m_IndexBuffer(0) {}
	
	~IndexBuffer() {
		GLCall(glDeleteBuffers(1, &m_IndexBuffer));
	}

	void Load(const unsigned int* data, unsigned int count) {
		ASSERT(sizeof(unsigned int) == sizeof(GLuint));
		m_Count = count;
		glGenBuffers(1, &m_IndexBuffer);
		glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, m_IndexBuffer);
		glBufferData(GL_ELEMENT_ARRAY_BUFFER, m_Count * sizeof(unsigned int), data, GL_STATIC_DRAW);
	}

	void Bind() const {
		glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, m_IndexBuffer);
	}

	void Unbind() const {
		glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, 0);
	}

	unsigned int GetCount() const { return m_Count; }

private:
	unsigned int m_IndexBuffer;
	unsigned int m_Count;
};


#endif //INDEXBUFFER_H
```

### Vertex Array Objects
Vertex Array Objects or VAO are an array of attributes arrays passes the data/addresses that are required by the shaders.

```c++{linenos=true}
//creates the vertex array object
glGenVertexArrays(1, &vertexArray);
//bind the current vertex array
glBindVertexArray(vertexArray);
//enable 3 attributes of the array
glEnableVertexAttribArray(0);
glEnableVertexAttribArray(1);
glEnableVertexAttribArray(2);
//bind vertex buffer required
m_VertexBuffer.Bind();
//set position, color, normal into 0,1,2 of the VAO from VBO
//glVertexAttribPointer(which index of VAO, number of attributes, type of attribute eg. GL_FLOAT, GL_FALSE, size of each vertex object (length of a vertex object), offset from start of every vertex object);
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void*)0);
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void*)sizeof(Position));
glVertexAttribPointer(2, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void*)(sizeof(Position) + sizeof(Color)));
//check if texture exist, if exist, pass in uv
if (textureArray[0] != nullptr)
{
    //bind texture
	textureArray[0]->Bind();
    //set uv coords to attribute index 3
	glEnableVertexAttribArray(3);
	glVertexAttribPointer(3, 2, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void*)(sizeof(Position) + sizeof(Color) + sizeof(glm::vec3)));
}
//bind index buffer
m_IndexBuffer.Bind();
//draw using mode of draw, IBO, VAO, shader
glDrawElements(mode, m_IndexBuffer.GetCount(), GL_UNSIGNED_INT, 0);
//unbind everything
textureArray[0]->Unbind();
if (textureArray[0] != nullptr)
{
	glDisableVertexAttribArray(3);
}
glDisableVertexAttribArray(2);
glDisableVertexAttribArray(1);
glDisableVertexAttribArray(0);

glDeleteVertexArrays(1, &vertexArray);
```

### Rendering
I'll just give a summary of how rendering works, I will continue writing about triangles and shaders in another blog.

Any comments/feedback can be directed to my socials on the main page. Thanks for reading.