# 你好三角

+ 在 `OpenGL` 中，一切都在 `3D` 空间中，但屏幕或窗口是 `2D` 像素数组，因此 `OpenGL` 的大部分工作是将所有 `3D` 坐标转换为适合屏幕的 `2D` 像素。将 `3D` 坐标转换为 `2D` 像素的过程由 `OpenGL` 的图形管道管理。图形管道可以分为两大部分：第一部分将 `3D` 坐标转换为 `2D` 坐标，第二部分将 `2D` 坐标转换为实际的彩色像素。在本章中，我们将简要讨论图形管道以及如何利用它来创建精美的像素。

+ 图形管道将一组 `3D` 坐标作为输入，并将它们转换为屏幕上的彩色 `2D` 像素。图形管道可以分为几个步骤，每个步骤都需要前一步的输出作为其输入。所有这些步骤都是高度专业化的（它们具有一个特定的功能）并且可以轻松地并行执行。由于其并行特性，当今的显卡具有数千个小型处理核心，可以快速处理图形管道中的数据。处理核心在 GPU 上为管道的每个步骤运行小程序。这些小程序称为着色器。

+ 其中一些着色器可由开发人员配置，这允许我们编写自己的着色器来替换现有的默认着色器。这使我们能够对管道的特定部分进行更细粒度的控制，并且由于它们在 `GPU` 上运行，因此还可以为我们节省宝贵的 `CPU` 时间。着色器是用 `OpenGL` 着色语言 ( `GLSL` ) 编写的，我们将在下一章中深入研究它。
  
+ 您将在下面找到图形管道所有阶段的抽象表示。请注意，蓝色部分代表我们可以注入自己的着色器的部分。

+ 正如您所看到的，图形管道包含大量部分，每个部分处理将顶点数据转换为完全渲染的像素的一个特定部分。我们将以简化的方式简要解释管道的每个部分，以便您更好地了解管道的运行方式。

+ 作为图形管道的输入，我们传入三个 `3D` 坐标的列表，这些坐标应在此处称为 `Vertex Data` 的数组中形成三角形；该顶点数据是顶点的集合。顶点是每个 `3D` 坐标的数据集合。该顶点的数据使用顶点属性来表示，顶点属性可以包含我们想要的任何数据，但为了简单起见，我们假设每个顶点仅包含一个 `3D` 位置和一些颜色值。

+ 为了让 `OpenGL` 知道如何处理坐标和颜色值的集合， `OpenGL` 要求您提示您想要使用数据形成哪种渲染类型。我们想要将数据渲染为点的集合、三角形的集合或者只是一条长线吗？这些提示称为基元，并在调用任何绘图命令时提供给 `OpenGL` 。其中一些提示是 `GL_POINTS` 、 `GL_TRIANGLES` 和 `GL_LINE_STRIP` 。

+ 管道的第一部分是顶点着色器，它将单个顶点作为输入。顶点着色器的主要目的是将 `3D` 坐标转换为不同的 `3D` 坐标（稍后详细介绍），顶点着色器允许我们对顶点属性进行一些基本处理。

+ 顶点着色器阶段的输出可以选择性地传递到几何着色器。几何着色器将形成图元的顶点集合作为输入，并且能够通过发出新顶点以形成新的（或其他）图元来生成其他形状。在此示例中，它会根据给定形状生成第二个三角形。

+ 图元组装阶段将来自顶点（或几何）着色器的所有顶点（或顶点，如果选择了 `GL_POINTS` ）作为输入，形成一个或多个图元，并组装给定图元形状中的所有点；在本例中是一个三角形。

+ 然后，几何着色器的输出被传递到光栅化阶段，在该阶段它将生成的图元映射到最终屏幕上的相应像素，从而产生供片段着色器使用的片段。在片段着色器运行之前，会执行裁剪。剪切会丢弃视图之外的所有片段，从而提高性能。

+ `OpenGL` 中的片段是 `OpenGL` 渲染单个像素所需的所有数据。

+ 片段着色器的主要目的是计算像素的最终颜色，这通常是所有高级 `OpenGL` 效果发生的阶段。通常，片段着色器包含有关 `3D` 场景的数据，可用于计算最终像素颜色（如灯光、阴影、灯光颜色等）。

+ 确定所有相应的颜色值后，最终对象将再经过一个阶段，我们称之为 `Alpha` 测试和混合阶段。此阶段检查片段的相应深度（和模板）值（我们稍后会介绍），并使用这些值来检查生成的片段是否位于其他对象的前面或后面，并相应地丢弃。该舞台还会检查 `Alpha` 值（ `Alpha` 值定义对象的不透明度）并相应地混合对象。因此，即使在片段着色器中计算像素输出颜色，在渲染多个三角形时，最终像素颜色仍然可能完全不同。

