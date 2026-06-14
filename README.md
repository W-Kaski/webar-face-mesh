# Hand Quad AR

实时双手追踪 + 手指间透视四边形 + 插件化特效系统。

## 效果

双手伸开手指 → 相邻手指间出现透视四边形，四边形作为特效视窗，透过它看到摄像头画面叠加不同艺术风格特效。

- 四边形外：正常摄像头画面，无处理
- 四边形内：同一摄像头画面 + GPU 实时艺术滤镜
- 四边形随手指实时移动、变形，手移走后平滑淡出

## 当前特效配置

| Quad | 位置 | 效果 |
|------|------|------|
| Quad 0 | 拇指 ↔ 食指 | PopArt4 — Warhol 波普四色 |
| Quad 1 | 食指 ↔ 中指 | GreenPix — 绿底像素风 |
| Quad 2 | 中指 ↔ 小指 | RedHalf — 红色 halftone 网点 |

## 插件化特效系统

特效以 `FX` 数组注册，每项包含名称、默认颜色、GLSL shader 代码。  
`EFFECTS` 数组通过 `fxIdx` 引用 FX 注册表，即插即用。

```js
// 添加新特效只需在 FX 数组追加一项：
{ name: 'MyEffect', c1: [...], c2: [...], c3: [...],
  code: `vec3 fx_N(vec2 suv, vec3 bg, float m, float lum, float edg, bool isP){ ... }` }

// 然后在 EFFECTS 里引用：
{ stencilRef: 1, fxIdx: N }
```

已注册 11 种特效：RedHalftone, BluePoster, GreenHalf, Lichtenstein, NeonGlow, Thermal, PopArt4, Glitch, RedHalf, Sketch, GreenPix。

## 技术栈

| 组件 | 作用 |
|---|---|
| MediaPipe Hands | 双手 21 点实时追踪 |
| MediaPipe SelfieSegmentation | 人物/背景像素级分割 |
| Three.js r157 | WebGL 渲染，Stencil Buffer 管线，自定义 Shader |

## 渲染管线

```
renderer.clear(stencil=0)
  → Stencil Quads (renderOrder=5): 写入 stencilRef 1/2/4
  → Effect Passes (renderOrder=10+): 读 stencil，只在匹配区域画特效
  → CSS <video> 作为底层背景，Three.js canvas 透明叠加
```

## 性能优化

- Hands: 每帧（modelComplexity=0）
- SelfieSegmentation: 每 5 帧
- Shader: GPU 实时合成，无 CPU 像素操作
- Stencil Buffer: 零额外 draw call 开销
