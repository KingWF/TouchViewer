# 移动端友好的可缩放组件库

这个项目实现了一个移动端友好的图片查看器组件，支持手势缩放、平移和切换图片。特别适用于需要在微信小程序等环境中避免使用web-view时整体缩放HTML的问题。

## 演示地址
暂无
## 技术背景

在微信小程序中使用web-view嵌入H5页面时，面临两个主要问题：
1. **整体页面缩放**：用户双指操作会导致整个HTML内容缩放，而不是特定内容
2. **无法自定义缩放组件**：无法实现指定区域内单独缩放的效果

本组件完美解决了这些问题，通过：

1. 监听并阻止默认触摸行为
2. 独立实现组件内部缩放逻辑
3. 使用`transform: scale()`实现局部缩放

## 主要特性

### 手势交互
- **双指缩放**：支持基于中心的自然缩放体验
- **单指拖动**：可平滑平移放大后的内容
- **边界控制**：自动限制内容在可视区域内

### 功能控制
- **放大/缩小按钮**：精确控制缩放比例
- **重置功能**：一键恢复原始比例和位置
- **多内容切换**：轻松切换显示不同内容

### 自适应设计
- 响应式布局，适配不同屏幕尺寸
- 触摸事件处理兼容多种移动设备

## 使用场景

1. 移动端图片查看器
2. 电子书阅读器组件
3. 可缩放的地图或矢量图组件
4. 需要自定义缩放的复杂组件
5. 在微信小程序中嵌入H5页面时使用的自定义缩放元素

## 安装使用

### 1. 将组件添加到您的项目

```vue
<template>
  <TouchImageViewer :images="imageList" />
</template>


<script>
import TouchImageViewer from './components/TouchImageViewer.vue'

export default {
  components: {
    TouchImageViewer
  },
  data() {
    return {
      imageList: [
        'image1.jpg',
        'image2.jpg'
      ]
    }
  }
}
</script>
```
### 2. 替换为电子书内容

```vue
<div class="image-viewer">
  <!-- 替换此处内容为电子书或其他元素 -->
  <EbookContent :content="currentBookContent" />
</div>
```

## API 文档

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `images` | `Array` | `[]` | 图片URL数组 |
| `minScale` | `Number` | `0.5` | 最小缩放比例 |
| `maxScale` | `Number` | `4` | 最大缩放比例 |
| `content` | `Any` | `-` | 自定义内容（如电子书） |

### 方法

| 方法名 | 说明 |
|--------|------|
| `zoomIn()` | 放大内容 |
| `zoomOut()` | 缩小内容 |
| `reset()` | 重置缩放比例和位置 |
| `selectItem(item)` | 选择新内容 |

### 事件

| 事件名 | 说明 |
|--------|------|
| `scale-changed` | 缩放比例变化时触发 |
| `content-selected` | 切换内容时触发 |
## 工作原理
### 核心逻辑
```javascript
// 阻止默认触摸行为，避免整体缩放
@touchmove.prevent="onTouchMove"

// 单指拖动逻辑
startX = touch.clientX - this.posX
this.posX = touch.clientX - startX

// 双指缩放逻辑
const scaleChange = currentDistance / initialDistance
this.scale *= scaleChange

// 边界检测
const scaledWidth = naturalWidth * this.scale
if (scaledWidth > containerWidth) {
  minX = -(scaledWidth - containerWidth) / 2
}
自适应处理
.image-viewer {
  height: 65vh; /* 自适应高度 */
  overflow: hidden; /* 隐藏超出部分 */
}

.image-container {
  transform: translate(${posX}px, ${posY}px) scale(${scale});
}
```
## 在微信小程序中的应用

将组件构建为独立的H5页面
在小程序中使用<web-view>加载
通过URL参数传递初始数据
```vue
<!-- 小程序页面中 -->
<web-view src="https://your-domain.com/touch-viewer?images=img1,img2" />
```
## 贡献指南
欢迎任何形式的贡献！以下是参与步骤：

## Fork 本项目
创建新的特性分支 (git checkout -b feature/amazing-feature)
提交您的修改 (git commit -m 'Add some amazing feature')
推送至分支 (git push origin feature/amazing-feature)
创建 Pull Request
## 许可证
本项目采用 MIT 许可证

## 支持
如遇到任何问题，请提交 GitHub Issue
