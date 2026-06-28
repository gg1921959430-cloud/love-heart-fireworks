# CLAUDE.md

## 项目概述

单文件 HTML 情感交互页面 — 渐变色跳动爱心 + Love 按钮 + 6 种随机烟花效果，整体风格清新少女感。

## 运行方式

- **开发/预览**：直接用浏览器打开 `index.html`，无需构建工具或本地服务器
- **唯一外部依赖**：Google Fonts（Dancing Script 字体），离线环境会自动回退到系统 cursive 字体

```
f:\cursur-python\love\index.html   ← 双击即可运行
```

## 文件结构

```
love/
├── index.html    ← 单文件包含所有 HTML/CSS/JS（约 350 行）
└── 建构计划        ← 原始需求与设计文档（中文）
```

## 技术架构

| 层 | 技术 | 用途 |
|----|------|------|
| 结构 | HTML5 | 页面骨架 |
| 样式 | 内联 CSS | 背景渐变、光斑、SVG 爱心、心跳 keyframes、按钮 |
| 动画 | CSS @keyframes | 心跳 lub-dub 双跳节律（0.83s / 72bpm） |
| 交互 | Canvas 2D + requestAnimationFrame | 烟花粒子引擎，单帧循环驱动所有粒子 |
| 事件 | DOM 事件 | animationiteration 驱动配色切换、click 触发烟花、resize 自适应画布 |

## 核心模块

### 1. 爱心系统
- **形状**：SVG `<path>`，使用心形参数方程 `x=16sin³t, y=13cos(t)-5cos(2t)-2cos(3t)-cos(4t)` 生成
- **配色**：7 种 `radialGradient`（玫红/珊瑚/紫罗兰/樱花粉/蜜桃/海蓝/薰衣草），存储在 SVG `<defs>` 中
- **切换时机**：监听 `heartWrapper` 的 `animationiteration` 事件，每次心跳循环结束时自动切换到下一个渐变
- **发光**：`drop-shadow` 滤镜跟随当前配色动态变化

### 2. 烟花系统
- **画布**：固定定位全屏 Canvas，`pointer-events: none` 不阻挡按钮
- **尺寸**：`window.resize` 时同步更新宽高
- **粒子对象池**：最大 6000 个复用对象，活跃粒子上限 2500
- **渲染**：单个 `requestAnimationFrame` 循环，无活动粒子时自动停止

### 3. 6 种烟花样式

| # | 函数名 | 粒子数 | 绽放原点 | 特色 |
|---|--------|--------|---------|------|
| 1 | `burstSphere` | 80–150 | 爱心中心 | 圆周均匀扩散 |
| 2 | `burstHeart` | 100–150 | 爱心中心 | 粒子沿心形曲线 + 法线方向扩散 |
| 3 | `burstSpiral` | 80–140 | 爱心中心 | 3–6 臂旋转，每粒子独立角速度 |
| 4 | `burstStar` | 80–120 | 爱心中心 | 五角放射 + 短尾迹 |
| 5 | `burstDoubleRing` | 100–140 | 爱心中心 | 内外两圈不同色系 |
| 6 | `burstShootingStars` | 22–40 | 爱心上方居中区域 | 向上抛射 + 重力回落 + 渐变尾迹 |

### 4. Love 按钮
- 圆角胶囊形，渐变背景（粉→紫）
- hover：上浮 + 扩大发光
- 点击：`pop` class 瞬态缩放弹跳（`btnPop` keyframes 0.35s）

## 关键函数索引

| 函数 | 位置 | 说明 |
|------|------|------|
| `generateHeartPath(cx, cy, scale)` | `<script>` 开头 | 生成 SVG path 的 d 属性字符串 |
| `heartPoint(t, scale)` | 烟花系统区域 | 心形参数方程，烟花计算用 |
| `heartNormal(t, scale)` | 烟花系统区域 | 心形曲线法线方向，心形烟花扩散方向用 |
| `getHeartCenter()` | 烟花系统区域 | 动态获取爱心在视口中的中心坐标 |
| `resizeCanvas()` | 烟花系统区域 | Canvas 尺寸跟随窗口 |
| `getParticle()` / `releaseParticle(p)` | 对象池 | 粒子复用，避免 GC |
| `animate()` | 渲染循环 | 统一更新+绘制所有活跃粒子 |
| `STYLE_FUNCTIONS[]` | 样式表 | 6 个样式函数数组，点击时随机选取 |

## 配色切换流程

```
心跳动画一个周期 ≈ 0.83s
  ↓
animationiteration 事件在 heartWrapper 上触发
  ↓
currentGradient = (currentGradient % 7) + 1
  ↓
heartPath.setAttribute('fill', 'url(#gradN)')
  ↓
同步更新 SVG 的 drop-shadow 发光色
```

## 粒子生命周期

```
点击 Love 按钮
  → getHeartCenter() 获取坐标
  → Math.random() 选样式
  → STYLE_FUNCTIONS[i](cx, cy) 从对象池取粒子，赋初值
  → 若无活动渲染循环，启动 requestAnimationFrame(animate)
  → animate() 每帧：更新位置/速度/透明度 → 绘制 → alpha≤0 时回收粒子到对象池
  → 所有粒子消失后停止 requestAnimationFrame
```

## 版本与推送规范

### 版本号规则

- 格式：`v主版本.次版本.修订号`（如 `v1.0.0`）
- 修订号采用**十进制递增**：`v1.0.0` → `v1.0.1` → … → `v1.0.9` → `v1.1.0`（满十进一）
- 每次推送到远程仓库前，必须递增版本号（通常是修订号 +1）

### 推送流程

每次用户说"推送"或将当前版本推送到远程仓库时，执行以下步骤：

```
1. 查看当前最新的 tag：git tag --sort=-v:refname | head -1
2. 递增版本号（修订号 +1，满十向次版本号进位）
3. git add <变更文件>
4. git commit -m "v<新版本号>: <一句话描述改动或新增内容>"
5. git tag -a v<新版本号> -m "<详细说明改动或新增了什么>"
6. git push -u origin master --tags
```

### 示例

```
当前最新 tag: v1.0.0
用户说"推送" → 新版本号: v1.0.1

git add index.html
git commit -m "v1.0.1: 修复烟花粒子闪烁问题"
git tag -a v1.0.1 -m "修复：烟花粒子在低帧率下闪烁，优化了渲染时序"
git push -u origin master --tags
```

### 远程仓库

- **地址**：`git@github.com:gg1921959430-cloud/love-heart-fireworks.git`
- **连接方式**：SSH
- **当前版本**：v1.0.0

---

## 注意事项

- 修改爱心形状时，需要同时改 `generateHeartPath`（SVG 生成）和 `heartPoint`/`heartNormal`（烟花用），保持公式一致
- 新增配色需要同步更新 3 处：SVG `<defs>` 中的 `<radialGradient>`、`GLOW_COLORS` 数组、`TOTAL_GRADIENTS` 常量
- 新增烟花样式只需编写样式函数并追加到 `STYLE_FUNCTIONS` 数组末尾
- 粒子系统有硬上限保护（MAX_ACTIVE=2500），防止性能下降
- 按钮的 `pop` 动画依赖 `void btnLove.offsetWidth` 强制回流，确保重复点击时动画可重新触发
