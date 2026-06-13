# Hand Quad AR

实时双手追踪 + 手指间连续透视纸带 + 人物剪影驱动的三材料印刷纹理。

## 效果

双手伸开手指 → 相邻手指间出现连续纸带面片，每个面片由左右手对应手指锚定。纸带使用三种材料：拇指到食指是红色半调点阵，食指到中指是蓝色油墨纹理，中指到小指是连续红色半调纹理。人物分割 mask 和 Face Mesh 脸部范围共同驱动剪影/脸部阴影。手指弯曲/靠近时面片自动退化为三角形。只有一只手时隐藏所有面片。

```
左手                                              右手
  👆thumb══════════════════════════════thumb👆
    ║        蓝色四边形 (thumb↔index)     ║
  👆index══════════════════════════════index👆
    ║        绿色四边形 (index↔middle)    ║
  👆middle══════════════════════════════middle👆
    ║        红色四边形 (middle↔ring)     ║
  👆ring════════════════════════════════ring👆
    ║        红色四边形 (ring↔pinky)      ║
  👆pinky═══════════════════════════════pinky👆
```

## 技术栈

| 组件 | 作用 |
|---|---|
| [MediaPipe Hands](https://google.github.io/mediapipe/solutions/hands.html) | 双手 21 点实时追踪 |
| [MediaPipe Face Mesh](https://google.github.io/mediapipe/solutions/face_mesh.html) | 468 点人脸检测，驱动半调头像纹理 |
| [MediaPipe Selfie Segmentation](https://google.github.io/mediapipe/solutions/selfie_segmentation.html) | 生成人物剪影 mask |
| [Three.js](https://threejs.org/) | 3D 渲染，连续四边形/三角形纸带 + 自定义 Shader |

## 交互

- 手指伸展/旋转 → 纸带跟随对应手指形成透视梯形
- 双手靠近 → 绿色辉光增强
- 手指太近（退化） → 四边形自动变为三角形
- 单手 → 所有四边形隐藏
- 红色材料在无名指轨道附近保留留白，避免每个手指缝重复点阵

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

## 原理

```
摄像头画面
  ├─ MediaPipe Hands → 双手各 21 个 landmark（归一化坐标）
  ├─ MediaPipe Face Mesh → 468 个人脸 landmark → 绘制到 Canvas 纹理
  └─ Three.js 渲染：
       1. 将指尖 landmark 转换为 NDC 坐标（镜像翻转）
       2. 沿手指方向向内延伸 30%（覆盖手指主体）
       3. 预计算所有指尖位置（相邻四边形共享边 = 无缝拼接）
       4. 检测退化：对角线 < 阈值 → 合并顶点为三角形
       5. Selfie Segmentation 生成前景 mask，Face Mesh 提供脸部 bounds
       6. 三个 CanvasTexture 分别生成红色半调、蓝色油墨、下半段红色半调
       7. Shader 将纹理贴到对应手指面；中指→小指共享同一张红色纹理并用 UV 偏移保持连续
```