+ 正如您所看到的，图形管道是一个相当复杂的整体，包含许多可配置的部分。然而，对于几乎所有情况，我们只需要使用顶点和片段着色器。几何着色器是可选的，通常保留其默认着色器。还有我们没有在这里描述的曲面细分阶段和变换反馈循环，但这是稍后的内容。

+ 在现代 `OpenGL` 中，我们需要至少定义一个自己的顶点和片段着色器（ `GPU` 上没有默认的顶点/片段着色器）。因此，开始学习现代 `OpenGL` 通常非常困难，因为在渲染第一个三角形之前需要大量知识。一旦您在本章末尾最终渲染了三角形，您最终将了解更多有关图形编程的知识。

## 顶点输入

+ 要开始绘制一些东西，我们必须首先向 `OpenGL` 提供一些输入顶点数据。 `OpenGL` 是一个 `3D` 图形库，因此我们在 `OpenGL` 中指定的所有坐标都是 `3D` 坐标（ `x` 、 `y` 和 `z` 坐标）。 `OpenGL` 并不简单地将所有 `3D` 坐标转换为屏幕上的 `2D` 像素； `OpenGL` 仅在所有 `3` 个轴上位于 `-1.0` 和 `1.0` 之间的特定范围内时处理 `3D` 坐标（ `x` 、 `y` 和 `z` ）。这个所谓的标准化设备坐标范围内的所有坐标最终都将在屏幕上可见（而该区域之外的所有坐标则不会）。

+ 因为我们想要渲染单个三角形，所以我们想要指定总共三个顶点，每个顶点都有一个 `3D` 位置。我们在 `float` 数组中以标准化设备坐标（ `OpenGL` 的可见区域）定义它们：

    ```C++
    float vertices[] = {
        -0.5f, -0.5f, 0.0f,
        0.5f, -0.5f, 0.0f,
        0.0f,  0.5f, 0.0f
    };
    ```

+ 因为 `OpenGL` 在 `3D` 空间中工作，所以我们渲染一个 `2D` 三角形，每个顶点的 `z` 坐标为 `0.0` 。这样三角形的深度保持不变，使其看起来像二维的。

    + > 标准化设备坐标 ( `NDC` )
    
        * 在顶点着色器中处理顶点坐标后，它们应该处于标准化设备坐标中，这是一个小空间，其中 `x` 、 `y` 和 `z` 值从 `-1.0` 到 `1.0` 变化。任何超出此范围的坐标都将被丢弃/剪切，并且在屏幕上不可见。下面您可以看到我们在标准化设备坐标中指定的三角形（忽略 `z` 轴）：
        
        * 与通常的屏幕坐标不同，正 `y` 轴指向向上方向，并且 `(0,0)` 坐标位于图表的中心，而不是左上角。最终您希望所有（转换后的）坐标都位于该坐标空间中，否则它们将不可见。
        
        * 然后，您的 `NDC` 坐标将使用您通过 `glViewport` 提供的数据通过视口变换转换为屏幕空间坐标。然后，生成的屏幕空间坐标将转换为片段，作为片段着色器的输入。

+ 定义顶点数据后，我们希望将其作为输入发送到图形管道的第一个进程：顶点着色器。这是通过在 GPU 上创建存储顶点数据的内存、配置 `OpenGL` 应如何解释内存并指定如何将数据发送到显卡来完成的。然后，顶点着色器从内存中处理我们指定数量的顶点。

+ 我们通过所谓的顶点缓冲区对象  `(VBO)` 来管理此内存，该对象可以在 `GPU` 内存中存储大量顶点。使用这些缓冲区对象的优点是，我们可以一次向显卡发送大量数据，并在有足够内存的情况下将其保留在那里，而不必一次向一个顶点发送数据。从 `CPU` 向显卡发送数据的速度相对较慢，因此只要有可能，我们就尽量一次发送尽可能多的数据。一旦数据进入显卡内存，顶点着色器几乎可以立即访问顶点，使其速度极快。

+ 正如我们在 `OpenGL` 章节中讨论的那样，顶点缓冲区对象是我们第一次出现的 `OpenGL` 对象。就像 `OpenGL` 中的任何对象一样，该缓冲区具有与该缓冲区对应的唯一 `ID` ，因此我们可以使用 `glGenBuffers` 函数生成一个具有缓冲区 `ID` 的缓冲区：

    ```C++
    unsigned int VBO;
    glGenBuffers(1, &VBO)
    ```

