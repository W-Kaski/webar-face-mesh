# Hand Quad AR

实时双手追踪 + 手指间透视四边形 + 人物特效视窗。

## 效果

双手伸开手指 → 相邻手指间出现透视四边形，四边形作为特效视窗，透过它看到摄像头画面中的人物区域叠加不同特效。

- Quad1 (thumb↔index): 人物区域红色粗描边（白底）
- Quad2 (index↔middle): 人物区域白色剪影（蓝底）  
- Quad3 (middle↔pinky): 人物区域绿色粗描边（白底）

## 技术栈

| 组件 | 作用 |
|---|---|
| MediaPipe Hands | 双手 21 点实时追踪 |
| MediaPipe SelfieSegmentation | 人物/背景像素级分割 |
| MediaPipe Face Mesh | 人脸 468 点检测 |
| Three.js | WebGL 渲染，自定义 Shader 实时合成 |

## 原理

摄像头画面 → Hands landmark 计算四边形位置 + SelfieSegmentation 人物 mask → Shader 里 videoTexture + maskTexture GPU 实时合成 → 人物边缘检测画描边/填充

## 本地运行



## 线上地址

https://w-kaski.github.io/webar-face-mesh/

## 性能优化

- Hands: 每帧（modelComplexity=0）
- FaceMesh: 每 3 帧
- SelfieSegmentation: 每 5 帧
- Shader: GPU 实时合成，无 CPU 像素操作
