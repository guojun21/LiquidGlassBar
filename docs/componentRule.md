# 液态玻璃组件设计规范

## 🎯 规范目的

本文档定义了液态玻璃（LiquidGlass）组件库的设计规范，确保所有组件保持一致的视觉效果和交互体验。

**基准组件**: `LiquidGlassDock.vue` - 所有规范以此组件为标准

---

## 🏗️ 第一条规范：结构设计

### 黄金双层容器结构 ⭐⭐⭐⭐⭐

**规则**: 所有液态玻璃组件必须采用双层容器结构

```html
<!-- 标准结构模板 -->
<div class="liquidGlass-wrapper 组件名">  
  <!-- 第1层：效果层 - SVG 滤镜 -->
  <div class="liquidGlass-effect"></div>
  
  <!-- 第2层：着色层 - 半透明白色 -->
  <div class="liquidGlass-tint"></div>
  
  <!-- 第3层：高光层 - 内阴影 -->
  <div class="liquidGlass-shine"></div>
  
  <!-- 第4层：内容层 -->
  <div class="liquidGlass-text">
    <div class="内容容器">  
      <!-- 实际内容 -->
    </div>
  </div>
</div>
```

### 职责划分

| 层级 | 类名 | 职责 | 禁止 |
|------|------|------|------|
| 外层 | `.liquidGlass-wrapper` | 毛玻璃效果、边缘定义 | ❌ 不能有 padding |
| 内层 | `.内容容器` | 内容布局、间距控制 | - |

### 违规示例
```css
/* ❌ 错误：外层有 padding */
.liquidGlass-wrapper.组件名 {
  padding: 0.5rem;  /* 导致边缘混乱！*/
}

/* ✅ 正确：padding 在内层 */
.内容容器 {
  padding: 0.5rem;
}
```

---

## 🎨 第二条规范：四层极简原则

### 规则：各层只使用 inset: 0 ⭐⭐⭐⭐⭐

**四层的标准写法**:

```css
/* 效果层 - 最简洁的定义 */
.liquidGlass-effect {
  position: absolute;
  z-index: 0;
  inset: 0;                      /* ✅ 只用这个 */
  backdrop-filter: blur(3px);
  filter: url(#滤镜ID);
  overflow: hidden;
  isolation: isolate;
  /* ❌ 不要添加: width, height, border-radius */
}

/* 着色层 */
.liquidGlass-tint {
  z-index: 1;
  position: absolute;
  inset: 0;                      /* ✅ 只用这个 */
  background: rgba(255, 255, 255, 0.25);
  /* ❌ 不要添加: width, height, border-radius */
}

/* 高光层 */
.liquidGlass-shine {
  position: absolute;
  inset: 0;                      /* ✅ 只用这个 */
  z-index: 2;
  overflow: hidden;
  box-shadow: 
    inset 2px 2px 1px 0 rgba(255, 255, 255, 0.5),
    inset -1px -1px 1px 1px rgba(255, 255, 255, 0.5);
  /* ❌ 不要添加: width, height, border-radius */
}

/* 内容层 */
.liquidGlass-text {
  z-index: 3;
  font-size: 2rem;
  color: black;
  /* ❌ 不要添加: width, height, border-radius */
}
```

### 为什么要极简

1. **边缘渲染**: 额外属性干扰 SVG 滤镜的边界计算
2. **子像素问题**: width/height 可能与 inset 产生亚像素差异
3. **圆角继承**: border-radius: inherit 可能继承错误的父元素
4. **稳定性**: 越简单越不容易出问题

### 验证标准
✅ 边缘平滑无折射混乱  
✅ 四个圆角整齐一致  
✅ 悬停时边缘保持完整  

---

## 💫 第三条规范：果冻弹性效果

### 果冻效果的三要素 ⭐⭐⭐⭐⭐

**规则**: 所有可交互组件必须实现果冻弹性效果

#### 要素1: padding 增大
```css
/* 正常状态 */
.内容容器 {
  padding: 基础值;
}

/* 悬停状态 */
.组件名:hover .内容容器 {
  padding: 基础值 + 增幅;  /* 增幅建议 20%-50% */
}
```

**推荐增幅**:
- 轻微：+0.1rem - 0.2rem
- 适中：+0.2rem - 0.4rem（推荐）
- 强烈：+0.4rem - 0.6rem

#### 要素2: border-radius 增大
```css
/* 正常状态 */
.内容容器 {
  border-radius: 基础值;
}

/* 悬停状态 */
.组件名:hover .内容容器 {
  border-radius: 基础值 + 增幅;  /* 建议增加 0.3-0.7rem */
}
```

#### 要素3: 弹性贝塞尔曲线
```css
transition: 
  padding 0.4s cubic-bezier(0.175, 0.885, 0.32, 2.2),
  border-radius 0.4s cubic-bezier(0.175, 0.885, 0.32, 2.2);
```

**标准贝塞尔曲线**: `cubic-bezier(0.175, 0.885, 0.32, 2.2)`

**参数说明**:
- P2y = 2.2（大于 1.0）→ 超出目标值 20%
- 创造"膨胀-回弹"的果冻质感

**调整建议**:
- 更强回弹：2.2 → 2.5 - 3.0
- 更弱回弹：2.2 → 1.5 - 1.8

### Dock 组件示例（标准）
```css
.dock {
  padding: 0.6rem;
  border-radius: 2rem;
  transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 2.2);
}

.dock:hover {
  padding: 0.8rem;        /* +33% */
  border-radius: 2.5rem;  /* +25% */
}
```

---

## 🎨 第四条规范：样式定义原则

### 规则1: 不重复定义 box-shadow ⭐⭐⭐⭐

