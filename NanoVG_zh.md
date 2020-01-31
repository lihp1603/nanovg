### NanoVG 渲染引擎

#### NanoVG介绍

NanoVG是OpenGL的一个小的抗锯齿矢量图形绘制库。它有模仿HTML5 canvas API的精益API。它的目标是成为一个用于构建可伸缩的用户界面和可视化的实用而有趣的工具集。

它是一个基于OpenGL的二维矢量图库，用于UI和可视化方向。

#### 绘制图形

使用NanoVG绘制一个简单的图形包括四个步骤:1)开始一个新的形状，2)定义绘制的路径，3)设置填充或描边，4)最后填充或描边路径。

```c
nvgBeginPath(vg);
nvgRect(vg, 100,100, 120,30);
nvgFillColor(vg, nvgRGBA(255,192,0,255));
nvgFill(vg);
```

调用nvgBeginPath()将清除所有现有路径，并开始从零开始。有许多函数定义要绘制的路径，如矩形、圆角矩形和椭圆，或者您也可以使用一些公共的moveTo、lineTo、bezierTo和arcTo API来逐步组合路径。

#### 理解复合(合成)路径

由于渲染后端是在NanoVG中构建的，所以绘制一个复合路径(由多个定义空洞和填充的路径组成)要复杂一些。

NanoVG使用偶数-奇数填充规则，默认情况下路径是逆时针方向缠绕的。

在使用低级别的draw API绘图时，请记住这一点。

为了将一个预定义的形状包装成一个孔，您应该在定义路径之后调用nvgPathWinding(vg, NVG_HOLE)`, or `nvgPathWinding(vg, NVG_CW)这些函数。

#### 渲染错误，怎么办?

a) 确保使用nvgCreatexxx()调用之一创建了NanoVG上下文。

b) 确保你已经用模板缓冲初始化了OpenGL.

c) 确保已清除模板缓冲区

d) 确保所有的渲染调用都发生在nvgBeginFrame()和nvgEndFrame()之间

e) 要启用对OpenGL错误的更多检查，将NVG_DEBUG标记添加到nvgCreatexxx()

f) 如果问题仍然存在，请报告一个问题!

#### 后端渲染器的OpenGL状态

OpenGL后端的状态如下:

当纹理被上传或更新时，下面的像素存储被设置为默认值：GL_UNPACK_ALIGNMENT`, `GL_UNPACK_ROW_LENGTH`, `GL_UNPACK_SKIP_PIXELS`, `GL_UNPACK_SKIP_ROWS.

纹理绑定也受到影响。当用户加载图像或添加新的字体符号时，会发生纹理更新.在调用nvgBeginFrame()和nvgEndFrame()之间根据需要添加Glyphs

整个框架的数据在nvgEndFrame()中进行缓冲buffered和刷新flushed,下面的代码演示了渲染代码所触及的OpenGL状态：

```c
	glUseProgram(prog);
	glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
	glEnable(GL_CULL_FACE);
	glCullFace(GL_BACK);
	glFrontFace(GL_CCW);
	glEnable(GL_BLEND);
	glDisable(GL_DEPTH_TEST);
	glDisable(GL_SCISSOR_TEST);
	glColorMask(GL_TRUE, GL_TRUE, GL_TRUE, GL_TRUE);
	glStencilMask(0xffffffff);
	glStencilOp(GL_KEEP, GL_KEEP, GL_KEEP);
	glStencilFunc(GL_ALWAYS, 0, 0xffffffff);
	glActiveTexture(GL_TEXTURE0);
	glBindBuffer(GL_UNIFORM_BUFFER, buf);
	glBindVertexArray(arr);
	glBindBuffer(GL_ARRAY_BUFFER, buf);
	glBindTexture(GL_TEXTURE_2D, tex);
	glUniformBlockBinding(... , GLNVG_FRAG_BINDING);
```



