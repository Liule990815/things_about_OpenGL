## 1.1什么是openGL
### openGL是一种API，对硬件设备进行访问的软件库。硬件无关，不考虑任何执行窗口任务或者用户输入的函数。不提供任何三维物体模型，以及读取图像文件的操作。通过一系列几何图元（点、线、三角形、面片）进行操作。
### 用来渲染图象的OpenGL的程序的主要操作:
* 从OpenGL的几何元图设置数据，构建形状；
* 着色器（shader）对输入图形进行计算，判断位置、颜色，以及其它属性；
* 将输入图元的数学描述转换为与屏幕位置对应的像素片元，即光栅化；
* 针对光栅化产生的每个片元，执行片元着色器，确定片元最终的颜色和位置；
* 对每个片元执行额外操作，可见、融合等。 
### OpenGL是使用而客户端-服务端的形式实现的。把图形加速器作为服务端，把用户程序作为客户端来看待。

## 1.2初识OpenGL程序
### 一些基础的图形学名词：
* 渲染(render):从模型创建最终图像的过程
* 光栅化
* 光线追踪(ray tracing)
* 光子映射(photo mapping)
* 路径跟踪(path tracing)
* 基于图像的渲染(image-based rendering)
* 模型/场景对象(model)：通过几何图元（点、线、三角形）来构建的
* 顶点(vertex):模型与顶点也存在着各种对应关系
* 着色器：图形硬件所执行的一类特殊函数，专为图形处理单元编译的小型程序
* 着色阶段(shader stage):最常用的包括顶点着色器(vertex shader)以及片元着色器，前者用于顶点数据处理，后者用于处理光栅化后的片元数据
* 最终生成的图像包含所有的像素点(pixel)，并保存到帧缓存(framebuffer)中，是图形设备的独立内存，可做直接映射到屏幕。
### 代码示例：
```c++
//////////////////////////////////////////////////////////////////////////////
//
//  Triangles.cpp
//
//////////////////////////////////////////////////////////////////////////////

#include "vgl.h"
#include "LoadShaders.h"

enum VAO_IDs { Triangles, NumVAOs };
enum Buffer_IDs { ArrayBuffer, NumBuffers };
enum Attrib_IDs { vPosition = 0 };

GLuint  VAOs[NumVAOs];
GLuint  Buffers[NumBuffers];

const GLuint  NumVertices = 6;

//----------------------------------------------------------------------------
//
// init:设置程序中所用到的数据
//

void
init( void )
{
    glGenVertexArrays( NumVAOs, VAOs );
    glBindVertexArray( VAOs[Triangles] );

    GLfloat  vertices[NumVertices][2] = {   //两个三角形的位置信息
        { -0.90f, -0.90f }, {  0.85f, -0.90f }, { -0.90f,  0.85f },  // Triangle 1
        {  0.90f, -0.85f }, {  0.90f,  0.90f }, { -0.85f,  0.90f }   // Triangle 2
    };

    glCreateBuffers( NumBuffers, Buffers );
    glBindBuffer( GL_ARRAY_BUFFER, Buffers[ArrayBuffer] );
    glBufferStorage( GL_ARRAY_BUFFER, sizeof(vertices), vertices, 0);

    ShaderInfo  shaders[] =
    {
        { GL_VERTEX_SHADER, "media/shaders/triangles/triangles.vert" },
        { GL_FRAGMENT_SHADER, "media/shaders/triangles/triangles.frag" },
        { GL_NONE, NULL }
    };

    GLuint program = LoadShaders( shaders );    //着色器进入GPU
    glUseProgram( program );

    glVertexAttribPointer( vPosition, 2, GL_FLOAT,
                           GL_FALSE, 0, BUFFER_OFFSET(0) );
    glEnableVertexAttribArray( vPosition );
}

//----------------------------------------------------------------------------
//
// display：真正执行了渲染操作
//

void
display( void )
{
    static const float black[] = { 0.0f, 0.0f, 0.0f, 0.0f };

    glClearBufferfv(GL_COLOR, 0, black);    //清除窗口内容

    glBindVertexArray( VAOs[Triangles] );   //渲染对象
    glDrawArrays( GL_TRIANGLES, 0, NumVertices );   //输出最终图像
}

//----------------------------------------------------------------------------
//
// main
//

#ifdef _WIN32
int CALLBACK WinMain(
  _In_ HINSTANCE hInstance,
  _In_ HINSTANCE hPrevInstance,
  _In_ LPSTR     lpCmdLine,
  _In_ int       nCmdShow
)
#else
int
main( int argc, char** argv )
#endif
{
    glfwInit();

    GLFWwindow* window = glfwCreateWindow(800, 600, "Triangles", NULL, NULL);

    glfwMakeContextCurrent(window);
    gl3wInit();

    init();

    while (!glfwWindowShouldClose(window))
    {
        display();
        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    glfwDestroyWindow(window);

    glfwTerminate();
}
```
* init():负责设置程序中用到的数据，如顶点信息、纹理映射的图像数据。(本例中，init()指定了位置信息，又指定了程序中所用到的着色器。
* display():真正执行了渲染的工作，基本的三个步骤：调用glClearBufferfv()来清除窗口内容，调用OpenGL命令来渲染对象，将最终图形输出到屏幕。
* 第三方库GLFW和GL3W，使用它快速完成一些简单功能
## 1.3OpenGL语法
* 前缀：
  * gl库："gl"开头的函数；
  * glfw库："glfw"开头的函数；
  * gl3w库："gl3w"开头的函数；
* 数据类型：为了方便在不同的操作系统之间移植OpenGL程序：
* GLfloat：浮点数类型


    | 后缀        | 数据类型       | 通常对应C语言的数据类型 | 对应OpenGL类型  |
    | :-----------: | :-------------: | :---------------------: | :--------------: |
    | b           | 8位整型        | signed char           | GLbyte | 
    | s           | 16位整型       | signed short          | GLshort |
    | i           | 32位整型       | int                   | GLint/GLsizei |
    | f           | 32位浮点型     | float                 | GLfloat/GLclampf |
    | d           | 64位浮点型     | double               | GLdouble/GLclampd |
    | ub          | 8位无符号整型  | unsigned char | GLubyte |
    | us          | 16位无符号整型 | unsigned short | GLushort |
    | ui          | 32位无符号整型 | unsigned int | GLuint/GLenum/GLbitfield |


* C语言形式的库，不能使用函数的重载来处理不同类型的数据；
*  后缀：
   *  glUniform2f():"2"表示这个函数需要传入2个参数值，都是GLfloat类型的；
   *  glUniform3fv()："3"表示三个参数，f表示GLfloat，v表示vector，也就是需要一个GLfloat数组来传入2个浮点数值，而不是两个单独的参数值。

## 1.4OpenGL渲染管线
### OpenGL实现了渲染管线(rendering pipline)，也就是一系列数据处理过程，并将应用程序的数据转换到最终渲染的图像。
### 1.4.1 准备向OpenGL传输数据：将所需的数据都保存到缓存对象(buffer object)中，相当于OpenGL维护的一块内存区域。最常用办法就是使用 glNamedBufferStorage() 命令同时设置缓存的大小、内容。
### 1.4.2 将数据传输到 OpenGL: 缓存初始化完毕后，可以通过OpenGL的一个绘制命令来请求渲染几何图元。
### 1.4.3 顶点着色: 对于每个顶点，OpenGL都会调用一个顶点着色器来处理顶点相关的数据。通常来说，一个复杂的应用程序可能包含许多个顶点着色器，但是同一时刻只能有一个顶点着色器起作用。
### 1.4.4 细分着色：顶点着色器如果处理了每个定点相关联的数据后，若同时激活了细分着色器(tessellation shader)，那么它将进一步处理这些数据，细分着色器会使用面片(patch)来描述一个物体的形状，细分着色阶段会用到两个着色器来分别管理面片数据并生成最终的形状。
### 1.4.5 几何着色：几何着色允许光栅化之前，对每个几何图元做进一步的处理，例如创建新的图元（可选阶段）。
### 1.4.6 图元装配：前面着色阶段处理的都是顶点数据，图元装配阶段将这些顶点与相关的几何图元组织起来，准备下一步的剪切和光栅化工作。
### 1.4.7 剪切：顶点可能会落到视口(viewport)之外，此时与顶点相关的图元会作出改动，保证相关像素不会在视口之外绘制，这一过程叫做剪切(clipping)，他由OpenGL自动完成。
### 1.4.8 光栅化：剪切之后马上要执行的工作，将更新后的图元传递到光栅化(rasterize)单元，生成对应的片元。光栅化的工作是：判断某一部分的几何体(点、线、三角形)所覆盖的屏幕空间。得到了屏幕空间信息和输入的顶点数据后，光栅化单元可以直接对片元着色器中的每个可变变量进行线性插值，然后将结果值传递给用户的片元着色器。我们可以将一个片元视为一个"候选的像素"，也就是可以放置在帧缓存中的像素。之后的两个阶段将会执行片元的处理，即片元着色和逐片元操作。光栅化意味着一个片元的生命伊始，而片元着色器中的计算过程本质上决定了片元最终的颜色。
### 1.4.9 片元着色：编程控制屏幕上颜色显示的阶段。此阶段使用片元着色器，来计算片元的最终颜色和他的深度值(不考虑下一阶段微调)。这里采用的是纹理映射的方式，对顶点处理阶段所计算的颜色值进行补充。区别：顶点着色决定了一个图元应该位于屏幕的什么位置，而片元着色使用这些信息来决定某片元的颜色是什么样。
### 1.4.10 逐片元操作：最后的独立片元处理过程，这个阶段会使用深度测试(depth test，或称z缓存)，和模板测试(stencil test)的方式来决定一个片元是否是可见的。若一个片元通过了所有激活测试，那么他就可以被直接绘制到帧缓存中，它对应的像素颜色值(包括深度值)会被更新，如果开启了融混(blending)模式，那么片元的颜色会与该像素当前的颜色相叠加。像素数据来自图像文件，尽管他也可能是OpenGL直接渲染的。像素贴图通常保存在纹理贴图中，通过纹理映射的方式调用。
## 1.5 第一个程序：深入分析
### 1.5.1 进入main()函数
```C++
int
main( int argc, char** argv )
#endif
{
    glfwInit();

    GLFWwindow* window = glfwCreateWindow(800, 600, "Triangles", NULL, NULL);

    glfwMakeContextCurrent(window);
    gl3wInit();

    init();

    while (!glfwWindowShouldClose(window))
    {
        display();
        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    glfwDestroyWindow(window);

    glfwTerminate();
}
```
### 第一个函数glfwInit()负责初始化GLFW库，该函数必须是GLFW库被调用的第一个函数，他会负责设置其他GLFW例程所必需的数据结构。
### glfwCreateWindow()设置了程序所使用的窗口关联的OpenGL设备环境，在使用环境之前，必须把它设置为当前环境。
### 接下来调用gl3wInit()函数，属于GL3W库，该库可以简化获取函数地址的过程，并且包含了可以跨平台使用的其他一些OpenGL的编程方法。到这里完成了全部的设置工作。
### Init()负责初始化所有的OpenGL相关的所有数据。
### while无限循环判定，调用glfwWindowShouldClose();重绘它的内容，并展现给用户，调用glfwSwapBuffers();然后检查操作系统返回的任何信息，调用glfwPollEvents()。
### 需要关闭窗口，应用程序退出的话，会调用glfwDestroyWindow()来清理窗口，然后调用glfwTerminate()关闭GLFW库。
### 1.5.2 OpenGL的初始化过程
```C++
void
init( void )
{
    glGenVertexArrays( NumVAOs, VAOs );
    glBindVertexArray( VAOs[Triangles] );

    GLfloat  vertices[NumVertices][2] = {   //两个三角形的位置信息
        { -0.90f, -0.90f }, {  0.85f, -0.90f }, { -0.90f,  0.85f },  // Triangle 1
        {  0.90f, -0.85f }, {  0.90f,  0.90f }, { -0.85f,  0.90f }   // Triangle 2
    };

    glCreateBuffers( NumBuffers, Buffers );
    glBindBuffer( GL_ARRAY_BUFFER, Buffers[ArrayBuffer] );
    glBufferStorage( GL_ARRAY_BUFFER, sizeof(vertices), vertices, 0);

    ShaderInfo  shaders[] =
    {
        { GL_VERTEX_SHADER, "media/shaders/triangles/triangles.vert" },
        { GL_FRAGMENT_SHADER, "media/shaders/triangles/triangles.frag" },
        { GL_NONE, NULL }
    };

    GLuint program = LoadShaders( shaders );    //着色器进入GPU
    glUseProgram( program );

    glVertexAttribPointer( vPosition, 2, GL_FLOAT,
                           GL_FALSE, 0, BUFFER_OFFSET(0) );
    glEnableVertexAttribArray( vPosition );
}
```
### **<u>初始化顶点数组</u>**
### 调用glGenVertexArrays()分配顶点数组对象，这里共有NumVAOs个对象, 返回的是对象名的数组VAOs。
```
void glCreateVertexArrays(GLsizei n, GLuint *arrays);
    返回n个未使用的对象名到数组arrays中，用作顶点数组对象。返回的名字可以用来分配更多的缓存对象，并且他们已经使用未初始化的顶点数组集合的默认状态进行了数值的初始化。如果n是负数，产生GL_INVALID_VALUE错误。
``` 
### 很多OpenGL命令都是glCreate*的形式，他们负责分配不同类型的OpenGL对象名称，此名称类似于C语言的指针变量，我们可以分配内存对象并且用名称引用它。当我们得到对象之后，可以将他绑定到OpenGL环境以便使用。当我们绑定对象时，OpenGL内部会把它作为当前对象，及所有后续的操作都会对这个对象进行，例如，这里的顶点数组对象的状态就会被后面执行的代码所改变。
### 总体上来说，有两种情况会用到绑定：创建对象并初始化它所对应的数据时；每次我们准备使用这个对象，而这个对象并不是当前所绑定的对象时。
```
void glBindVertexArray(GLuint array);
    完成了两项工作，如果输入的变量array非0，并且是glCreateVertexArray()所返回的，那么会激活这个顶点数组对象，并且直接影响对象中所保存的顶点数组状态，如果是如的变量array为0，那么OpenGL将不再使用之前绑定的顶点数组。
    若array不是glCreateVertexArrays()所返回的数值，或者它已经被glDeleteVertexArrays()函数释放了，那么将产生GL_INVALID_OPERATION错误。
```
### 在较大的程序里，当我们完成对顶点数组对象的操作后，是可以调用glDeleteArrays()将他释放的。
```
void glDeleteVertexArrays(GLsizei n, const GLuint *arrays);
删除n个在arrays中定义的顶点数组对象，这样所有的名称可以再次用作顶点数组，当所有的顶点数组对象都被删除，那么当前绑定的数组对象被重设为0，并不再存在一个当前对象。
```
### 最后，为了确保程序的完整性，我们可以调用gllsVertexArrays()检查某个名称是否已经被保留为一个顶点数组对象了。
```
GLboolean gllsVertexArray(GLuint array);
    如果array是一个已经用glCreateArray()创建并且没有被删除的顶点数组对象的名称，返回true，否则，返回GL_FALSE。
```
### 对于OpenGL中其他类型的对象，我们都可以看到类似名为\*glDelete和\*glls的例程。
### **<u>分配缓存对象</u>**
### 顶点数组负责处理一系列的顶点数据，这些数据保存在缓存对象中。缓存对象的初始化与顶点数组对象的初始化类似，不过需要有向缓存中添加数据的一个过程。
```
void glGenBuffers(GLsizei n, GLuint *buffers);
void glBindBuffer(GLenum target, GLuint buffer);
void glDeleteBuffers(GLsizei n, const GLuint *buffers);
GLboolean glIsBuffer(GLuint buffer);
```
### **<u>将数据载入缓存对象</u>**
### 初始化顶点缓存对象后，我们需要让OpenGL分配缓存对象的空间，并把顶点数据从对象输出到缓存对象当中。通过glNamedBufferStorage()完成的，主要有两个任务：分配顶点数据所需的存储空间，然后将数据从应用程序的数组中拷贝到OpenGL服务端的内存中。glNamedBufferStorage()为一处缓存分配空间，并进行命名(缓存空间不需要被绑定)。
```
glNamedBufferStorage(GLuint buffer, GLsizeiptr size, const void *data, GLbitfield flags);
    在OpenGL服务端内存中分配size个存储单元(通常为byte)，用于数据存储或者索引。glNamedBufferStorage()作用于名为buffer的缓存区域。它不需要设置target参数。
    size表示存储数据的总量，为元素总数*单位元素存储空间的结果。
    data要么是一个客户端内存的指针，一边初始化缓存对象，要么是NULL。如果传入的指针合法，将会有size大小的数据从客户端拷贝到服务端，如果传入NULL，那么将保留size大小的未初始化数据，以备后用。
    flags提供了缓存中存储的数据相关的用途信息，他是一系列标识量经过逻辑“与”运算的总和。
    若所需的size大小超过了服务端能够分配的额度，那么glNamedBufferData()将产生GL_INVALID_VALUE错误。
```
### OpenGL只能够绘制坐标空间内的几何图元，而具有该范围限制的坐标系统，称为规格化设备坐标系统(Normalized Device Coordinate, NDC)。
### 现在成功创建了一个顶点数组，并将它传递到缓存对象中，下一步，要设置程序中用到的着色器了。
### **<u>初始化顶点与片元着色器</u>**
### 每个OpenGL程序进行绘制的时候，都至少准备两个着色器：顶点着色器和片元着色器。在这个例子中，我们通过辅助函数LoadShaders()来实现这个要求。它需要输入一个ShaderInfo结构体数组(实现可参见源代码头文件LoadShaders.h)。
### 对于OpenGL程序员而言，着色器就是使用OpenGL着色语言(OpenGL Shading Language, GLSL)编写的小型程序。它与C++语言非常类似，尽管GLSL中的所有特性不能用于OpenGL的每个着色阶段。我们可以以字符串的形式传输GLSL着色器到OpenGL。我们可以将着色器字符串的内容存储到文件中，使用LoadShaders()读取文件和创建OpenGL着色器程序。
```
#version 450 core   //指定了所用的OpenGL着色语言的版本 450表示OpenGL4.50对应的GLSL版本

layout( location = 0 ) in vec4 vPosition;

void
main()
{
    gl_Position = vPosition;
}

```
### 以上就是传递着色器，他只负责将输入数据拷贝到输出数据中。
### 每个着色器的第一行都应该设置"#version"，否则系统会假设使用"110"版本，而这与OpenGL核心模式并不兼容。
### 下一步，我们分配一个着色器变量，它是与外部世界的联系所在。
    * vPosition就是变量的名称，我们使用v作为顶点属性名的前缀，这个变量保存的是位置信息。
    * vec4 是vPositon的类型，在这里是一个GLSL的四维浮点数向量。默认填充为(0.0, 0.0, 0.0, 1.0)。因此当坐标仅仅设为(x, y)时，其余的两个坐标(z, w)被设为(0, 1)。
    * in字段指定了数据进入着色器的流向。
    * layout( location = 0 ), 叫做布局限定符(layout qualifier), 目的是为变量提供元数据(meta data)。我们可以使用布局限定符来设置很多不同的属性，其中有些是与不同的着色阶段相关的。
    * 在这里，vposition的位置属性location被设置为0，这个设置与init()函数的最后两行共同起作用。
    * 最后，在main()函数中实现他的主体部分。OpenGL的所有着色器，无论是处于哪个阶段，都会有一个main()函数。对于这个着色器而言，她所实现的就是将输入的顶点位置复制到顶点着色器的指定输出位置gl_Position中。后文我们将会了解到OpenGL所提供的一些着色器变量，都是以gl_作为前缀的。
### 与之类似，我们也需要一个片元着色器来配合顶点着色器的工作。
```
#version 450 core

out vec4 fColor;

void main()
{
    fColor = vec4(0.5, 0.4, 0.8, 1.0);
}

``` 
### 一样的是，我们还是需要声明版本号、变量以及main()函数。
    * 声明的变量名为fColor。它使用了out限定符，在这里，着色器会把fColor对应的数值输出，而这也就是片元所对应的颜色值(用到了前缀字符'f')。
    * 在输出fColor的声明之前也需要加上限定符layout(location = 0)。片元着色器可以设置多个输出值，而某个变量所对应的输出结果就是通过location来设置的。
    * 设定片元的颜色，OpenGL中的颜色是通过RGB颜色空间来表示的，其中每个颜色分量(R, G, B)的范围都是[0, 1]。但是这是个四位的向量————因为OpenGL实际上使用的是RGBA颜色空间，其中第四个值并不是颜色值，它叫做alpha值，专用于度量透明度。这里设置为1.0，表示片元颜色是完全不透明的。
### 我们已经基本完成了初始化过程，init()中最后的两个函数指定了顶点着色器的变量与我们存储在缓存对象中的数据的关系。这也就是我们所说的着色管线装配过，即：将应用程序与着色器之间，以及不同着色阶段之间的数据通道连接起来。
```
void glVertexAttribPointer(GLuint index, GLint size, GLenum type, GLboolean normalized, GLsizei stride, const GLvoid *pointer);

            设置顶点着色器属性indx位置对应的数据值。pointer表示缓存对象中，从起始位置开始计算的数组数据的偏移值(0开始)，使用基本的单位byte。size表示每个顶点需要更新的分量数目，可以是1、2、3、4或者GL_BGRA。type指定了数组中每个元素的数据类型(GL_BYTE、GL_SHORT等等)，normalized设置顶点数据在存储前是否要进行归一化，或者使用glVertexAttribFourN*()函数。stride数组中两个元素之间的大小的偏移值(byte)，若为零，数据应紧密的封装在一起。
```
### 在这里，我们还需要启用顶点属性数组，通过调用glEnableVertexAttributeArray()来完成这样工作。
```
void glEnableVertexAttribArray(GLuint index);
void glEnableVertexAttribArray(GLuint index);
    设置是否启用与index相关联的顶点数组，index必须是一个介于0到GL_MAX_VERTEX_ATTRIBS-1之间的距离。
```
### 绑定到设备环境中？/不绑定到设备环境中(直接状态访问)？
## 1.5.3 第一次使用OpenGL进行渲染
### 在设置和初始化所有数据之后，渲染的工作就非常简单了，display()只有四行代码，不过它包含的内容，在所有OpenGL中都会用到。
```C++
void 
display(void)
{
    static const float black[] = {0.0f, 0.0f, 0.0f, 0.0f};
    glClearBufferfv(GL_COLOR, 0, black);
    glBindVertexArray(VAOs[Triangles]);
    glBindVertexArray(GL_TRIANGLES, 0, NumVertices);
}
```
### 首先，我们要清除缓存的数据在进行渲染，清除的工作由glClearBufferfv()完成。
```
void glClearBufferfv(GLenum buffer, GLint drawbuffer, const GLfloat *value);
    清除当前绘制缓存中的指定缓存类型，清除结果为value。参数buffer设置了要清楚的缓存类型，它可以是GL_COLOR、GL_DEPTH(深度缓存)，或者GL_STENCIL(模板缓存)。参数drawbuffer设置了要清除的缓存索引。如果当前绑定的是默认帧缓存，或者buffer设置为GL_DEPTH或GL_STENCIL，那么drawbuffer必须是0。否则他表示需要被清除的颜色缓存的索引。
    参数value是一个数组的指针，其中包含了一个或者四个浮点数，用来设置清除缓存之后的颜色。如果buffer设置为GL_COLOR，那么value必须是一个最少四个数值的数组，以表示颜色值。如果buffer是GL_DEPTH或者GL_STENCIL，那么value可以是一个单独的浮点数，分别用来设置深度缓存或者模板缓存清除后的结果。
```
### **<u>使用OpenGL进行绘制</u>**
### 例子display()函数中，后面两行的工作是选择我们准备绘制的顶点数据，然后进行请求绘制。首先调用glBindVertexArray()来选择作为顶点数据使用的顶点数组。正如前文所说的，我们可以用这个函数来切换程序中保存的多个顶点数据对象的集合。
```
void glDrawArrays(GLenum mode, GLint first, GLsizei count);
    使用当前绑定的顶点数组元素来建立一系列的几何图元，起始位置为first，而结束位置为first + count - 1，mode设置了构建图元的类型，它可以是GL_POINTS、GL_LINES、GL_LINE_STRIP、GL_LINE_LOOP、GL_TRIANGLES、GL_TRIANGLE_STRIP、GL_TRIANGLE_FAN和GL_PATCHES中的任意一种。
```
### glDrawArrays()函数可以被认为是更复杂的glDrawArraysInstancedBaseInstance()函数的一个简化版本，后者包含了更多的参数。
### 在这个例子中，我们使用glVertexAttribPointer()设置渲染模式为GL_TRIANGLES，起始位置于缓存的0偏移位置，共渲染NumVertices个元素，这样就可以渲染出独立的三角形图元了。
### **<u>启用和禁用OpenGL的操作</u>**
    在第一个例子当中，有一个重要的特性并没有用到，但是在后文中我们会反复用到它，那就是对于OpenGL操作模式的启用和禁用。绝大多数命令都可由glEnable()和glDisable()命令开启或者关闭。
    ```
    void glEnable(GLenum capability);
    void glDisable(GLenum capability);
        glEnable()会开启一个模式，glDisable()会关闭它，有很多枚举量可以作为模式参数传入glEnable()和glDisable()。例如GL_DEPTH_TEST可以用来开启或者关闭深度测试；GL_BLEND可以用来控制融合的操作，而GL_RASTERIZER_DISCARD用于transform feedback过程中的高级渲染控制。
    ```
### 有时候，尤其是我们用OpenGL编写的库需要给其他程序员使用的时候，可以根据自己的需要来判断是否开启某个特性，这时候可以使用glIsEnable()来返回是否启用指定模式的信息。
```
GLboolean glIsEnbled(GLenum capability);
    根据使用启用当前指定的模式，返回GL_TRUE或者GL_FALSE。
```