**正确做法**:
```css
/* ✅ 只在 .liquidGlass-wrapper 定义一次 */
.liquidGlass-wrapper {
  box-shadow: 0 6px 6px rgba(0, 0, 0, 0.2), 0 0 20px rgba(0, 0, 0, 0.1);
}

/* ✅ 内层容器不定义 box-shadow */
.内容容器 {
  /* 不要重复定义 box-shadow */
}
```

**错误做法**:
```css
/* ❌ 在多处定义 box-shadow */
.liquidGlass-wrapper {
  box-shadow: ...;
}

.组件名 {
  box-shadow: ...;  /* 重复定义，导致叠加和渲染问题！*/
}
```

### 规则2: 必须有 cursor: pointer ⭐⭐⭐

```css
.liquidGlass-wrapper {
  cursor: pointer;  /* 可交互组件必须有 */
}

/* 例外：纯展示组件可用 cursor: default */
```

### 规则3: 圆角联动系统 ⭐⭐⭐⭐

```css
/* 外层和内层圆角保持一致 */
.组件名,
.组件名 > div {
  border-radius: 2rem;
}

/* 悬停时同步增大 */
.组件名:hover,
.组件名:hover > div {
  border-radius: 2.5rem;
}
```

---

## 🔮 第五条规范：SVG 滤镜标准

### 标准6步滤镜链 ⭐⭐⭐⭐⭐

**规则**: 所有液态玻璃组件使用相同的 SVG 滤镜参数

```xml
<filter id="glass-distortion-组件名" x="0%" y="0%" width="100%" height="100%" filterUnits="objectBoundingBox">
  <!-- 第1步: 生成分形噪声纹理 -->
  <feTurbulence 
    type="fractalNoise" 
    baseFrequency="0.01 0.01" 
    numOctaves="1" 
    seed="5" 
    result="turbulence" 
  />
  
  <!-- 第2步: 调整噪声的颜色通道，控制扭曲方向 -->
  <feComponentTransfer in="turbulence" result="mapped">
    <feFuncR type="gamma" amplitude="1" exponent="10" offset="0.5" />
    <feFuncG type="gamma" amplitude="0" exponent="1" offset="0" />
    <feFuncB type="gamma" amplitude="0" exponent="1" offset="0.5" />
  </feComponentTransfer>
  
  <!-- 第3步: 软化噪声纹理，使扭曲更平滑 -->
  <feGaussianBlur in="turbulence" stdDeviation="3" result="softMap" />
  
  <!-- 第4步: 添加镜面高光效果，模拟玻璃反射 -->
  <feSpecularLighting 
    in="softMap" 
    surfaceScale="5" 
    specularConstant="1" 
    specularExponent="100" 
    lighting-color="white" 
    result="specLight"
  >
    <fePointLight x="-200" y="-200" z="300" />
  </feSpecularLighting>
  
  <!-- 第5步: 合成高光效果 -->
  <feComposite 
    in="specLight" 
    operator="arithmetic" 
    k1="0" 
    k2="1" 
    k3="1" 
    k4="0" 
    result="litImage" 
  />
  
  <!-- 第6步: 应用位移映射，创建液态扭曲效果 -->
  <feDisplacementMap 
    in="SourceGraphic" 
    in2="softMap" 
    scale="150" 
    xChannelSelector="R" 
    yChannelSelector="G" 
  />
</filter>
```

### 关键参数（不可修改）

| 参数 | 值 | 说明 |
|------|---|------|
| baseFrequency | 0.01 0.01 | 低频噪声，大片扭曲 |
| numOctaves | 1 | 单层噪声 |
| seed | 5 | 随机种子 |
| stdDeviation | 3 | 模糊强度 |
| scale | 150 | 扭曲强度 |

**注意**: 这些参数经过验证，修改可能导致边缘混乱！

---

## 🎪 第六条规范：动画系统

### 统一的贝塞尔曲线 ⭐⭐⭐⭐⭐

**规则**: 所有过渡动画使用统一的弹性曲线

```css
/* 标准弹性曲线 */
cubic-bezier(0.175, 0.885, 0.32, 2.2)
```

**参数解析**:
```
P1: (0.175, 0.885) - 控制起始加速
P2: (0.32, 2.2)    - 控制结束回弹
                     ↑
                   关键！Y值2.2 > 1.0
                   动画会超出目标20%后回弹
```

**动画时长标准**:
- 主要过渡：0.4s
- 快速反馈：0.2s
- 缓慢展开：0.6s

**应用场景**:
```css
/* 容器果冻效果 */
transition: 
  padding 0.4s cubic-bezier(0.175, 0.885, 0.32, 2.2),
  border-radius 0.4s cubic-bezier(0.175, 0.885, 0.32, 2.2);

/* 元素缩放 */
transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 2.2);
```

---

## 📏 第七条规范：数值配置标准

### Padding 体系

```css
/* 基础值范围 */
padding: 0.4rem - 0.6rem

/* 悬停增幅 */
增幅: 0.2rem - 0.4rem (30% - 70%)

/* Dock 标准 */
正常: 0.6rem
悬停: 0.8rem
增幅: 0.2rem (33%)
```

### 圆角体系

```css
/* 外层容器圆角 */
基础: 2rem - 2.5rem
悬停: 基础值 + 0.3rem - 0.7rem

/* 内层元素圆角 */
比例: 外层的 60% - 90%

/* Dock 标准 */
外层正常: 2rem
外层悬停: 2.5rem
内层: 2rem（与外层一致）
```

### 阴影体系

```css
/* 标准双层外阴影 */
box-shadow: 
  0 6px 6px rgba(0, 0, 0, 0.2),    /* 近距投影 */
  0 0 20px rgba(0, 0, 0, 0.1);     /* 扩散光晕 */

/* 标准双层内阴影（高光层）*/
box-shadow: 
  inset 2px 2px 1px 0 rgba(255, 255, 255, 0.5),   /* 左上高光 */
  inset -1px -1px 1px 1px rgba(255, 255, 255, 0.5); /* 右下高光 */
```

