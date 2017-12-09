<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [LoadShaders](#loadshaders)
	* [glShadersource "整理source code"](#glshadersource-整理source-code)
	* [glCompileShader "进行编译"](#glcompileshader-进行编译)
	* [glAttachShader 和 glLinkProgram](#glattachshader-和-gllinkprogram)
	* [状态检查](#状态检查)
	* [举个超级吓人的例子：（文字超级长）](#举个超级吓人的例子文字超级长)

<!-- /code_chunk_output -->
Thanks to [极客学院 OpenGL教程翻译 第四课 Shaders](http://wiki.jikexueyuan.com/project/modern-opengl-tutorial/tutorial4.html)
# LoadShaders
这里是两个参数
（可以理解为输入和输出的样式？）
先读取文件为一个string，然后进行“遍历”

- vertex_file_path
    输入顶点的信息
- fragment_file_path
    输入框架的信息

到第36行读取结束。（kotlin写多了表示你这玩意只需要一行，C++你这是浪费时间啊……做好哪来那么多破事……重复造轮子）
一行能解决的事情浪费10行
## glShadersource "整理source code"
- 对 Shader 对象进行编译之前我们必须为其指定 Shader 源程序，glShadersource 函数需要一个 Shader 对象作为参数，这个函数为 Shader 源程序的指定提供了一种很灵活的方法。源程序可以分布在多个字符串数组中，你需要提供这些数组的指针数组以及一个用于存放对应数组长度的整数数组。
- 为了简单起见，整个着色器代码我们仅使用一个字符串数组，源程序指针数组和和长度数组都只有一个元素。`glShadersource(ShaderObj, 1, p, Lengths)`的第二个参数是这两个数组的元素个数。
- （相当于预编译？？？然后再进行整理）
## glCompileShader "进行编译"
```c++
glCompileShader(VertexShaderID);
glCompileShader(FragmentShaderID);
```

## glAttachShader 和 glLinkProgram
最后，我们将编译之后的 Shader 对象附加到程序对象上，这和在 Makefile 中链接一个对象链表类似。因为我们这儿没有 Makefile 所以通过编程来实现这种功能。只有被附加到程序对象上的 Shader 对象才会参与链接过程。
```c++
GLuint ProgramID = glCreateProgram();
glAttachShader(ProgramID, VertexShaderID);
glAttachShader(ProgramID, FragmentShaderID);
glLinkProgram(ProgramID);
```

## 状态检查
```c++
glGetProgramiv(ProgramID, GL_LINK_STATUS, &Result);
glGetProgramiv(ProgramID, GL_INFO_LOG_LENGTH, &InfoLogLength);
```
glValidateProgram(ShaderProgram);



我们可以通过将多个 Shader 对象链接到一起来创建你自己的着色器，但是在每个着色器阶段（VS，GS，FS）只能有一个 main 函数作为着色器的入口点。例如你可以通过多个函数创建一个灯光库，并且将它链接到你所提供的 Shader 程序对象上，其中一个函数名为 main 函数。

## 举个超级吓人的例子：（文字超级长）
```
gl_Position = vec4(0.5 * Position.x, 0.5 *Position.y, Position.z, 1.0);
```
- 在这里我们通过编码对传入的顶点位置进行变换
-我们将顶点的 X、Y 分量的值减半而保持 Z 方向值不变
- `gl-Position` 是一个特殊的内置变量
    - 他能够存放齐次（包含X,Y,Z和W分量）顶点坐标。
    - 在光栅化过程中系统会寻找这个变量并使用它作为顶点在屏幕上的位置（需要经过一些矩阵变换）。
    - 将顶点的X,Y分量减半意味着我们将会看到一个面积只有前面教程中的四分之一的三角形。
    - 需要注意的是我们将W分量设置为1.0。这对于三角形的正确显示是非常重要的。
- 实现从3D到2D的投影变换实际上是在两个不同的阶段实现的。
    首先你需要让所有的顶点都乘上投影矩阵之后 GPU 在对其进行光栅化之前自动对位置属性（Position）执行透视分割。
    - 这意味着 gl-Position 中的所有分量都会除以W分量。
    - 在本节的顶点着色器中我们并没有进行任何与投影有关的操作，但是我们不能禁用透视分割阶段。
    - 不论我们从顶点着色器中输出 gl_Position 中的任何值都会被除以其W分量。为了得到我们所期望的结果我们需要记住这一点。
    - 为了避免透视分割对结果产生影响我们将W分量设置为1.0。除以1.0并不会影响 Position 向量中的其他分量，并使其依旧处于规范化盒子中。 
- 如果所有部分都正确工作，那么这三个顶点(-0.5, -0.5), (0.5, -0.5) 和(0.0, 0.5)会进入光栅化阶段。
- 由于所有点都正好处于规范化盒子之中，所以裁剪器并不需要做任何事。- 这些值会被映射到屏幕坐标系中
    - 之后光栅化阶段开始遍历处于三角形内部的所有点。
    - 对于三角形中的每个点都会对其执行片元着色器，下面的代码就来自于片元着色器。