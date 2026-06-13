# WebAR Face Mesh

实时 AR 人脸网格追踪 + 标记图叠加效果。

**效果**：打印标记图 → 摄像头识别 → 人脸 Mesh 实时渲染在标记图上 → 移动/翻转标记图，特效跟随变化。

## 技术栈

| 组件 | 作用 |
|---|---|
| [AR.js](https://github.com/AR-js-org/AR.js) | ArUco / Hiro 标记图实时追踪，获取 3D 位姿 |
| [MediaPipe Face Mesh](https://google.github.io/mediapipe/solutions/face_mesh.html) | 468 点人脸 Mesh 实时检测 |
| [Three.js](https://threejs.org/) | 3D 渲染，将 Mesh 投影到标记图坐标系 |
| [A-Frame](https://aframe.io/) | WebXR 场景管理，简化 AR.js 集成 |

## 快速开始

### 1. 启动本地服务器（必须，摄像头需要 localhost 或 HTTPS）

```bash
# Python 3
cd /home/EK/projects/webar-face-mesh
python3 -m http.server 8080

# 或 Node.js
npx serve -l 8080
```

### 2. 打开浏览器

```
http://localhost:8080
```

用 **Chrome**（桌面或 Android）效果最佳。Safari 部分功能可能受限。

### 3. 允许摄像头访问

浏览器会弹出权限请求，点击「允许」。

### 4. 打印或显示标记图

使用 **Hiro 标记图**（AR.js 内置）：

打印这张图，或者在另一个屏幕上显示：
![Hiro Marker](https://raw.githubusercontent.com/AR-js-org/AR.js/master/aframe/examples/marker-training/examples/pattern-files/hiro.png)

> 直接用手机/平板打开这个链接显示标记图也行：https://github.com/AR-js-org/AR.js/raw/master/aframe/examples/marker-training/examples/pattern-files/hiro.png

### 5. 玩起来

1. 将摄像头对准标记图
2. 人脸 Mesh 会以 `#64ffda` 色（青色）叠加到标记图上
3. 移动、翻转、旋转标记图 → Mesh 实时跟随

## 文件结构

```
webar-face-mesh/
├── index.html    ← 单文件，所有逻辑内联（CDN 加载依赖）
└── README.md
```

## 自定义

在 `index.html` 顶部修改常量：

```javascript
const FACE_COLOR   = [100, 255, 218];  // 人脸 Mesh 颜色 (RGB)
const FACE_OPACITY = 0.7;              // 纹理透明度
const POINT_SIZE   = 0.018;            // 3D 点大小
```

## 部署到线上

因为使用摄像头，**必须 HTTPS**。推荐：

- **GitHub Pages**：推送到 GitHub → Settings → Pages → Deploy
- **Vercel**：`vercel deploy` 直接上线
- **Netlify**：拖拽文件夹到 netlify.com

## 原理

```
摄像头画面
  ├─ AR.js 检测标记图 → markerRoot.matrixWorld（4×4 位姿矩阵）
  ├─ MediaPipe 检测人脸 → 468 个归一化 2D 坐标
  └─ 坐标转换：
       NDC → unproject → camera space → world space → marker local space
     渲染：
       face plane (纹理) + point cloud (顶点) → 作为 markerRoot 的子对象
     结果：所有效果随标记图移动/旋转
```