### 颜色体系

```css
/* 着色层 */
background: rgba(255, 255, 255, 0.25);  /* 半透明白色 */

/* 文字颜色 */
color: black;  /* 深色背景用白色 */
```

---

## 🎯 第八条规范：交互反馈与状态变化动画

### 规则A: 三态动画标准 ⭐⭐⭐⭐⭐

**规则**: 可交互元素必须有 hover, active 两种状态

```css
/* 正常状态 */
.element {
  transform: scale(1);
}

/* 悬停状态 - 轻微放大 */
.element:hover {
  transform: scale(1.05);       /* 5%-10% 放大 */
  filter: brightness(1.05);     /* 轻微增亮 */
  box-shadow: 阴影增强;
}

/* 激活状态 - 显著反馈 */
.element:active {
  transform: scale(1.35);       /* 30%-40% 放大 */
  filter: brightness(1.15);     /* 明显增亮 */
  box-shadow: 
    多层外阴影,
    多层内高光;
  border-radius: 增大;
}
```

### Dock 图标示例（标准）

```css
/* 正常 */
img { transform: scale(1); }

/* 悬停 */
img:hover { 
  transform: scale(1.05);  /* 5% */
  filter: brightness(1.05);
}

/* 点击 */
img:active { 
  transform: scale(1.35);  /* 35% */
  filter: brightness(1.15);
  border-radius: 16px;     /* 从 12px 增大 */
}
```

---

### 规则B: 状态变化必须有果冻过渡 ⭐⭐⭐⭐⭐ 

**核心原则**: 任何导致界面内容变化的操作，都必须有果冻弹性过渡动画

#### 适用场景

所有会改变界面状态的操作：
- ✅ 点击按钮后内容改变（如：清除、确定、提交）
- ✅ 选择选项后界面更新（如：下拉选择、单选）
- ✅ 切换页签后内容切换
- ✅ 展开/收起面板
- ✅ 添加/删除项目
- ✅ 任何导致布局变化的操作

#### 实现方式

##### 方式1: 按钮点击果冻反馈（推荐）⭐⭐⭐⭐⭐

```css
/* 正常状态 */
.button {
  transform: scale(1);
  transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 2.2);
}

/* 点击时 - 瞬间放大 */
.button:active {
  transform: scale(1.2);  /* 放大 20% */
  box-shadow: 
    0 0.5px 4px 0 rgba(0, 0, 0, 0.12),
    0 6px 13px 0 rgba(0, 0, 0, 0.12),
    inset 1px 1px 0 rgba(255, 255, 255, 0.8),
    inset 0 0 5px rgba(255, 255, 255, 0.8);
  /* 快速弹性过渡 */
  transition: all 0.2s cubic-bezier(0.175, 0.885, 0.32, 2.5);
}
```

**效果**: 
1. 点击瞬间放大到 120%
2. 释放后回弹到 100%
3. 贝塞尔曲线 2.5 创造明显回弹
4. 整个过程约 0.2-0.3 秒

**示例**: DatePicker 的"清除"和"确定"按钮

---

##### 方式2: 内容容器震动反馈

```css
/* 为触发变化的容器添加动画类 */
@keyframes contentChange {
  0%   { transform: scale(1); }
  30%  { transform: scale(1.08); }    /* 膨胀 */
  60%  { transform: scale(0.98); }    /* 收缩 */
  100% { transform: scale(1); }       /* 回弹 */
}

.content-container.changing {
  animation: contentChange 0.4s cubic-bezier(0.175, 0.885, 0.32, 2.2);
}
```

**使用方法**:
```javascript
// 点击清除按钮
clearDates() {
  // 添加动画类
  this.$refs.container.classList.add('changing')
  
  // 清除数据
  this.startDate = null
  this.endDate = null
  
  // 动画结束后移除类
  setTimeout(() => {
    this.$refs.container.classList.remove('changing')
  }, 400)
}
```

---

##### 方式3: 元素淡入淡出 + 缩放

```css
/* 进入动画 */
.fade-enter-active {
  transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 2.2);
}

.fade-enter-from {
  opacity: 0;
  transform: scale(0.8);  /* 从小放大 */
}

.fade-enter-to {
  opacity: 1;
  transform: scale(1);
}

/* 离开动画 */
.fade-leave-active {
  transition: all 0.2s cubic-bezier(0.175, 0.885, 0.32, 2.2);
}

.fade-leave-to {
  opacity: 0;
  transform: scale(1.2);  /* 放大消失 */
}
```

**Vue 使用**:
```vue
<transition name="fade">
  <div v-if="showContent">内容</div>
</transition>
```

---

#### 禁止的做法 ❌

```javascript
/* ❌ 直接瞬间改变，没有过渡 */
clearDates() {
  this.startDate = null  // 瞬间清空，生硬！
}

/* ❌ 使用普通的线性过渡 */
transition: all 0.3s ease;  // 没有弹性！

/* ❌ 没有视觉反馈 */
.button:active {
  /* 什么都不做，用户感知不到点击 */
}
```

---

#### 标准实现示例

##### DatePicker 清除按钮（标准）✅

```css
.clear-btn:active {
  transform: scale(1.2);  /* 点击瞬间放大 */
  box-shadow: 多层内外阴影;
  transition: all 0.2s cubic-bezier(0.175, 0.885, 0.32, 2.5);
}
```

**完整流程**:
```
1. 鼠标按下 → 按钮瞬间放大到 120%
2. 执行清除逻辑 → 日期被清空
3. 鼠标释放 → 按钮回弹到 100%
4. 贝塞尔曲线 2.5 → 超出后回弹，创造果冻感
```

##### Toggle 开关（标准）✅

```css
.toggle:active::before {
  transform: scale(1.35);  /* 滑块放大 */
  box-shadow: 多层阴影;
  transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 2.2);
}
```

