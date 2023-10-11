---
title: OpenGL [10] 데이터(3) - 쉐이더 스토리지 블록
categories: [OpenGL]
tags: [OpenGL, Python]
comments: true
---

유니폼 블록이 제공하는 버퍼 객체를 읽기 전용으로만 사용하지 않고, **쉐이더 스토리지 블록**을 사용하면 쉐이더에서 쓰기 공간으로 사용할 수도 있다.
뿐만 아니라 쉐이더 스토리지 공간의 멤버에 대해 **어토믹 연산**을 수행할 수도 있다.
쉐이더 스토리지 블록은 크기 제한값이 훨씬 크다.

`uniform` 대신 `buffer` 지시어를 사용한다.

```glsl
layout(binding=0, std430) buffer my_storage_block
{
    vec4 foo;
    vec3 bar;
    int baz[40];
};
```

`glBufferData()` 같은 함수를 사용하여 버퍼에 데이터를 전달할 수도 있다. 쉐이더를 통해 버퍼에 값을 쓸 수 있기 때문에 `glMapBuffer()`를 `GL_READ_ONLY` 접근 모드를 사용하여 호출하면 쉐이더에서 생산한 데이터를 읽을 수 있다.

### 어토믹 메모리 연산

어토믹 연산이란, 메모리 읽기와 쓰기 중간에 인터럽트가 일어나지 않아 결과가 보장되는 것을 말한다.
쉐이더 스토리지 블록의 멤버에 대해 어토믹 연산을 수행하기 위해서는 어토믹 메모리 함수를 사용해야 한다.

자세한 함수 설명은 [_[여기]_](https://registry.khronos.org/OpenGL-Refpages/gl4/html/atomicAdd.xhtml)에서 확인할 수 있다.

### Reference

• OpenGL Super Bible 개정 6판