+ `OpenGL` 有多种类型的缓冲对象，顶点缓冲对象的缓冲类型是 `GL_ARRAY_BUFFER` 。 `OpenGL` 允许我们同时绑定到多个缓冲区，只要它们具有不同的缓冲区类型。我们可以使用 `glBindBuffer` 函数将新创建的缓冲区绑定到 `GL_ARRAY_BUFFER` 目标：

    ```C++
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    ```

+ 从那时起，我们（在 `GL_ARRAY_BUFFER` 目标上）进行的任何缓冲区调用都将用于配置当前绑定的缓冲区，即 `VBO` 。然后我们可以调用 `glBufferData` 函数将之前定义的顶点数据复制到缓冲区的内存中：

    ```C++
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    ```

+ `glBufferData` 是一个专门用于将用户定义的数据复制到当前绑定的缓冲区中的函数。它的第一个参数是我们要将数据复制到的缓冲区的类型：当前绑定到 `GL_ARRAY_BUFFER` 目标的顶点缓冲区对象。第二个参数指定我们要传递到缓冲区的数据大小（以字节为单位）；简单的 `sizeof` 顶点数据就足够了。第三个参数是我们要发送的实际数据。

+ 第四个参数指定我们希望显卡如何管理给定的数据。这可以采取 3 种形式：

    * `GL_STREAM_DRAW` ：数据仅设置一次，最多被 `GPU` 使用几次。

    * `GL_STATIC_DRAW` ：数据仅设置一次并使用多次。

    * `GL_DYNAMIC_DRAW` ：数据变化较大，使用多次。

+ 三角形的位置数据不会改变，被大量使用，并且对于每次渲染调用都保持不变，因此其使用类型最好是 `GL_STATIC_DRAW` 。例如，如果缓冲区中的数据可能会频繁更改，则 `GL_DYNAMIC_DRAW` 使用类型可确保显卡将数据放置在内存中，从而实现更快的写入速度。

+ 到目前为止，我们将顶点数据存储在显卡内存中，由名为 `VBO` 的顶点缓冲区对象管理。接下来我们想要创建一个实际处理这些数据的顶点和片段着色器，所以让我们开始构建它们。

## 顶点着色器

+ 顶点着色器是像我们这样的人可以编程的着色器之一。现代 `OpenGL` 要求如果我们想要进行一些渲染，我们至少需要设置一个顶点和片段着色器，因此我们将简要介绍着色器并配置两个非常简单的着色器来绘制我们的第一个三角形。在下一章中，我们将更详细地讨论着色器。

+ 我们需要做的第一件事是用着色器语言 `GLSL` （ `OpenGL` 着色语言）编写顶点着色器，然后编译该着色器，以便我们可以在应用程序中使用它。您将在下面找到 `GLSL` 中非常基本的顶点着色器的源代码：

    ```C++
    #version 330 core
    layout (location = 0) in vec3 aPos;

    void main()
    {
        gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
    }
    ```

+ 正如您所看到的， `GLSL` 看起来与 `C` 类似。每个着色器都以对其版本的声明开头。由于 `OpenGL 3.3` 及更高版本， `GLSL` 的版本号与 `OpenGL` 的版本匹配（例如， `GLSL` 版本 `420` 对应于 `OpenGL` 版本 `4.2` ）。我们还明确提到我们正在使用核心配置文件功能。

+ 接下来，我们使用 `in` 关键字声明顶点着色器中的所有输入顶点属性。现在我们只关心位置数据，所以我们只需要一个顶点属性。 `GLSL` 有一个矢量数据类型，根据其后缀数字包含 `1` 到 `4` 个浮点数。由于每个顶点都有一个 `3D` 坐标，我们创建一个名为 `aPos` 的 `vec3` 输入变量。我们还通过 `layout (location = 0)` 专门设置了输入变量的位置，稍后您将了解为什么我们需要该位置。

    + > 向量

        * 在图形编程中，我们经常使用向量的数学概念，因为它巧妙地表示任何空间中的位置/方向，并且具有有用的数学属性。 `GLSL` 中的向量的最大大小为 `4` ，其每个值都可以通过 `vec.x` 、 `vec.y` 、 `vec.z` 和 `vec.w` 检索分别，其中每个代表空间中的一个坐标。请注意， `vec.w` 组件不用作空间中的位置（我们处理的是 `3D` ，而不是 `4D` ），而是用于称为透视划分的东西。我们将在后面的章节中更深入地讨论向量。