**完整流程**:
```
1. 点击 → 滑块放大
2. 状态切换 → Grid 列变化（流变动画）
3. 滑块移动 → 平滑流动
4. 滑块回弹 → 稳定在新位置
```

---

#### 检查清单

所有状态变化操作必须检查：

- [ ] 按钮点击是否有 :active 放大效果
- [ ] 放大后是否有回弹（贝塞尔 Y > 1.0）
- [ ] 过渡时长是否合适（0.2s - 0.4s）
- [ ] 是否使用标准弹性曲线
- [ ] 内容变化是否平滑过渡
- [ ] 用户是否能感知到操作反馈

---

#### 推荐参数

**按钮点击反馈**:
```css
transform: scale(1.2 - 1.35);  /* 放大 20%-35% */
transition: 0.2s cubic-bezier(0.175, 0.885, 0.32, 2.5);  /* Y值 2.5 强回弹 */
box-shadow: 多层内外阴影;
```

**内容变化过渡**:
```css
transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 2.2);
```

---

#### 贝塞尔曲线参数对照

| 场景 | P2y 值 | 回弹强度 | 适用 |
|------|--------|---------|------|
| 容器悬停 | 2.2 | 适中 | 果冻效果 |
| 按钮点击 | 2.5 | 强 | 操作反馈 |
| 快速反馈 | 2.8 | 很强 | 瞬时操作 |
| 平缓动画 | 1.5 | 轻微 | 内容过渡 |

---

### 核心思想 💡

**所有界面变化都是一个"事件"，事件必须有"仪式感"**

- 不是冰冷的瞬间切换
- 而是有生命力的弹性过渡
- 让用户感知到"发生了什么"
- 通过果冻动画提供确认反馈

---

## 🔄 第九条规范：圆角协调性

### 圆角层次系统 ⭐⭐⭐⭐

**规则**: 外层、内层、子元素圆角保持协调比例

```
外层容器:    100% (2rem)
内层容器:    100% (2rem) - 与外层一致
子元素:      60%-90% (1.2-1.8rem)
指示器:      80%-90% (1.6-1.8rem)
```

**Dock 标准**:
```css
.liquidGlass-wrapper.dock { border-radius: 无; }
.dock { border-radius: 2rem; }
.dock > div { border-radius: 2rem; }  /* 子元素同步 */
.dock img { border-radius: 12px; }    /* 60% */
```

**TabBar 标准**:
```css
.tab-bar { border-radius: 2rem; }
.tabs-container { border-radius: 1.8rem; }  /* 90% */
.tab-indicator { border-radius: 1.6rem; }   /* 80% */
.tab { border-radius: 1.6rem; }             /* 80% */
```

### 悬停时圆角联动

```css
/* 外层和内层同步增大 */
.组件名,
.组件名 > div {
  border-radius: 2rem;
}

.组件名:hover,
.组件名:hover > div {
  border-radius: 2.5rem;  /* 同步增大 */
}
```

---

## 🎨 第十条规范：颜色与透明度

### 标准配色方案

```css
/* 着色层背景 */
rgba(255, 255, 255, 0.25)  /* 25% 白色 */

/* 高光 */
rgba(255, 255, 255, 0.5)   /* 50% 白色 */
rgba(255, 255, 255, 0.75)  /* 75% 白色 */

/* 阴影 */
rgba(0, 0, 0, 0.2)  /* 20% 黑色 - 近距投影 */
rgba(0, 0, 0, 0.1)  /* 10% 黑色 - 扩散光晕 */

/* 文字 */
color: black;              /* 深色文字 */
color: rgba(0, 0, 0, 0.8); /* 80% 深色 */
color: rgba(0, 0, 0, 0.6); /* 60% 深色（次要文字）*/
```

### 悬停时透明度变化

```css
/* 元素悬停 */
background: rgba(255, 255, 255, 0.3);  /* 正常 */
background: rgba(255, 255, 255, 0.5);  /* 悬停 - 增强 */
```

---

## 🎯 第十一条规范：响应式设计

### 移动端适配

```css
@media (max-width: 768px) {
  .组件名 {
    /* 减小尺寸 */
  }
}

@media (max-width: 480px) {
  .组件名 {
    border-radius: 1.5rem;  /* 小屏幕圆角减小 */
    /* 进一步减小尺寸 */
  }
}
```

---

## 📋 第十二条规范：组件必备元素

### 所有组件必须包含 ⭐⭐⭐⭐⭐

#### 1. HTML 结构
- ✅ 四层结构（effect, tint, shine, text）
- ✅ 双层容器（外层滤镜 + 内层内容）
- ✅ SVG 滤镜定义

#### 2. CSS 样式
- ✅ `.liquidGlass-wrapper` 基础样式
- ✅ 四层的样式定义
- ✅ 内容容器样式
- ✅ 悬停效果
- ✅ 果冻弹性动画

#### 3. SVG 滤镜
- ✅ 唯一的 filter ID
- ✅ 6步滤镜链
- ✅ 标准参数配置

#### 4. 交互
- ✅ cursor: pointer
- ✅ 悬停状态
- ✅ 激活状态（可选）
- ✅ 过渡动画

---

## ⚠️ 第十三条规范：禁止事项

### 严禁的操作 ❌

#### 1. 外层容器加 padding
```css
/* ❌ 绝对禁止 */
.liquidGlass-wrapper.组件名 {
  padding: 任何值;  /* 导致边缘混乱！*/
}
```

#### 2. 各层添加额外属性
```css
/* ❌ 不要添加 */
.liquidGlass-effect {
  width: 100%;           /* 不需要 */
  height: 100%;          /* 不需要 */
  border-radius: inherit; /* 不需要，反而有害 */
}
```

#### 3. 重复定义 box-shadow
```css
/* ❌ 不要重复 */
.liquidGlass-wrapper { box-shadow: ...; }
.组件名 { box-shadow: ...; }  /* 重复！*/
```

