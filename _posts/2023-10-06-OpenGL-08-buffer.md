---
title: OpenGL [08] 데이터(1) - 버퍼
categories: [OpenGL]
tags: [OpenGL]
comments: true
---

OpenGL은 데이터를 저장하고 접근하기 위해 2개 형태의 데이터 스토리지를 제공한다.

1. **버퍼**
2. 텍스처

# 버퍼

버퍼는 타입이 정해져 있지 않은 데이터의 연속 공간이다. 버퍼는 **이름**으로 구별된다. 버퍼를 사용하기 전에 OpenGL을 통해 사용할 이름을 예약해야 하고, 그 이름을 사용해서 메모리를 할당하고 그 메모리에 데이터를 저장한다.

버퍼의 이름을 알았다면, 이 이름을 OpenGL contaxt에 attach가 가능하다. attach하기 위해서는 그 이름을 버퍼 바인딩 포인트에 **바인딩**시켜야 한다. 바인딩 포인트는 **타깃**이라고도 부른다.

## 버퍼를 사용하여 메모리 할당하기

버퍼 객체를 사용하여 메모리를 할당하기 위해 사용하는 함수는 `glBufferData()`다.
자세한 함수 설명은 [_[여기]_](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glBufferData.xhtml)에서 확인할 수 있다.

다음 코드는 `glGenBuffers()`를 호출하여 버퍼에 대한 **이름을 예약하는 방법**이다. 그리고 `glBindBuffer()`를 사용하여 그 버퍼가 해당 콘텍스트에 어떻게 바인딩 되는지, 또 어떻게 호출하여 스토리지를 할당하는지 알 수 있다.

```python
# 버퍼에 대한 이름을 생성한다.
vbo = glGenBuffers(1)

# GL_ARRAY_BUFFER 바인딩 포인트를 사용하여 콘텍스트에 바인딩한다.
glBindBuffer(GL_ARRAY_BUFFER, vbo)

# 버퍼에 사용하고자 하는 스토리지의 크기를 반영한다.
glBufferData(GL_ARRAY_BUFFER, 1024 * 1024, NULL, GL_STATIC_DRAW)
```

vbo는 어떤 용도일지는 모르지만 NULL로 초기화된 1메가바이트의 스토리지를 갖도록 초기화된 버퍼 객체의 이름을 담는다.

이 vbo에 데이터를 전달하고 싶다면 `glBufferSubData()`를 사용하여 OpenGL로 데이터를 보내 직접 복사하도록 하는 것이다.
자세한 함수 설명은 [_[여기]_](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glBufferSubData.xhtml)에서 확인할 수 있다.

## 버퍼로부터 버텍스 쉐이더에 입력 전달하기

버택스 배열 객체(vertex array object, VAO)를 하나 만들어 버텍스 배열 상태를 저장하도록 하자.

```python
vao = glGenVertexArray(1)
glBindVertexArray(vao)

glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, vertices)
glEnableVertexAttribArray(0)

```

vao를 생성했고 바인딩했다. OpenGL에 데이터가 버퍼 객체의 어디에 있는지 알려주기 위해 `glVertexAttribPointer()` 함수를 사용하고, 속성을 자동으로 채우도록 `glEnableVertexAttribArray()`를 호출한다. 자세한 함수 설명은 [_[여기]_](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glVertexAttribPointer.xhtml)에서 확인할 수 있다.

이제 버텍스 쉐이더를 보자.

```python
vertex_shader_source = """
#version 430 core
layout(location=0) in vec3 position;

void main()
{
    gl_Position = vec4(position, 1.0);
}
"""
```

이렇게 된다면, `glEnableVertexAttribArray()`가 실행될 당시 바인딩된 버퍼로부터 읽어온 데이터를 자동적으로 버텍스 쉐이더의 첫 번째 속성에 채운다.

## 여러 개의 버텍스 쉐이더 입력 사용하기

버텍스 쉐이더에 색상을 추가해보자.

```python
vertex_shader_source = """
#version 430 core
layout(location=0) in vec3 position;
layout(location=1) in vec3 color;

out vec3 newColor;
void main()
{
    gl_Position = vec4(position, 1.0);
    newColor = color;
}
"""
```

버텍스 쉐이더 입력을 애플리케이션 데이터로 연결시키는 2개 방식이 있다.

1. 독립 속성
2. 인터리브 속성

**속성이 독립**이라는 의미는 다른 버퍼에 위치해 있을 수 있거나 적어도 동일한 버퍼에서 다른 위치에 있을 수 있다는 의미다.

```python
# -1에서 1사이의 랜덤한 정점 좌표 생성
vertices = np.random.uniform(-1.0, 1.0, (20, 3)).astype(np.float32)
# 0에서 1사이의 랜덤한 색상 생성
colors = np.random.uniform(0.0, 1.0, (20, 3)).astype(np.float32)

# VBO 생성
vbo_position = glGenBuffers(1)
vbo_color = glGenBuffers(1)

glBindBuffer(GL_ARRAY_BUFFER, vbo_position)
glBufferData(GL_ARRAY_BUFFER, vertices.nbytes, vertices, GL_STATIC_DRAW)

glBindBuffer(GL_ARRAY_BUFFER, vbo_color)
glBufferData(GL_ARRAY_BUFFER, colors.nbytes, colors, GL_STATIC_DRAW)

# enable vertex attributes
glBindBuffer(GL_ARRAY_BUFFER, vbo_position)
position_location = glGetAttribLocation(shader, "position")
glVertexAttribPointer(position_location, 3, GL_FLOAT, GL_FALSE, 0, ctypes.c_void_p(0))
glEnableVertexAttribArray(position_location)

glBindBuffer(GL_ARRAY_BUFFER, vbo_color)
color_location = glGetAttribLocation(shader, "color")
glVertexAttribPointer(color_location, 3, GL_FLOAT, GL_FALSE, 0, ctypes.c_void_p(0))
glEnableVertexAttribArray(color_location)
```

이렇게 vbo_position과 vbo_color 두 개의 버퍼 객체를 생성하고 배열을 입력하였다.

**인터리브 속성**은 동일한 버퍼의 다른 오프셋 위치에 넣는 것이다.

```python
# -1에서 1사이의 랜덤한 정점 좌표 생성
vertices = np.random.uniform(-1.0, 1.0, (20, 3)).astype(np.float32)
# 0에서 1사이의 랜덤한 색상 생성
colors = np.random.uniform(0.0, 1.0, (20, 3)).astype(np.float32)

# 두 배열을 하나의 배열로 합침
interleaved_data = np.concatenate((vertices, colors))

# VBO 생성
vbo = glGenBuffers(1)
glBindBuffer(GL_ARRAY_BUFFER, vbo)
glBufferData(GL_ARRAY_BUFFER, interleaved_data.nbytes, interleaved_data, GL_STATIC_DRAW)

# enable vertex attributes
position_location = glGetAttribLocation(shader, "position")
glVertexAttribPointer(position_location, 3, GL_FLOAT, GL_FALSE, 0, ctypes.c_void_p(0))
glEnableVertexAttribArray(position_location)

color_location = glGetAttribLocation(shader, "color")
glVertexAttribPointer(color_location, 3, GL_FLOAT, GL_FALSE, 0, ctypes.c_void_p(3 * 4))
glEnableVertexAttribArray(color_location)
```

### Reference

• OpenGL Super Bible 개정 6판