+ 要设置顶点着色器的输出，我们必须将位置数据分配给预定义的 `gl_Position` 变量，该变量是幕后的 `vec4` 。在主函数的末尾，无论我们设置 `gl_Position` 是什么，都将用作顶点着色器的输出。由于我们的输入是大小为 `3` 的向量，因此我们必须将其转换为大小为 `4` 的向量。我们可以通过在 `vec4` 的构造函数中插入 `vec3` 值并设置其 `w` 组件到 `1.0f` （我们将在后面的章节中解释原因）。

+ 当前的顶点着色器可能是我们能想象到的最简单的顶点着色器，因为我们没有对输入数据进行任何处理，只是将其转发到着色器的输出。在实际应用中，输入数据通常尚未处于标准化设备坐标中，因此我们首先必须将输入数据转换为落在 `OpenGL` 可见区域内的坐标。

## 编译着色器

+ 我们暂时将顶点着色器的源代码存储在代码文件顶部的 `const C` 字符串中：

    ```C++
    const char *vertexShaderSource = "#version 330 core\n"
        "layout (location = 0) in vec3 aPos;\n"
        "void main()\n"
        "{\n"
        "   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
        "}\0";
    ```

+ 为了让 `OpenGL` 使用着色器，它必须在运行时从源代码动态编译它。我们需要做的第一件事是创建一个着色器对象，同样由 `ID` 引用。因此，我们将顶点着色器存储为 `unsigned int` 并使用 `glCreateShader` 创建着色器：

    ```C++
    unsigned int vertexShader;
    vertexShader = glCreateShader(GL_VERTEX_SHADER);
    ```

+ 我们提供要创建的着色器类型作为 `glCreateShader` 的参数。由于我们正在创建一个顶点着色器，因此我们传入 `GL_VERTEX_SHADER` 。

+ 接下来我们将着色器源代码附加到着色器对象并编译着色器：

    ```C++
    glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
    glCompileShader(vertexShader);
    ```

+ `glShaderSource` 函数将要编译的着色器对象作为其第一个参数。第二个参数指定我们作为源代码传递的字符串数量，这只是一个。第三个参数是顶点着色器的实际源代码，我们可以将第四个参数保留为 `NULL` 。

    * 您可能想在调用 `glCompileShader` 后检查编译是否成功，如果没有，则发现了哪些错误，以便您可以修复这些错误。检查编译时错误的方法如下：

        ```C++
        int  success;
        char infoLog[512];
        glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
        ```
    
    * 首先，我们定义一个整数来指示成功，并定义一个用于存储错误消息（如果有）的容器。然后我们用 `glGetShaderiv` 检查编译是否成功。如果编译失败，我们应该使用 `glGetShaderInfoLog` 检索错误消息并打印错误消息。

        ```C++
        if(!success)
        {
            glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
            std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
        }
        ```

+ 如果编译顶点着色器时未检测到错误，则现在将对其进行编译。

## 片段着色器

+ 片段着色器是我们要为渲染三角形而创建的第二个也是最后一个着色器。片段着色器主要用于计算像素的颜色输出。为了简单起见，片段着色器将始终输出橙色。

    * 计算机图形学中的颜色由 `4` 个值组成的数组表示：红色、绿色、蓝色和 `alpha` （不透明度）分量，通常缩写为 `RGBA` 。在 `OpenGL` 或 `GLSL` 中定义颜色时，我们将每个分量的强度设置为 `0.0` 和 `1.0` 之间的值。例如，如果我们将红色设置为 `1.0` ，将绿色设置为 `1.0` ，我们将得到两种颜色的混合并得到黄色。给定这 `3` 个颜色分量，我们可以生成超过 `1600` 万种不同的颜色！

    ```C++
    #version 330 core
    out vec4 FragColor;

    void main()
    {
        FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
    }
    ```

+ 片段着色器只需要一个输出变量，即一个大小为 `4` 的向量，它定义了我们应该自己计算的最终颜色输出。我们可以使用 `out` 关键字声明输出值，我们在这里将其命名为 `FragColor` 。接下来，我们简单地将 `vec4` 分配给颜色输出，作为橙色， `alpha` 值为 `1.0` （ `1.0` 完全不透明）。

+ 编译片段着色器的过程与顶点着色器类似，尽管这次我们使用 `GL_FRAGMENT_SHADER` 常量作为着色器类型：

    ```C++
    unsigned int fragmentShader;
    fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
    glCompileShader(fragmentShader);
    ```