#### 4. 修改 SVG 滤镜参数
```css
/* ❌ 不要随意修改 */
baseFrequency="0.02"  /* 会导致效果变化 */
scale="200"           /* 会导致扭曲过度 */
```

#### 5. 使用复杂的 SVG 滤镜
```xml
<!-- ❌ 避免使用 feImage + 内嵌 SVG -->
<feImage xlink:href="data:image/svg+xml..." />
<!-- 可能导致渲染裂缝 -->
```

---

## 📊 第十四条规范：组件命名

### 命名规则

```
LiquidGlass[功能名]

示例：
- LiquidGlassDock
- LiquidGlassTabBar
- LiquidGlassToggle
- LiquidGlassDatePicker
- LiquidGlassCard (通用容器用 Card)
```

### 类名规则

```css
/* 主容器 */
.liquidGlass-wrapper.组件名

/* 四层固定名称 */
.liquidGlass-effect
.liquidGlass-tint
.liquidGlass-shine
.liquidGlass-text

/* 内容容器 */
.组件名 或 .组件名-container
```

---

## 🔍 第十五条规范：代码注释

### 注释要求

```css
/* ✅ 必须有的注释 */

/* 外层容器圆角 - 修改此值调整整体圆角大小 */
border-radius: 2rem;

/* 悬停时 padding 增大 - 创造果冻扩张效果 */
padding: 0.8rem;

/* SVG 滤镜 - 添加液态扭曲效果 */
filter: url(#glass-distortion);
```

### 关键配置必须注释
- 所有圆角值
- padding 变化
- 动画参数
- 重要的视觉效果

---

## 📐 第十六条规范：尺寸标准

### 最小尺寸要求

```css
/* 组件最小宽度 */
min-width: 280px;

/* 最小高度（可选）*/
min-height: 100px;

/* 圆角最小值 */
border-radius: >= 1rem;  /* 保持圆润感 */
```

### 间距标准

```css
/* 元素间距 */
gap: 0.5rem - 1rem

/* 内边距 */
padding: 0.4rem - 1rem

/* 外边距 */
margin: 根据布局需要
```

---

## 🎨 第十七条规范：通用性要求

### 组件必须支持 ⭐⭐⭐⭐⭐

#### 1. Props 配置
```javascript
// 至少支持基础配置
props: {
  // 尺寸、颜色等基础属性
}
```

#### 2. 插槽系统
```vue
<!-- 支持自定义内容 -->
<slot>默认内容</slot>
```

#### 3. 事件系统
```javascript
// 关键交互必须发射事件
this.$emit('change', value)
this.$emit('confirm', data)
```

#### 4. v-model 支持（数据组件）
```javascript
props: ['modelValue']
emits: ['update:modelValue']
```

---

## 🏛️ 第十八条规范：Vue3 组件设计原则

### 核心设计原则 ⭐⭐⭐⭐⭐

#### 1. 单一职责原则（Single Responsibility）

**规则**: 每个组件只负责一个明确的功能

```vue
<!-- ✅ 正确：职责单一 -->
<LiquidGlassButton @click="handleClick">
  点击
</LiquidGlassButton>

<!-- ❌ 错误：职责混乱 -->
<LiquidGlassButtonWithFormAndValidation>
  <!-- 按钮、表单、验证混在一起 -->
</LiquidGlassButtonWithFormAndValidation>
```

**要求**:
- 一个组件只处理一种业务逻辑
- 避免将多个不相关功能混在一个组件中
- 便于测试、维护和复用

---

#### 2. 组合优于继承（Composition over Inheritance）

**规则**: 利用 Composition API 进行逻辑组合

```javascript
// ✅ 正确：使用 composables 组合逻辑
import { useLiquidGlass } from '@/composables/useLiquidGlass'
import { useAnimation } from '@/composables/useAnimation'
import { useInteraction } from '@/composables/useInteraction'

export default {
  setup() {
    const { glassEffect } = useLiquidGlass()
    const { jellyAnimation } = useAnimation()
    const { handleHover, handleClick } = useInteraction()
    
    return {
      glassEffect,
      jellyAnimation,
      handleHover,
      handleClick
    }
  }
}
```

**Composables 标准结构**:
```javascript
// composables/useLiquidGlass.js
export function useLiquidGlass(options = {}) {
  const filterId = computed(() => `glass-distortion-${uniqueId()}`)
  const baseFrequency = ref(options.baseFrequency || 0.01)
  const scale = ref(options.scale || 150)
  
  const applyFilter = () => {
    // 应用滤镜逻辑
  }
  
  return {
    filterId,
    baseFrequency,
    scale,
    applyFilter
  }
}
```

---

#### 3. 松耦合高内聚

**规则**: 组件间依赖最小化，内部逻辑紧密相关

```vue
<!-- ✅ 正确：通过 props/events 通信 -->
<LiquidGlassCard
  :title="cardTitle"
  @click="handleCardClick"
/>

<!-- ❌ 错误：直接访问父组件 -->
<script>
// this.$parent.someMethod()  // 禁止！
</script>
```

**要求**:
- 通过 props/events 进行通信
- 避免直接访问父子组件
- 相关功能聚集在同一组件内

---

### 组件分类标准 ⭐⭐⭐⭐⭐

#### 1. 基础组件（Base Components）

**定义**: 纯展示型组件，只包含 HTML 元素和基础样式

```
命名规范: LiquidGlass[功能名]
示例:
- LiquidGlassButton
- LiquidGlassInput  
- LiquidGlassCard
- LiquidGlassModal
```

**特征**:
- ✅ 不包含业务逻辑
- ✅ 不依赖全局状态
- ✅ 可在整个应用中复用
- ✅ 只通过 props 接收数据
- ✅ 只通过 events 发送事件

