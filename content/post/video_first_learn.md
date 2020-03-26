---
title: "Video_first_learn"
date: 2019-12-21T20:05:04+08:00
draft: true
---

## 音频流程

> 采样、量化、编码
>
> - 采样 奈奎斯特定律，按人耳能听到的频率的二倍进行采集，人耳可听到的范围是 20Hz～20kHz，故采用44.1kHz频率进行采集，代表1秒会采样44100次。
>
>   摘录来自: 展晓凯. “音视频开发进阶指南：基于Android与iOS平台的实践。” Apple Books. 
>
> - 量化 使用16bit进行数据采集，也就是每个声音的振幅在[-32768，32767]
>
> - 编码 顺序存储或压缩存储，去除冗余信息，包含人耳不因感觉的信息。



## 视频流程

- YUV 4：2：0的模型进行存储，数据量少
- YUV 转化 RGB 比较典型的场景是在iOS平台中使用摄像头采集出YUV数据之后，上传显卡成为一个纹理ID，这个时候就需要做YUV到RGB的转换（具体的细节会在后面的章节中详细讲解）。在iOS的摄像头采集出一帧数据之后（CMSampleBufferRef），我们可以在其中调用CVBufferGetAttachment来获取YCbCrMatrix，用于决定使用哪一个矩阵进行转换。

### 视频编码

- 帧间编码技术
- I帧 关键帧，依靠当前帧可以解码出整张帧。
- P帧 解码依赖于前一帧的数据。
- B帧 解码依赖于前后帧的数据。
- IDR帧 特别的I帧，鉴于P帧可能依赖于一个未解出的帧数据，所以出现IDR帧后，清空所有解码数据，后面所有依赖I帧的P帧解码都依赖于IDR帧。

> video encode include two part:视频编码层(VCL, Video Coding Layer)和网络提取层(NAL, Network Abstraction Layer), https://www.jianshu.com/p/9f627701aa1e

## OPENGL 学习

- glBindAttribLocation(programShader,0, "VertexPosition"); 绑定一个int类型到shader的变量。 参数1：shader程序；参数2：参数index；参数3：shader 变量名

- glVertexAttribPointer(ATTRIBUTE_VERTEX, 2, GL_FLOAT, 0, 0, _vertices); 给shader中指定变量设置值。

- glEnableVertexAttribArray(ATTRIBUTE_VERTEX); 开启变量功能。

- glGetUniformLocation(_program, "ourColor");   用来查询查询uniform `ourColor`的位置值。我们为查询函数提供着色器程序和uniform的名字（这是我们希望获得的位置值的来源）。如果glGetUniformLocation返回`-1`就代表没有找到这个位置值。最后，我们可以通过glUniform4f函数设置uniform值。

- glUniformMatrix4fv(GLint location, GLsizei count, GLboolean transpose, const GLfloat *value);