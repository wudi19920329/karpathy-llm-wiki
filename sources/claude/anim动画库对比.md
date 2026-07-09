---
title: "动画库anim对比"
source: "https://claude.ai/chat/22e44786-aad9-419e-ad9d-202cdbc5c9f9"
author:
published:
created: 2026-07-09
description: "Claude conversation with 2 messages"
tags:
  - "clippings"
---
**You**

做动画 [https://github.com/remotion-dev/remotion这个是最强的吗](https://github.com/remotion-dev/remotion%E8%BF%99%E4%B8%AA%E6%98%AF%E6%9C%80%E5%BC%BA%E7%9A%84%E5%90%97)

---

**Claude**

Remotion 确实很强，但"最强"要看你想做什么样的动画——它是一个用 React 写视频的框架，最适合的场景是 **程序化生成视频** ，而不是泛指所有动画。

它的优势在于:你可以用代码(React 组件)描述每一帧,然后渲染成真正的 mp4 视频。所以特别适合数据驱动的视频(比如年度报告、个性化视频批量生成)、需要版本控制和复用的视频内容、开发者做产品演示或宣传片。生态也成熟,有付费的公司在背后维护。

但换个场景,它就不一定是最佳选择了:

- **代码讲解/教学动画**:Motion Canvas 更顺手,它的时间线基于 generator 函数,做"逐步演示"类动画比 Remotion 直觉得多
- **数学/科学动画**:Manim(3Blue1Brown 那种风格)是事实标准
- **网页里的交互动画** (不是导出视频):GSAP、Framer Motion、CSS 动画才是正路,用 Remotion 反而是杀鸡用牛刀
- **设计师主导的动效**:After Effects + Lottie 导出到 web/app,工作流更成熟
- **3D 动画**:Blender,或者代码向的 Three.js(Remotion 也能嵌 Three.js,但只是渲染管线)

另外注意一点:Remotion 对公司使用是要付许可费的(个人和小团队免费),这在选型时值得考虑。

所以简单说:**如果你是开发者、想用代码生成视频文件,Remotion 大概率是最好的选择;如果是其他类型的"动画",各有更专业的工具。** 你具体想做哪类动画?我可以给更具体的建议。