**示例**:
```vue
<!-- LiquidGlassButton.vue -->
<template>
  <div class="liquidGlass-wrapper button" @click="handleClick">
    <div class="liquidGlass-effect"></div>
    <div class="liquidGlass-tint"></div>
    <div class="liquidGlass-shine"></div>
    <div class="liquidGlass-text">
      <slot>{{ label }}</slot>
    </div>
  </div>
</template>

<script setup>
defineProps({
  label: String,
  size: {
    type: String,
    default: 'medium',
    validator: v => ['small', 'medium', 'large'].includes(v)
  }
})

const emit = defineEmits(['click'])

const handleClick = (e) => {
  emit('click', e)
}
</script>
```

---

#### 2. 业务组件（Feature Components）

**定义**: 包含特定业务逻辑的功能组件

```
命名规范: [业务领域][功能名]
示例:
- UserProfileCard
- ProductListItem
- OrderSummaryPanel
- DateRangePicker
```

**特征**:
- ✅ 包含特定业务逻辑
- ✅ 可以使用基础组件构建
- ✅ 针对具体场景优化
- ✅ 可以访问业务状态

**示例**:
```vue
<!-- UserProfileCard.vue -->
<template>
  <LiquidGlassCard>
    <template #header>
      <h3>{{ user.name }}</h3>
    </template>
    
    <div class="profile-content">
      <img :src="user.avatar" />
      <p>{{ user.bio }}</p>
    </div>
    
    <template #footer>
      <LiquidGlassButton @click="editProfile">
        编辑
      </LiquidGlassButton>
    </template>
  </LiquidGlassCard>
</template>

<script setup>
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()
const user = computed(() => userStore.currentUser)

const editProfile = () => {
  // 业务逻辑
}
</script>
```

---

#### 3. 容器组件（Container Components）

**定义**: 管理状态和数据流的容器

```
命名规范: [功能][Container/View/Page]
示例:
- UserDashboardView
- ProductManagementContainer
- OrderListPage
```

**特征**:
- ✅ 处理数据获取和状态管理
- ✅ 协调多个子组件的交互
- ✅ 连接业务逻辑和展示组件
- ✅ 通常对应路由页面

**示例**:
```vue
<!-- UserDashboardView.vue -->
<template>
  <div class="dashboard">
    <UserProfileCard />
    
    <div class="stats-grid">
      <StatCard
        v-for="stat in stats"
        :key="stat.id"
        :data="stat"
      />
    </div>
    
    <RecentActivityList :activities="activities" />
  </div>
</template>

<script setup>
import { onMounted } from 'vue'
import { useUserStore } from '@/stores/user'
import { useStatsStore } from '@/stores/stats'

const userStore = useUserStore()
const statsStore = useStatsStore()

const stats = computed(() => statsStore.userStats)
const activities = computed(() => userStore.recentActivities)

onMounted(async () => {
  await userStore.fetchUserData()
  await statsStore.fetchStats()
})
</script>
```

---

### 组件拆分策略 ⭐⭐⭐⭐

#### 1. 按复杂度拆分

**规则**: 当组件超过 200 行代码时进行拆分

```vue
<!-- ❌ 拆分前：一个大组件 (300+ 行) -->
<LargeFormComponent>
  <!-- 表单头部 -->
  <!-- 多个表单字段 -->
  <!-- 验证逻辑 -->
  <!-- 提交按钮 -->
</LargeFormComponent>

<!-- ✅ 拆分后：多个小组件 -->
<FormContainer>
  <FormHeader :title="formTitle" />
  
  <FormFieldGroup
    v-for="group in fieldGroups"
    :key="group.id"
    :fields="group.fields"
  />
  
  <FormActions
    @submit="handleSubmit"
    @cancel="handleCancel"
  />
</FormContainer>
```

---

#### 2. 按数据流拆分

**规则**: 根据数据的来源和流向划分

**智能组件**（管理状态）:
```vue
<script setup>
// 调用 API，管理状态
const { data, loading, error } = useAsyncData(fetchUsers)
</script>
```

**展示组件**（接收 props）:
```vue
<script setup>
// 只接收数据，不管理状态
defineProps({
  users: Array,
  loading: Boolean
})
</script>
```

---

#### 3. 按生命周期拆分

**规则**: 将不同生命周期的逻辑分离到 composables

```javascript
// ✅ 正确：按生命周期拆分逻辑
export default {
  setup() {
    // 数据获取
    const { data, loading } = useDataFetching()
    
    // 表单验证
    const { errors, validate } = useFormValidation()
    
    // 动画效果
    const { animationClass } = useAnimations()
    
    return {
      data,
      loading,
      errors,
      validate,
      animationClass
    }
  }
}
```

---

### 命名和组织规范 ⭐⭐⭐⭐⭐

#### 1. 组件命名规范

```javascript
// ✅ 正确：多词组件名
LiquidGlassButton.vue
UserProfileCard.vue
DateRangePicker.vue

// ❌ 错误：单词组件名
Button.vue
Card.vue
Picker.vue
```

#### 2. 父子组件命名

```
<!-- 紧密耦合的组件使用前缀 -->
SearchSidebar.vue
SearchSidebarNavigation.vue
SearchSidebarNavigationLink.vue

<!-- 或使用文件夹组织 -->
SearchSidebar/
  ├── index.vue
  ├── Navigation.vue
  └── NavigationLink.vue
```

#### 3. 文件结构组织

```
src/
├── components/
│   ├── base/              # 基础液态玻璃组件
│   │   ├── LiquidGlassButton.vue
│   │   ├── LiquidGlassCard.vue
│   │   └── LiquidGlassInput.vue
│   │
│   ├── business/          # 业务组件
│   │   ├── UserProfileCard.vue
│   │   └── ProductListItem.vue
│   │
│   └── layout/            # 布局组件
│       ├── AppHeader.vue
│       └── AppSidebar.vue
│
├── composables/           # 可复用逻辑
│   ├── useLiquidGlass.js
│   ├── useAnimation.js
│   └── useInteraction.js
│
├── views/                 # 页面组件
│   ├── UserDashboard.vue
│   └── ProductList.vue
│
└── utils/                 # 工具函数
    └── filterHelpers.js
```

