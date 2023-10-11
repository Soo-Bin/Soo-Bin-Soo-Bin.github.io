---
title: OpenGL [09] 데이터(2) - 유니폼
categories: [OpenGL]
tags: [OpenGL, Python]
comments: true
---

비록 스토리지의 형태는 아니지만, 유니폼을 사용하면 어플이 직접 쉐이더 스테이지로 데이터를 전달할 수 있다.
어떻게 선언하느냐에 따라 2개의 방식으로 나뉜다.

1. 디폴트 블록에 선언
2. 유니폼 블록에 선언

# 디폴트 블록 유니폼

유니폼은 전체 프리미티브 배치를 렌더링하는 동안 또는 그 후에도 변하지 않고 균일하게 남아 있는 데이터를 쉐이더에 전달하는 방법이다.
어떤 쉐이더 변수라도 유니폼으로 선언할 수 있으며, `uniform` 키워드를 넣기만 하면 된다.

```glsl
uniform float fTime;
uniform int iIndex;
uniform vec4 vColorValue;
uniform mat4 mvpMatrix;
```

유니폼은 항상 **상수**로 간주된다. 쉐이더 코드에서 할당할 수 없다.
만약 동일한 유니폼을 여러 쉐이더 스테이지에서 선언한다면, 각각의 스테이지는 동일한 유니폼값을 갖게 된다.

## 유니폼 위치 설정하기

버텍스 속성과 마찬가지로 프로그램 객체 내의 **위치**를 통해 유니폼을 참조한다.
**위치 레이아웃 지시어**를 사용하면 유니폼의 위치를 지정할 수 있다.

```glsl
layout(location=10) uniform vec4 uni;
```

여기서 `uni`는 위치 10에 할당된다. 만약 쉐이더에서 유니폼의 위치를 지정하지 않으면 OpenGL이 자동으로 할당해준다.
어떤 위치가 할당되었는지 확인하려면 `glGetUniformLocation()`을 사용하면 된다.

```python
uni_location = glGetUniformLocation(shader, "uni")
```

리턴 위치값이 `-1`이라면, 이는 유니폼 이름이 프로그램에 없다는 의미다. 쉐이더 변수 이름은 대소문자를 구별하므로, 위치를 검색하려는 유니폼 이름의 대소문자에 유의해야 한다.

## 유니폼에 값 설정하기

유니폼에 값을 설정하고 싶다면 `glUniform*()` 함수의 변종을 통해서 설정 가능하다.
자세한 함수 설명은 [_[여기]_](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glUniform.xhtml)에서 확인할 수 있다.

# 유니폼 블록

어플에서 많은 쉐이더를 사용한다면, 모든 쉐이더에 대해 유니폼을 설정해야 한다. 즉, 수많은 `glUniform*()` 함수를 호출하게 된다.
이 함수를 매번 호출하는 성능 저하를 막기 위해, 많은 유니폼을 업데이트하는 작업을 더 단순하게 하기 위해, 그리고 다른 프로그램들 간에 유니폼을 더 쉽게 공유하기 위해, OpenGL은 유니폼을 그룹화할 수 있게 하고 블록 전체를 버퍼 객체에 저장할 수 있게 했다.

[**인터페이스 블록**](../OpenGL-05-interface-block)과 유사해보이지만, `in`이나 `out` 키워드 대신에 `uniform` 키워드를 사용한다.
다음은 그 예시이다.

```glsl
uniform TransformBlock
{
    float scale;
    vec3 translation;
    float rotation[3];
    mat4 projection_matrix;
} transform;
```

### 유니폼 블록 만들기

일반적으로 어플의 임무는 `glBufferData()`나 `glMapBuffer()` 같은 함수를 사용하여 버퍼 객체에 데이터를 채우는 것이다. 그렇다면 문제는 **'어떻게 버퍼의 데이터를 채울까'** 하는 것이다. 실제로 여기에는 2개의 옵션이 있다. 어떤 것을 선택하든 장단점이 있다.