+ 现在，两个着色器都已编译完毕，唯一剩下要做的就是将两个着色器对象链接到可用于渲染的着色器程序中。请务必在此处检查编译错误！

## 着色器程序

+ 着色器程序对象是多个着色器组合的最终链接版本。要使用最近编译的着色器，我们必须将它们链接到着色器程序对象，然后在渲染对象时激活该着色器程序。当我们发出渲染调用时，将使用激活的着色器程序的着色器。

+ 将着色器链接到程序时，它将每个着色器的输出链接到下一个着色器的输入。如果您的输出和输入不匹配，这也是您会收到链接错误的地方。

+ 创建程序对象很简单：

    ```C++
    unsigned int shaderProgram;
    shaderProgram = glCreateProgram();
    ```

+ `glCreateProgram` 函数创建一个程序并返回新创建的程序对象的 `ID` 引用。现在我们需要将之前编译的着色器附加到程序对象，然后使用 `glLinkProgram` 将它们链接起来：

    ```C++
    glAttachShader(shaderProgram, vertexShader);
    glAttachShader(shaderProgram, fragmentShader);
    glLinkProgram(shaderProgram);
    ```

+ 代码应该非常不言自明，我们将着色器附加到程序并通过 `glLinkProgram` 链接它们。

    * 就像着色器编译一样，我们也可以检查链接着色器程序是否失败并检索相应的日志。但是，我们现在使用的不是 glGetShaderiv 和 glGetShaderInfoLog：

        ```C++
        glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
        if(!success) {
            glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
            ...
        }
        ```

+ 结果是一个程序对象，我们可以通过调用 `glUseProgram` 以新创建的程序对象作为其参数来激活它：

    ```C++
    glUseProgram(shaderProgram);
    ```

+ `glUseProgram` 之后的每个着色器和渲染调用现在都将使用此程序对象（以及着色器）。

+ 哦，是的，一旦我们将着色器对象链接到程序对象中，不要忘记删除它们；我们不再需要它们了：

    ```C++
    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);  
    ```

+ 现在，我们将输入顶点数据发送到 GPU，并指示 GPU 如何在顶点和片段着色器中处理顶点数据。我们已经快到了，但还没有完全实现。 `OpenGL` 还不知道如何解释内存中的顶点数据以及如何将顶点数据连接到顶点着色器的属性。我们会很乐意告诉 `OpenGL` 如何做到这一点。

## 链接顶点属性

+ 顶点着色器允许我们以顶点属性的形式指定我们想要的任何输入，虽然这提供了很大的灵活性，但这确实意味着我们必须手动指定输入数据的哪一部分进入顶点着色器中的哪个顶点属性。这意味着我们必须在渲染之前指定 `OpenGL` 如何解释顶点数据。

+ 我们的顶点缓冲区数据的格式如下：

    * 位置数据存储为 32 位（4 字节）浮点值。

    * 每个位置由其中 3 个值组成。

    * 每组 3 个值之间没有空格（或其他值）。这些值紧密地封装在数组中。

    * 数据中的第一个值位于缓冲区的开头。

+ 有了这些知识，我们就可以告诉 `OpenGL` 如何使用 `glVertexAttribPointer` 解释顶点数据（每个顶点属性）：

    ```C++
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0); 
    ```

+ 函数 `glVertexAttribPointer` 有相当多的参数，所以让我们仔细看看它们：

    * 第一个参数指定我们要配置的顶点属性。请记住，我们使用 `layout (location = 0)` 指定了顶点着色器中位置顶点属性的位置。这将顶点属性的位置设置为 `0` ，并且由于我们想要将数据传递给该顶点属性，因此我们传入 `0` 。

    * 下一个参数指定顶点属性的大小。顶点属性是 `vec3` ，因此它由 `3` 值组成。

    * 第三个参数指定数据类型 `GL_FLOAT` （ `GLSL` 中的 `vec*` 由浮点值组成）。

    * 下一个参数指定我们是否希望数据标准化。如果我们输入整数数据类型（ `int` 、 `byte` ）并将其设置为 `GL_TRUE` ，则整数数据将标准化为 `0` （或对于有符号数据为 `-1` ）和 `1` 转换为浮点数时。这与我们无关，因此我们将其保留为 `GL_FALSE` 。

    * 第五个参数称为步幅，告诉我们连续顶点属性之间的空间。由于下一组位置数据正好位于 `float` 距离大小的 `3` 倍处，我们将该值指定为步幅。请注意，由于我们知道数组是紧密排列的（下一个顶点属性值之间没有空格），因此我们还可以将步幅指定为 `0` 以让 `OpenGL` 确定步幅（这仅在以下情况下有效）：值紧密排列）。每当我们有更多顶点属性时，我们都必须仔细定义每个顶点属性之间的间距，但稍后我们会看到更多这样的示例。
    
    > 每个顶点属性从 `VBO` 管理的内存中获取数据，并且从哪个 `VBO` （可以有多个 `VBO` ）获取数据由调用 `glVertexAttribPointer` 时当前绑定到 `GL_ARRAY_BUFFER` 的 `VBO` 确定。由于先前定义的 `VBO` 在调用 `glVertexAttribPointer` 之前仍处于绑定状态，因此顶点属性 `0` 现在与其顶点数据关联。

