# 你好窗口

+ 让我们看看是否可以启动并运行 `GLFW` 。首先，创建一个 .cpp 文件并将以下包含内容添加到新创建的文件的顶部。

    ```C++
    #include <glad/glad.h>
    #include <GLFW/glfw3.h>
    ```

+ 请务必在 `GLFW` 之前包含 `GLAD` 。 `GLAD` 的包含文件在后台包含所需的 `OpenGL` 标头（如 `GL/gl.h` ），因此请务必在需要 `OpenGL` 的其他标头文件（如 `GLFW` ）之前包含 `GLAD` 。

+ 接下来，我们创建主函数，在其中实例化 `GLFW` 窗口：

    ```C++
    int main()
    {
        glfwInit();
        glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
        glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
        glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
        //glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
    
        return 0;
    }
    ```

+ 在 `main` 函数中，我们首先使用 `glfwInit` 初始化 `GLFW` ，之后我们可以使用 `glfwWindowHint` 配置 `GLFW` 。 `glfwWindowHint` 的第一个参数告诉我们要配置什么选项，我们可以从以 `GLFW_` 为前缀的大量可能选项中选择该选项。第二个参数是一个整数，用于设置选项的值。所有可能选项及其相应值的列表可以在 `GLFW` 的窗口处理文档中找到。如果您现在尝试运行该应用程序，并且它给出了许多未定义的引用错误，则意味着您没有成功链接 `GLFW` 库。

+ 确保您的系统/硬件上安装了 `OpenGL 3.3` 或更高版本，否则应用程序将崩溃或显示未定义的行为。要查找计算机上的 `OpenGL` 版本，请在 `Linux` 计算机上调用 `glxinfo` 或使用适用于 `Windows` 的 `OpenGL Extension Viewer` 等实用程序。如果您支持的版本较低，请尝试检查您的显卡是否支持 `OpenGL 3.3+`（否则它真的很旧）和/或更新您的驱动程序。

+ 接下来我们需要创建一个窗口对象。该窗口对象保存所有窗口数据，并且是 `GLFW` 的大多数其他函数所需要的。

    ```C++
    GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
    if (window == NULL)
    {
        std::cout << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }
    glfwMakeContextCurrent(window);
    ```

+ `glfwCreateWindow` 函数需要窗口宽度和高度分别作为其前两个参数。第三个参数允许我们为窗口创建一个名称；现在我们将其称为 "LearnOpenGL" ，但您可以随意命名。我们可以忽略最后两个参数。该函数返回一个 `GLFWwindow` 对象，我们稍后将需要该对象来执行其他 `GLFW` 操作。之后，我们告诉 `GLFW` 将窗口上下文设置为当前线程的主上下文。

## GLAD

+ 在上一章中我们提到 `GLAD` 管理 `OpenGL` 的函数指针，因此我们希望在调用任何 `OpenGL` 函数之前初始化 `GLAD` ：

    ```C++
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
    {
        std::cout << "Failed to initialize GLAD" << std::endl;
        return -1;
    }
    ```

+ 我们向 `GLAD` 传递函数来加载特定于操作系统的 `OpenGL` 函数指针的地址。 `GLFW` 为我们提供了 `glfwGetProcAddress` ，它根据我们正在编译的操作系统定义了正确的函数。

## 视口

+ 在开始渲染之前，我们必须做最后一件事。我们必须告诉 `OpenGL` 渲染窗口的大小，以便 `OpenGL` 知道我们想要如何显示数据和相对于窗口的坐标。我们可以通过 `glViewport` 函数设置这些尺寸：

    ```C++
    glViewport(0, 0, 800, 600);
    ```

+ `glViewport` 的前两个参数设置窗口左下角的位置。第三个和第四个参数设置渲染窗口的宽度和高度（以像素为单位），我们将其设置为等于 `GLFW` 的窗口大小。

+ 在幕后， `OpenGL` 使用通过 `glViewport` 指定的数据将其处理的 `2D` 坐标转换为屏幕上的坐标。例如，处理后的位置 `(-0.5,0.5)` 点将（作为其最终变换）映射到屏幕坐标中的 `(200,450)` 。请注意， `OpenGL` 中处理的坐标介于 `-1` 和 `1` 之间，因此我们有效地从范围 `(-1 到 1)` 映射到 `(0, 800)` 和 `(0, 600)` 。

+ 然而，当用户调整窗口大小时，视口也应该调整。我们可以在窗口上注册一个回调函数，每次调整窗口大小时都会调用该函数。这个调整大小回调函数具有以下原型：

    ```C++
    void framebuffer_size_callback(GLFWwindow* window, int width, int height);
    ```

+ 帧缓冲区大小函数采用 `GLFWwindow` 作为其第一个参数和两个指示新窗口尺寸的整数。每当窗口大小发生变化时， `GLFW` 都会调用此函数并填写适当的参数供您处理。

    ```C++
    void framebuffer_size_callback(GLFWwindow* window, int width, int height)
    {
        glViewport(0, 0, width, height);
    } 
    ```

+ 我们必须通过注册来告诉 `GLFW` 我们想要在每次调整窗口大小时调用此函数：

    ```C++
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
    ```

+ 当窗口首次显示时， `framebuffer_size_callback` 也会被调用，并产生窗口尺寸。对于视网膜显示器，宽度和高度最终将明显高于原始输入值。

+ 我们可以设置很多回调函数来注册我们自己的函数。例如，我们可以创建一个回调函数来处理操纵杆输入更改、处理错误消息等。我们在创建窗口之后和启动渲染循环之前注册回调函数。

## 准备好你的引擎

