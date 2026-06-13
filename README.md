# Hand Quad AR

实时双手追踪 + 手指间透视四边形 + 人物特效视窗。

## 效果

双手伸开手指 → 相邻手指间出现透视四边形，四边形作为"特效视窗"，透过它看到摄像头画面中的人物区域叠加不同特效。

```
左手                                              右手
  👆thumb══════════════════════════════thumb👆
    ║  Quad1: 人物区域红色粗描边（白底）  ║
  👆index══════════════════════════════index👆
    ║  Quad2: 人物区域白色剪影（蓝底）   ║
  👆middle══════════════════════════════middle👆
    ║  Quad3: 人物区域绿色粗描边（白底）  ║
  👆pinky═══════════════════════════════pinky👆
  (ring finger 不参与)
```

## 技术栈

| 组件 | 作用 |
|---|---|
| [MediaPipe Hands](https://google.github.io/mediapipe/solutions/hands.html) | 双手 21 点实时追踪 |
| [MediaPipe SelfieSegmentation](https://google.github.io/mediapipe/solutions/selfie_segmentation.html) | 人物/背景像素级分割 |
| [MediaPipe Face Mesh](https://google.github.io/mediapipe/solutions/face_mesh.html) | 人脸 468 点检测 |
| [Three.js](https://threejs.org/) | WebGL 渲染，自定义 Shader 实时合成 |

## 原理

```
摄像头画面
  ├─ MediaPipe Hands → 双手 landmark → 计算四边形四个角的位置
  ├─ MediaPipe SelfieSegmentation → 人物 mask（每 5 帧更新）
  └─ Three.js Shader:
       videoTexture + maskTexture 传入 GPU
       每个像素：
         mask < 0.2 → 显示底色（白/蓝/白）
         mask > 0.2 且是边缘 → 显示对应颜色描边（红/绿）
         mask > 0.2 且是内部 → 白色填充（仅 Quad2）
       所有计算在 GPU 完成，无 CPU 像素遍历
```

## 交互

- 手指伸展 → 四边形跟随手指形成透视梯形
- 手指弯曲/靠近 → 四边形退化为三角形
- 单手 → 所有四边形隐藏
- 人物遮挡区域自动显示对应特效

## 本地运行

摄像头需要 `localhost` 或 HTTPS。

```bash
cd /home/EK/projects/webar-face-mesh
python3 -m http.server 8080
```

打开 http://localhost:8080 ，用 Chrome 效果最佳。

## 线上地址

```
https://w-kaski.github.io/webar-face-mesh/
```

## 文件结构

```
webar-face-mesh/
├── index.html    ← 单文件，CDN 加载依赖，无需构建
└── README.md
```

## 性能优化

- Hands: 每帧推理（modelComplexity=0）
- FaceMesh: 每 3 帧推理
- SelfieSegmentation: 每 5 帧推理
- Shader: GPU 实时合成，无 CPU 像素操作
