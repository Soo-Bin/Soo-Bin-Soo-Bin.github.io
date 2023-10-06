---
title: OpenGL 인터페이스 블록
categories: [OpenGL]
tags: [OpenGL, Python]
comments: true
---

# 스테이지 간 데이터 전달

in 키워드를 사용해서 버텍스 속성을 생성하고, 코드에서 버텍스 쉐이더로 데이터를 전달할 수 있다. 예를 들어 `glVertexAttrib*()`.

한 쉐이더에서 out 키워드를 사용하여 데이터를 출력하면 다음 스테이지에서 in 키워드로 선언된 유사한 이름의 변수로 보내진다.

_참고-[OpenGL 점 선 면 그리기](./2023-04-12-OpenGL-sec.md)_

# 인터페이스 블록

실제 애플리케이션에서는 많은 다른 종류의 데이터를 스테이지별로 전송한다. 이러한 데이터에는 배열, 구조체, 그리고 다른 복잡한 형태의 변수들이 포함된다. 이를 위해 여러 변수를 하나의 **인터페이스 블록**으로 그룹화할 수 있다.

```python
import OpenGL.GL.shaders as shaders

vertex_shader_source = """
#version 330
in vec3 position;
in vec3 color;

// VS_OUT을 출력 인터페이스 블록으로 선언
out VS_OUT
{
    vec3 newColor;
} vs_out;

void main()
{
    gl_Position = vec4(position, 1.0);
    vs_out.newColor = color;
}
"""

fragment_shader_source = """
#version 330

// VS_OUT을 입력 인터페이스 블록으로 선언
in VS_OUT
{
    vec3 newColor;
} fs_in;

void main()
{
    gl_FragColor = vec4(fs_in.newColor, 1.0);
}
"""

vertex_shader = shaders.compileShader(vertex_shader_source, GL_VERTEX_SHADER)
fragment_shader = shaders.compileShader(fragment_shader_source, GL_FRAGMENT_SHADER)
shader = shaders.compileProgram(vertex_shader, fragment_shader)
```

![gl_interface_block](/assets/img/post/gl_interface_block.png)

### Reference

• OpenGL Super Bible 개정 6판