+ 我们不希望应用程序绘制单个图像，然后立即退出并关闭窗口。我们希望应用程序继续绘制图像并处理用户输入，直到程序被明确告知停止。因此，我们必须创建一个 `while` 循环，我们现在将其称为渲染循环，它会继续运行，直到我们告诉 `GLFW` 停止。以下代码显示了一个非常简单的渲染循环：

    ```C++
    while(!glfwWindowShouldClose(window))
    {
        glfwSwapBuffers(window);
        glfwPollEvents();    
    }
    ```

+ `glfwWindowShouldClose` 函数在每次循环迭代开始时检查 `GLFW` 是否已被指示关闭。如果是这样，该函数返回 `true` 并且渲染循环停止运行，之后我们可以关闭应用程序。

+ `glfwPollEvents` 函数检查是否触发了任何事件（如键盘输入或鼠标移动事件），更新窗口状态，并调用相应的函数（我们可以通过回调方法注册）。 `glfwSwapBuffers` 将交换在此渲染迭代期间用于渲染的颜色缓冲区（一个大型 `2D` 缓冲区，其中包含 `GLFW` 窗口中每个像素的颜色值）并将其显示为屏幕的输出。

+ 双缓冲
    * 当应用程序在单个缓冲区中绘制时，生成的图像可能会显示闪烁问题。这是因为生成的输出图像不是立即绘制的，而是逐像素绘制的，通常是从左到右、从上到下。由于该图像在渲染过程中不会立即显示给用户，因此结果可能包含伪影。为了避免这些问题，窗口应用程序应用双缓冲区进行渲染。前缓冲区包含屏幕上显示的最终输出图像，而所有渲染命令都绘制到后缓冲区。一旦所有渲染命令完成，我们就将后台缓冲区交换到前台缓冲区，以便可以显示图像而无需渲染，从而消除所有上述伪影。

## 最后一件事

+ 一旦退出渲染循环，我们就希望正确清理/删除所有已分配的 `GLFW` 资源。我们可以通过在主函数末尾调用的 `glfwTerminate` 函数来完成此操作

    ```C++
    glfwTerminate();
    return 0;
    ```

+ 这将清理所有资源并正确退出应用程序。现在尝试编译您的应用程序，如果一切顺利，您应该看到以下输出：

+ 如果这是一个非常沉闷无聊的黑色图像，那么你做对了！如果您没有获得正确的图像，或者您对所有内容如何组合在一起感到困惑，请在此处查看完整的源代码（如果它开始闪烁不同的颜色，请继续阅读）。

+ 如果您在编译应用程序时遇到问题，请首先确保所有链接器选项设置正确，并且您在 IDE 中正确包含了正确的目录（如上一章所述）。还要确保您的代码正确；您可以通过将其与完整源代码进行比较来验证它。

## 输入

+ 我们还希望在 `GLFW` 中有某种形式的输入控制，我们可以通过 `GLFW` 的几个输入函数来实现这一点。我们将使用 `GLFW` 的 `glfwGetKey` 函数，该函数将窗口与键一起作为输入。该函数返回当前是否按下该键。我们正在创建一个 `processInput` 函数来组织所有输入代码：

    ```C++
    void processInput(GLFWwindow *window)
    {
        if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
            glfwSetWindowShouldClose(window, true);
    }
    ```

+ 这里我们检查用户是否按下了退出键（如果没有按下， `glfwGetKey` 返回 `GLFW_RELEASE` ）。如果用户确实按下了转义键，我们将使用 `glfwSetwindowShouldClose` 将其 `WindowShouldClose` 属性设置为 `true` 来关闭 `GLFW` 。主 `while` 循环的下一个条件检查将失败并且应用程序关闭。
  
+ 然后，我们在渲染循环的每次迭代中调用 `processInput` ：

    ```C++
    while (!glfwWindowShouldClose(window))
    {
        processInput(window);

        glfwSwapBuffers(window);
        glfwPollEvents();
    }
    ```

+ 这为我们提供了一种简单的方法来检查特定的按键并在每一帧做出相应的反应。渲染循环的迭代通常称为帧。

## 渲染

+ 我们希望将所有渲染命令放入渲染循环中，因为我们希望在循环的每次迭代或帧中执行所有渲染命令。这看起来有点像这样：

    ```C++
    // render loop
    while(!glfwWindowShouldClose(window))
    {
        // input
        processInput(window);

        // rendering commands here
        ...

        // check and call events and swap the buffers
        glfwPollEvents();
        glfwSwapBuffers(window);
    }
    ```

+ 只是为了测试事情是否真的有效，我们想用我们选择的颜色来清除屏幕。在帧开始时我们想要清除屏幕。否则，我们仍然会看到前一帧的结果（这可能是您正在寻找的效果，但通常您不会）。我们可以使用 `glClear` 清除屏幕的颜色缓冲区，其中我们传入缓冲区位来指定要清除的缓冲区。我们可以设置的可能位是 `GL_COLOR_BUFFER_BIT` 、 `GL_DEPTH_BUFFER_BIT` 和 `GL_STENCIL_BUFFER_BIT` 。现在我们只关心颜色值，所以我们只清除颜色缓冲区。

    ```C++
    glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT);
    ```

+ 请注意，我们还使用 `glClearColor` 指定清除屏幕的颜色。每当我们调用 `glClear` 并清除颜色缓冲区时，整个颜色缓冲区都会填充由 `glClearColor` 配置的颜色。这将导致深绿色-蓝色。

+ 您可能还记得 `OpenGL` 章节中的内容， `glClearColor` 函数是一个状态设置函数，而 `glClear` 是一个状态使用函数，因为它使用当前状态来检索清除颜色。

+ 可以在此处找到该应用程序的完整源代码。

+ 现在我们已经做好了一切准备，可以用大量渲染调用来填充渲染循环，但这是下一章的内容。我想我们在这里闲逛已经够久了。