---

### Props 和 Events 设计规范 ⭐⭐⭐⭐⭐

#### 1. Props 定义标准

```javascript
// ✅ 正确：详细的 prop 定义
defineProps({
  // 基础类型
  title: {
    type: String,
    required: true
  },
  
  // 带默认值
  size: {
    type: String,
    default: 'medium'
  },
  
  // 带验证器
  variant: {
    type: String,
    default: 'primary',
    validator: v => ['primary', 'secondary', 'danger'].includes(v)
  },
  
  // 复杂类型
  user: {
    type: Object,
    default: () => ({})
  },
  
  // 数组类型
  items: {
    type: Array,
    default: () => []
  }
})
```

#### 2. Events 定义标准

```javascript
// ✅ 正确：清晰的事件命名
const emit = defineEmits([
  'update:modelValue',  // v-model 支持
  'submit',             // 提交事件
  'cancel',             // 取消事件
  'item:select',        // 命名空间事件
  'item:delete'
])

// 发射事件时携带数据
const handleSubmit = (data) => {
  emit('submit', {
    timestamp: Date.now(),
    data
  })
}
```

#### 3. v-model 支持标准

```vue
<!-- 组件内部 -->
<script setup>
const props = defineProps({
  modelValue: [String, Number, Date]
})

const emit = defineEmits(['update:modelValue'])

const updateValue = (newValue) => {
  emit('update:modelValue', newValue)
}
</script>

<!-- 使用 -->
<LiquidGlassDatePicker v-model="selectedDate" />
```

---

### 插槽设计规范 ⭐⭐⭐⭐

#### 1. 具名插槽标准

```vue
<template>
  <div class="card">
    <!-- 头部插槽 -->
    <div v-if="$slots.header" class="card-header">
      <slot name="header" />
    </div>
    
    <!-- 默认插槽 -->
    <div class="card-body">
      <slot />
    </div>
    
    <!-- 底部插槽 -->
    <div v-if="$slots.footer" class="card-footer">
      <slot name="footer" />
    </div>
  </div>
</template>
```

#### 2. 作用域插槽标准

```vue
<!-- 组件内部 -->
<template>
  <ul>
    <li
      v-for="item in items"
      :key="item.id"
    >
      <slot
        name="item"
        :item="item"
        :index="index"
      >
        <!-- 默认内容 -->
        {{ item.name }}
      </slot>
    </li>
  </ul>
</template>

<!-- 使用 -->
<ListComponent :items="users">
  <template #item="{ item, index }">
    <strong>{{ index + 1 }}.</strong>
    {{ item.name }}
  </template>
</ListComponent>
```

---

### Composables 设计规范 ⭐⭐⭐⭐⭐

#### 1. Composable 命名规范

```
命名: use[功能名]
示例:
- useLiquidGlass
- useAnimation
- useFormValidation
- useAsyncData
```

#### 2. Composable 标准结构

```javascript
// composables/useLiquidGlass.js
import { ref, computed, onMounted, onUnmounted } from 'vue'

export function useLiquidGlass(options = {}) {
  // 1. 响应式状态
  const filterId = ref(`glass-distortion-${uniqueId()}`)
  const isActive = ref(false)
  
  // 2. 计算属性
  const filterUrl = computed(() => `url(#${filterId.value})`)
  
  // 3. 方法
  const activate = () => {
    isActive.value = true
  }
  
  const deactivate = () => {
    isActive.value = false
  }
  
  // 4. 生命周期
  onMounted(() => {
    // 初始化逻辑
  })
  
  onUnmounted(() => {
    // 清理逻辑
  })
  
  // 5. 返回值
  return {
    // 状态
    filterId,
    isActive,
    
    // 计算属性
    filterUrl,
    
    // 方法
    activate,
    deactivate
  }
}
```

#### 3. Composable 组合使用

```javascript
// 在组件中组合多个 composables
export default {
  setup() {
    // 液态玻璃效果
    const { filterId, filterUrl } = useLiquidGlass()
    
    // 动画效果
    const { jellyAnimation, startAnimation } = useAnimation()
    
    // 交互处理
    const { isHovered, handleHover } = useInteraction()
    
    // 组合逻辑
    const handleClick = () => {
      startAnimation()
      // 其他逻辑
    }
    
    return {
      filterId,
      filterUrl,
      jellyAnimation,
      isHovered,
      handleHover,
      handleClick
    }
  }
}
```

---

### Provide/Inject 使用规范 ⭐⭐⭐⭐

#### 1. 使用场景

**适用**: 跨多层级组件传递数据，避免 props drilling

```vue
<!-- ✅ 适用场景：主题配置 -->
<!-- App.vue -->
<script setup>
import { provide } from 'vue'

const theme = {
  glassOpacity: 0.25,
  primaryColor: '#007bff'
}

provide('theme', theme)
</script>

<!-- 深层子组件 -->
<script setup>
import { inject } from 'vue'

const theme = inject('theme')
</script>
```

**不适用**: 简单的父子通信

```vue
<!-- ❌ 错误：简单父子通信用 provide/inject -->
<!-- 应该用 props -->
<ParentComponent>
  <ChildComponent :data="data" />