+ 现在我们指定了 `OpenGL` 应如何解释顶点数据，我们还应该使用 `glEnableVertexAttribArray` 启用顶点属性，将顶点属性位置作为其参数；默认情况下禁用顶点属性。从那时起，我们就完成了所有设置：我们使用顶点缓冲区对象初始化缓冲区中的顶点数据，设置顶点和片段着色器，并告诉 `OpenGL` 如何将顶点数据链接到顶点着色器的顶点属性。在 `OpenGL` 中绘制对象现在看起来像这样：

    ```C++
    // 0. copy our vertices array in a buffer for OpenGL to use
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    // 1. then set the vertex attributes pointers
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);  
    // 2. use our shader program when we want to render an object
    glUseProgram(shaderProgram);
    // 3. now draw the object 
    someOpenGLFunctionThatDrawsOurTriangle(); 
    ```

+ 每次我们想要绘制一个对象时，我们都必须重复这个过程。它可能看起来没有那么多，但想象一下，如果我们有超过 `5` 个顶点属性，并且可能有 `100` 个不同的对象（这并不罕见）。绑定适当的缓冲区对象并为每个对象配置所有顶点属性很快就会成为一个繁琐的过程。如果有某种方法我们可以将所有这些状态配置存储到一个对象中并简单地绑定该对象以恢复其状态怎么办？

## 顶点数组对象

+ 顶点数组对象（也称为 `VAO` ）可以像顶点缓冲区对象一样进行绑定，并且从该点开始的任何后续顶点属性调用都将存储在 `VAO` 内。这样做的好处是，在配置顶点属性指针时，您只需调用一次，并且每当我们想要绘制对象时，我们只需绑定相应的 `VAO` 即可。这使得不同顶点数据和属性配置之间的切换就像绑定不同的 `VAO` 一样简单。我们刚刚设置的所有状态都存储在 `VAO` 中。

    > 核心 `OpenGL` 要求我们使用 `VAO` ，以便它知道如何处理我们的顶点输入。如果我们无法绑定 `VAO` ， `OpenGL` 很可能会拒绝绘制任何内容。

+ 顶点数组对象存储以下内容：

    * 调用 `glEnableVertexAttribArray` 或 `glDisableVertexAttribArray` 。

    * 通过 `glVertexAttribPointer` 进行顶点属性配置。

    * 通过调用 `glVertexAttribPointer` 与顶点属性关联的顶点缓冲区对象。

+ 生成 `VAO` 的过程与 `VBO` 类似：

    ```C++
    unsigned int VAO;
    glGenVertexArrays(1, &VAO);
    ```

+ 要使用 `VAO` ，您只需使用 `glBindVertexArray` 绑定 `VAO` 。从那时起，我们应该绑定/配置相应的 `VBO` 和属性指针，然后取消绑定 `VAO` 以供以后使用。一旦我们想要绘制一个对象，我们只需在绘制对象之前将 `VAO` 与首选设置绑定即可。在代码中，这看起来有点像这样：

    ```C++
    // ..:: Initialization code (done once (unless your object frequently changes)) :: ..
    // 1. bind Vertex Array Object
    glBindVertexArray(VAO);
    // 2. copy our vertices array in a buffer for OpenGL to use
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    // 3. then set our vertex attributes pointers
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);  

    
    [...]

    // ..:: Drawing code (in render loop) :: ..
    // 4. draw the object
    glUseProgram(shaderProgram);
    glBindVertexArray(VAO);
    someOpenGLFunctionThatDrawsOurTriangle();
    ```

+ 就是这样！我们在过去几百万页中所做的一切都导致了这一刻，一个 VAO 存储了我们的顶点属性配置以及要使用的 `VBO` 。通常，当您想要绘制多个对象时，您首先生成/配置所有 `VAO` （以及所需的 `VBO` 和属性指针）并存储它们以供以后使用。当我们想要绘制一个对象时，我们获取相应的 `VAO` ，绑定它，然后绘制该对象并再次取消绑定 `VAO` 。

