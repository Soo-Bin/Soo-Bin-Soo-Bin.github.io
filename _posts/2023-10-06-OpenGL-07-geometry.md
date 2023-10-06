---
title: OpenGL [07] 지오메트리 쉐이더
categories: [OpenGL]
tags: [OpenGL, Python]
comments: true
---

지오메트리 쉐이더는 프리미티브당 한 번 수행되며, 수행되는 프리미티브를 구성하는 모든 버텍스에 대한 입력 버텍스 데이터에 접근할 수 있다.
다른 고유한 기능으로 파이프라인 중간에 프리미티브의 모드를 변경하는 기능이 있다. 예를 들면 삼각형들을 입력으로 하여 여러 점이나 선을 출력으로 만들어낼 수 있다.

아래 코드와 같이 말이다.

```python
geometry_shader_source = """
#version 430 core
layout(triangles) in;
layout(points, max_vertices=3) out;

void main()
{
    int i;

    for (i=0; i<gl_in.length(); i++)
    {
        gl_Position = gl_in[i].gl_Position;
        EmitVertex();
    }
}
"""

geometry_shader = shaders.compileShader(geometry_shader_source, GL_GEOMETRY_SHADER)
shader = shaders.compileProgram(..., geometry_shader, ...)
```

첫 번째 레이아웃 지시어를 통해 지오메트리 쉐이더가 삼각형을 입력으로 받는다는 것을 알 수 있다.

두 번째 레이아웃 지시어는 지오메트리 쉐이더가 점을 생성할 때 각 쉐이더가 생성하는 점의 최대 개수가 3이라는 것을 OpenGL에 알려준다.

---

코드 결과는 아래와 같다.

![gl_geometry_shader](/assets/img/post/gl_geometry_shader.png)

### Reference

• OpenGL Super Bible 개정 6판
