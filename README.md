# Hand Quad AR

实时双手追踪 + 手指间透视四边形 + 人脸 Mesh 纹理叠加。

## 效果

双手伸开手指 → 相邻手指间出现透视菱形（四边形），每个菱形内叠加人脸 Mesh 纹理。手指弯曲/靠近时四边形自动变为三角形。只有一只手时隐藏所有四边形。

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
| [MediaPipe Face Mesh](https://google.github.io/mediapipe/solutions/face_mesh.html) | 468 点人脸 Mesh 检测，生成纹理 |
| [Three.js](https://threejs.org/) | 3D 渲染，四边形/三角形几何 + 自定义 Shader |

## 交互

- 手指伸展 → 四边形放大；手指弯曲 → 四边形缩小
- 双手靠近 → 辉光增强 + 颜色色相偏移
- 手指太近（退化） → 四边形自动变为三角形
- 单手 → 所有四边形隐藏
- 边缘有脉冲呼吸光效

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
       5. 人脸 Mesh 纹理贴到每个四边形上，Shader 添加辉光 + 边缘脉冲
```