## 我们一直在等待的三角形

+ 为了绘制我们选择的对象， `OpenGL` 为我们提供了 `glDrawArrays` 函数，该函数使用当前活动的着色器、先前定义的顶点属性配置和 `VBO` 的顶点数据（通过 `VAO` 间接绑定）来绘制图元。

    ```C++
    glUseProgram(shaderProgram);
    glBindVertexArray(VAO);
    glDrawArrays(GL_TRIANGLES, 0, 3);
    ```

+ `glDrawArrays` 函数将我们想要绘制的 `OpenGL` 基元类型作为其第一个参数。因为我一开始就说过我们要画一个三角形，而且我不喜欢骗你，所以我们传入 `GL_TRIANGLES` 。第二个参数指定我们要绘制的顶点数组的起始索引；我们将其保留在 `0` 处。最后一个参数指定我们想要绘制多少个顶点，即 `3` （我们只从数据中渲染 `1` 个三角形，正好是 `3` 个顶点长）。

+ 现在尝试编译代码，如果出现任何错误，请向后进行。应用程序编译后，您应该会看到以下结果：

+ 完整程序的源代码可以在这里找到。

+ 如果你的输出看起来不一样，你可能在这个过程中做错了什么，所以检查完整的源代码，看看你是否错过了任何东西。

## 元素缓冲区对象

+ 渲染顶点时我们要讨论的最后一件事是元素缓冲区对象，缩写为 `EBO` 。为了解释元素缓冲区对象如何工作，最好举一个例子：假设我们想绘制一个矩形而不是三角形。我们可以用两个三角形来绘制一个矩形（ `OpenGL` 主要使用三角形）。这将生成以下顶点集：

    ```C++
    float vertices[] = {
        // first triangle
        0.5f,  0.5f, 0.0f,  // top right
        0.5f, -0.5f, 0.0f,  // bottom right
        -0.5f,  0.5f, 0.0f,  // top left 
        // second triangle
        0.5f, -0.5f, 0.0f,  // bottom right
        -0.5f, -0.5f, 0.0f,  // bottom left
        -0.5f,  0.5f, 0.0f   // top left
    }; 
    ```

+ 正如您所看到的，指定的顶点有一些重叠。我们指定 `bottom` `right` 和 `top` `left` 两次！这是 50% 的开销，因为同一个矩形也可以只指定 `4` 个顶点，而不是 `6` 个。一旦我们有更复杂的模型，其中有超过 `1000` 个三角形，其中会有大块，这种情况只会变得更糟。重叠。更好的解决方案是只存储唯一的顶点，然后指定绘制这些顶点的顺序。在这种情况下，我们只需为矩形存储 `4` 个顶点，然后指定绘制的顺序我们想画它们。如果 `OpenGL` 为我们提供这样的功能岂不是很棒？

+ 值得庆幸的是，元素缓冲区对象的工作原理与此完全相同。 `EBO` 是一个缓冲区，就像顶点缓冲区对象一样，它存储 `OpenGL` 用来决定绘制哪些顶点的索引。这个所谓的索引绘图正是我们问题的解决方案。首先，我们必须指定（唯一的）顶点和索引以将它们绘制为矩形：

    ```C++
    float vertices[] = {
        0.5f,  0.5f, 0.0f,  // top right
        0.5f, -0.5f, 0.0f,  // bottom right
        -0.5f, -0.5f, 0.0f,  // bottom left
        -0.5f,  0.5f, 0.0f   // top left 
    };
    unsigned int indices[] = {  // note that we start from 0!
        0, 1, 3,   // first triangle
        1, 2, 3    // second triangle
    }; 
    ```

+ 您可以看到，使用索引时，我们只需要 `4` 个顶点，而不是 `6` 个。接下来我们需要创建元素缓冲区对象：

    ```C++
    unsigned int EBO;
    glGenBuffers(1, &EBO);
    ```

+ 与 `VBO` 类似，我们绑定 `EBO` 并使用 `glBufferData` 将索引复制到缓冲区中。另外，就像 `VBO` 一样，我们希望将这些调用放在绑定和取消绑定调用之间，尽管这次我们指定 `GL_ELEMENT_ARRAY_BUFFER` 作为缓冲区类型。

    ```C++
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW); 
    ```