1. 데이터의 레이아웃에 의존한다. (표준적인 방법)
   > 어플이 버퍼로 그냥 데이터를 복사하고 멤버의 블록 내 위치가 그대로 일치한다고 가정.
   > 때문에 여러 멤버 사이에 빈 공간을 둘 여지가 있음.
   >
   > 실제 필요한 양보다 버퍼를 더 크게 잡아야 하는 경우도 있으며, 편의성에 비해 성능을 손해 볼 수도 있음.
   > 하지만 대부분의 경우에 안전한 방법.
2. 데이터가 어디에 위치할지 OpenGL이 결정한다.
   > 가장 효율적인 쉐이더를 만들 수 있지만, OpenGL이 읽을 데이터가 어디에 위치하는지 어플이 알아내야 함. 이 방식으로 쉐이더 성능이 개선되지도 하지만, 어플 입장에서는 할 일이 더 많아진다는 뜻.
   >
   > 저장된 데이터는 **공유된** 레이아웃 형태로 정렬. 공유 레이아웃을 사용하기 위해서는 유니폼 블록 멤버에 대한 버퍼 객체 상의 위치를 어플이 결정해야 함.

**표준 레이아웃**을 사용한다고 하려면 유니폼 블록을 레이아웃 지시어로 선언해야 한다.

```glsl
layout(std140) uniform TransformBlock
{
    float scale;
    vec3 translation;
    float rotation[3];
    mat4 projection_matrix;
} transform;
```

유니폼 블록이 일단 표준 또는 `std140`으로 선언되면, 각 블록의 멤버는 버퍼에 기정의된 양만큼 공간을 차지하고, 규칙에 따라 오프셋만큼 지난 위치에서 시작한다.

규칙을 요약하면 다음과 같다고 한다.

1. 버퍼 내에서 N바이트를 차지하는 타입은 N의 배수인 바이트에서 시작한다. 즉 int, float, bool 등 32비트 즉 4바이트를 차지하는 타입은 4의 배수인 바이트에서 시작한다.
2. N바이트 타입의 2원소 벡터는 2N의 배수 바이트에서 시작한다.
3. N바이트 타입의 3, 4원소 벡터는 4N의 배수 바이트에서 시작한다.
4. 스칼라 또는 벡터의 배열 각 멤버는 동일한 규칙에 하에 위치가 정해지지만, vec4의 정렬 방식을 따라 올림 처리된다.
   이는 vec4 이외의 타입의 배열 (그리고 Nx4 행렬)이 조밀하게 배치되지 않으며 원소간에 공백이 생긴다는 것을 뜻한다.
5. 행렬은 벡터의 짧은 배열처럼 취급되고 행렬의 배열은 벡터의 매우 긴 배열로 취급된다.
6. 구조체와 구조체의 배열은 크기가 가장 큰 멤버에게 필요한 경계에서 시작하며 vec4의 크기로 반올림된다.

모든 규칙은 https://www.opengl.org/registry/specs/ARB/uniform_buffer_object.txt 를 참조하면 된다.

유니폼 블록 멤버에 대한 정보를 얻고 싶으면 아래와 같이 하면 된다.