</ParentComponent>
```

---

### 组件质量检查清单 ⭐⭐⭐⭐⭐

#### 好组件的特征

- [ ] **可预测**: 相同输入产生相同输出
- [ ] **可复用**: 可在不同上下文中使用
- [ ] **可测试**: 逻辑清晰，便于编写测试
- [ ] **可维护**: 代码结构清晰，易于理解和修改
- [ ] **单一职责**: 只负责一个明确的功能
- [ ] **松耦合**: 与其他组件依赖最小化
- [ ] **高内聚**: 内部逻辑紧密相关

#### 避免的反模式

- [ ] ❌ 组件过大（超过 200 行）
- [ ] ❌ 深层次的 prop 传递（超过 3 层）
- [ ] ❌ 直接修改 props
- [ ] ❌ 在模板中使用复杂逻辑
- [ ] ❌ 缺乏 key 的 v-for 循环
- [ ] ❌ 直接访问 $parent 或 $refs
- [ ] ❌ 全局状态滥用

---

### 现代 Vue3 设计模式总结

#### 1. 容器/展示模式

```vue
<!-- 容器组件：处理逻辑 -->
<script setup>
const { users, loading, fetchUsers } = useUsers()
const { validate } = useValidation()
</script>

<template>
  <UserList
    :users="users"
    :loading="loading"
    @validate="validate"
  />
</template>

<!-- 展示组件：纯 UI -->
<script setup>
defineProps({
  users: Array,
  loading: Boolean
})
</script>
```

#### 2. 组合模式

```javascript
// 将多个 composables 组合使用
export function useUserManagement() {
  const { data } = useUserData()
  const { validate } = useValidation()
  const { save } = usePersistence()
  const { notify } = useNotification()
  
  const submitUser = async (userData) => {
    if (!validate(userData)) return
    
    await save(userData)
    notify('用户保存成功')
  }
  
  return {
    data,
    validate,
    submitUser
  }
}
```

---

## ✅ 规范检查清单

### 新组件开发前检查

#### 液态玻璃效果
- [ ] 采用双层容器结构
- [ ] 外层容器无 padding
- [ ] 四层只用 inset: 0
- [ ] 使用标准 SVG 滤镜
- [ ] 实现果冻弹性效果
- [ ] 不重复定义 box-shadow
- [ ] 添加 cursor: pointer
- [ ] 圆角联动系统
- [ ] 使用标准贝塞尔曲线
- [ ] 添加详细注释

#### Vue3 组件设计
- [ ] 遵循单一职责原则
- [ ] 使用 Composition API
- [ ] 组件分类正确（base/business/container）
- [ ] 组件代码少于 200 行
- [ ] Props 定义详细且有验证
- [ ] Events 命名清晰
- [ ] 支持 v-model（如需要）
- [ ] 使用具名插槽
- [ ] 逻辑抽离到 composables
- [ ] 响应式设计
- [ ] 避免反模式

### 问题排查清单

#### 边缘混乱？
- [ ] 外层容器是否有 padding？
- [ ] 各层是否添加了额外属性？
- [ ] SVG 滤镜参数是否正确？

#### 果冻效果不明显？
- [ ] padding 是否增大？
- [ ] border-radius 是否增大？
- [ ] 是否使用标准贝塞尔曲线？
- [ ] transition 是否添加了 padding 和 border-radius？

#### 圆角不协调？
- [ ] 内外层圆角比例是否合理？
- [ ] 是否使用圆角联动？
- [ ] 悬停时圆角是否同步增大？

#### 组件设计问题？
- [ ] 组件是否职责单一？
- [ ] 是否有深层 props 传递？
- [ ] 逻辑是否可以抽离到 composables？
- [ ] 是否有重复代码可以复用？

---

## 📚 参考组件对照表

| 组件 | 视觉效果 | 组件设计 | 说明 |
|------|---------|---------|------|
| LiquidGlassDock | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 标准组件，完全符合所有规范 |
| LiquidGlassTabBar.N01 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 修复后，完全符合规范 |
| LiquidGlassTabBar | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 基本符合，需调整悬停效果 |
| LiquidGlassToggle | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 特殊组件，部分规范不适用 |
| MenuCard | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 完全符合规范 |
| GlassCard | ⭐⭐⭐ | ⭐⭐⭐ | 使用简化滤镜，需完善 |
| CircleGlassButton | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 基本符合 |
| LiquidGlassDatePicker | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 新组件，遵循规范开发 |

---

## 🎓 设计哲学

### 核心理念

1. **简洁至上**: 不添加不必要的属性
2. **职责分离**: 外层效果，内层内容；容器逻辑，展示 UI
3. **遵循标准**: 参考成功的 Dock 组件
4. **一致性**: 所有组件使用统一的参数和设计模式
5. **验证优先**: 新组件必须与 Dock 对比验证
6. **组合优于继承**: 通过 composables 组合功能
7. **可复用性**: 基础组件可在整个应用复用

### 开发流程

```
1. 确定组件分类（base/business/container）
2. 复制对应的标准模板
3. 定义 Props 和 Events
4. 实现液态玻璃四层结构
5. 添加果冻弹性效果
6. 抽离逻辑到 composables
7. 添加插槽支持
8. 与标准组件对比验证
9. 边缘必须完美
10. 代码审查和测试
```

---

## 🚨 违规处理

### 发现不符合规范的组件

1. **立即标记**: 在代码中添加 `TODO: 不符合规范`
2. **创建日志**: 记录问题和修复计划
3. **对比标准**: 找出具体差异（视觉 + 设计）
4. **逐项修复**: 按规范逐条修复
5. **验证效果**: 边缘必须与 Dock 一样完美
6. **代码审查**: 确保符合 Vue3 设计原则

---

## 📖 规范版本

- **版本**: 2.0.0
- **制定时间**: 2025-10-14 (更新)
- **基准组件**: LiquidGlassDock.vue
- **适用范围**: 所有液态玻璃系列组件
- **设计框架**: Vue3 Composition API
- **执行要求**: 🚨 强制遵守

---

**规范制定完成！** ✅  
**执行要求**: 所有新组件和现有组件都必须遵循本规范  
**双重标准**: 视觉效果 + 组件设计同等重要