+ 请注意，我们现在将 `GL_ELEMENT_ARRAY_BUFFER` 作为缓冲区目标。最后要做的事情是用 `glDrawElements` 替换 `glDrawArrays` 调用，以指示我们要从索引缓冲区渲染三角形。当使用 `glDrawElements` 时，我们将使用当前绑定的元素缓冲区对象中提供的索引进行绘制：

    ```C++
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
    glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
    ```

+ 第一个参数指定我们想要绘制的模式，类似于 `glDrawArrays` 。第二个参数是我们想要绘制的元素的数量。我们指定了 `6` 个索引，因此我们总共要绘制 `6` 个顶点。第三个参数是索引的类型，其类型为 `GL_UNSIGNED_INT` 。最后一个参数允许我们指定 `EBO` 中的偏移量（或传入索引数组，但这是当您不使用元素缓冲区对象时），但我们只需将其保留为 `0` 。

+ `glDrawElements` 函数从当前绑定到 `GL_ELEMENT_ARRAY_BUFFER` 目标的 `EBO` 中获取其索引。这意味着每次我们想要用索引渲染一个对象时，我们都必须绑定相应的 `EBO` ，这又有点麻烦。碰巧顶点数组对象也跟踪元素缓冲区对象绑定。绑定 `VAO` 时绑定的最后一个元素缓冲区对象将存储为 `VAO` 的元素缓冲区对象。绑定到 `VAO` 后也会自动绑定该 `EBO` 。

    > 当目标是 `GL_ELEMENT_ARRAY_BUFFER` 时， `VAO` 存储 `glBindBuffer` 调用。这也意味着它存储其取消绑定调用，因此请确保在取消绑定   `VAO` 之前不要取消绑定元素数组缓冲区，否则它不会配置 `EBO` 。

+ 生成的初始化和绘图代码现在看起来像这样：

    ```C++
    // ..:: Initialization code :: ..
    // 1. bind Vertex Array Object
    glBindVertexArray(VAO);
    // 2. copy our vertices array in a vertex buffer for OpenGL to use
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    // 3. copy our index array in a element buffer for OpenGL to use
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
    // 4. then set the vertex attributes pointers
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);  

    [...]
    
    // ..:: Drawing code (in render loop) :: ..
    glUseProgram(shaderProgram);
    glBindVertexArray(VAO);
    glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
    glBindVertexArray(0);
    ```

+ 运行程序应该给出如下所示的图像。左图看起来应该很熟悉，右图是在线框模式下绘制的矩形。线框矩形显示该矩形确实由两个三角形组成。

    + > 线框模式

        > 要在线框模式下绘制三角形，您可以通过 `glPolygonMode(GL_FRONT_AND_BACK, GL_LINE)` 配置 `OpenGL` 如何绘制其图元。第一个参数表示我们要将其应用于所有三角形的正面和背面，第二行告诉我们将它们绘制为线。任何后续的绘图调用都将以线框模式渲染三角形，直到我们使用 `glPolygonMode(GL_FRONT_AND_BACK, GL_FILL)` 将其设置回默认值。

+ 如果有任何错误，请向后查看，看看是否遗漏了任何内容。您可以在这里找到完整的源代码。

+ 如果您像我们一样成功地绘制了一个三角形或一个矩形，那么恭喜您，您成功地通过了现代 `OpenGL` 最难的部分之一：绘制您的第一个三角形。这是一个困难的部分，因为在绘制第一个三角形之前需要大量的知识。值得庆幸的是，我们现在已经跨越了这个障碍，接下来的章节有望更容易理解。

## 其他资源

+ `antongerdelan.net/hellotriangle` ：`Anton Gerdelan` 对渲染第一个三角形的看法。

+ `open.gl/drawing` ： `Alexander Overvoorde` 对渲染第一个三角形的看法。]

+ `antongerdelan.net/vertexbuffers` ：对顶点缓冲区对象的一些额外见解。

+ `learnopengl.com/In-Practice/Debugging` ：本章涉及的步骤很多；如果您遇到困难，可能值得阅读一些有关 `OpenGL` 调试的内容（直到调试输出部分）。

## 练习

+ 为了真正很好地掌握所讨论的概念，我们设置了一些练习。建议在继续下一主题之前先完成这些内容，以确保您充分掌握正在发生的事情。

    * 1.尝试通过向数据添加更多顶点来使用 `glDrawArrays` 绘制两个相邻的三角形：解决方案。

    * 2.现在使用两个不同的 `VAO` 和 `VBO` 作为数据创建相同的 `2` 个三角形：解决方案。

    * 创建两个着色器程序，其中第二个程序使用输出黄色的不同片段着色器；再次绘制两个三角形，其中一个输出颜色为黄色：解决方案。