```python
vertex_shader_source = """
#version 430 core
layout(std140) uniform TransformBlock
{
    float scale;
    vec3 translation;
    float rotation[3];
    mat4 projection_matrix;
} transform;

void main()
{
}
"""
fragment_shader_source = """
"""

vertex_shader = shaders.compileShader(vertex_shader_source, GL_VERTEX_SHADER)
fragment_shader = shaders.compileShader(fragment_shader_source, GL_FRAGMENT_SHADER)
shader = shaders.compileProgram(vertex_shader, fragment_shader)

# Query the number of active Uniforms:
uniform_block_index = glGetUniformBlockIndex(shader, "TransformBlock")

num_active = GLint()
indices = (GLuint * num_active.value)()
indices_ptr = c.cast(c.addressof(indices), c.POINTER(GLint))
glGetActiveUniformBlockiv(shader, uniform_block_index, GL_UNIFORM_BLOCK_ACTIVE_UNIFORMS, num_active)
glGetActiveUniformBlockiv(shader, uniform_block_index, GL_UNIFORM_BLOCK_ACTIVE_UNIFORM_INDICES, indices_ptr)

# Create objects and pointers for query values:
offsets = (GLint * num_active.value)()
gl_types = (GLuint * num_active.value)()
offsets_ptr = c.cast(c.addressof(offsets), c.POINTER(GLint))
gl_types_ptr = c.cast(c.addressof(gl_types), c.POINTER(GLint))

# Query the indices, offsets, and types uniforms:
glGetActiveUniformsiv(shader, num_active.value, indices, GL_UNIFORM_OFFSET, offsets_ptr)
glGetActiveUniformsiv(shader, num_active.value, indices, GL_UNIFORM_TYPE, gl_types_ptr)

# 두 배열을 합침
combined = list(zip(offsets, gl_types))

# 특정 키를 기준으로 정렬
sorted_combined = sorted(combined, key=lambda x: x[0])  # 첫 번째 요소를 기준으로 정렬

print(sorted_combined)
```

![gl_uniform_block_result](/assets/img/post/gl_uniform_block_result.png)

여기서 `GL_UNIFORM_OFFSET`은 블록 내 유니폼의 오프셋이고 `GL_UNIFORM_TYPE`은 유니폼의 데이터 타입을 GLenum 형태로 얻는다.

유니폼 데이터 타입에 대해 알고 싶으면 [_[여기]_](https://github.com/KhronosGroup/glTF/blob/main/specification/1.0/README.md#parametertype-white_check_mark) 를 확인하면 된다.

**공유 레이아웃**을 사용한다고 하면 좀더 복잡한 과정을 거친다. 여기서는 생략하겠다.

어쨋든 버퍼에 데이터를 제대로 배치하기 위해서는 코드가 꽤 많이 필요하기 때문에 **표준 레이아웃을 권장**한다.
물론 이 방식을 사용하면 쉐이더 성능이 약간 떨어지긴 하겠지만, 코드 복잡도나 어플 성능면에서 그만한 가치가 있다.

## 유니폼 블록 바인딩하기

하나의 프로그램이 사용할 수 있는 유니폼 블록의 최대 개수에는 제한이 있다.
특정 쉐이더 스테이지에서 사용할 수 있는 유니폼 블록의 최대 개수도 제한이 있다.
이 제한값들은 `glGetIntegerv()`에 특정 인자를 전달하면 확인할 수 있다.

어떤 패킹 모드를 선택하더라도, 데이터 버퍼를 유니폼 블록에 바인딩할 수 있다. 2개 단계의 과정이 필요한데,

1. 유니폼 블록을 바인딩 포인트에 할당한 다음에
2. 이 바인딩 포인트에 버퍼를 바인딩하면

결과적으로 버퍼가 유니폼 블록에 연결된다.

바인딩 포인트 할당은

1. `glUniformBlockBinding()`함수를 코드에서 사용하거나,
2. `layout(std140, binding=2) uniform TransformBlock`으로 쉐이더에서 지정할 수 있다.

바인딩 포인트에 버퍼 할당은 `glBindBufferBase()` 함수를 사용하면 된다.

# 유니폼을 사용하여 지오메트리 변환하기

예제 프로그램을 만들 것이다. 정육면체가 회전하면서 돌아다니는 형태다.

![gl_roate_cube](/assets/img/post/gl_roate_cube.gif)

전체 코드는 [_[깃허브]_](https://github.com/Soo-Bin/helloGL/blob/main/09-uniform-block-1.py) 를 확인하면 된다.

### Reference

• OpenGL Super Bible 개정 6판
