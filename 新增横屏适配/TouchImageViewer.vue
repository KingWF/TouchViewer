<template>
  <div class="container" :class="{ landscape: isLandscape }">
    <div class="image-gallery"  v-show="galleryStyle">
      <div
        class="thumb"
        v-for="(img, index) in images"
        :key="index"
        @click="selectImage(img)"
      >
        <img :src="img" :alt="'图片' + (index + 1)" />
      </div>
    </div>
    <!-- <div

      style="
        height: 100px;
        width: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
      "
      
    >
      <div>偏移x{{ posX }}</div>
      <div>偏移y{{ posY }}</div> -->
      <!-- <div>边界检测：{{ msg }}</div>
      <div>{{ landscape }}</div>
    </div> -->
    <div
      class="image-viewer"
      @touchstart.prevent="onTouchStart"
      @touchmove.prevent="onTouchMove"
      @touchend.prevent="onTouchEnd"
      :style="viewerStyle"
    >
      <div class="image-container" :style="transformStyle">
        <img :src="currentImage" alt="查看图片" />
      </div>
      <div class="zoom-indicator">缩放: {{ Math.round(scale * 100) }}%</div>
    </div>

    <div class="controls" :style="controlsStyle">
      <button class="btn" @click="zoomIn" ><i>+</i> 放大</button>
      <button class="btn" @click="zoomOut" ><i>-</i> 缩小</button>
      <button class="btn" @click="reset" >重置</button>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      images: [
        "https://images.unsplash.com/photo-1501854140801-50d01698950b?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1200&q=80",
        "https://images.unsplash.com/photo-1475924156734-496f6cac6ec1?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1200&q=80",
        "https://images.unsplash.com/photo-1433086966358-54859d0ed716?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1200&q=80",
        "https://images.unsplash.com/photo-1476820865390-c52aeebb9891?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1200&q=80",
      ],
      currentImage: "",
      scale: 1,
      minScale: 0.5,
      maxScale: 4,
      posX: 0,
      posY: 0,
      startX: 0,
      startY: 0,
      initialDistance: null,
      isDragging: false,
      isPinching: false,
      msg: 0,
      isLandscape: false,
    };
  },
  computed: {
    transformStyle() {
      return {
        transform: `translate(${this.posX}px, ${this.posY}px) scale(${this.scale})`,
      };
    },
    galleryStyle() {
      // 横屏时更改缩略图布局
      return !this.isLandscape
    },
    viewerStyle(){
      return this.isLandscape?{
        height:"100vh"
      }:{}
    },  
    controlsStyle() {
      // 横屏时控制按钮位置调整
      return this.isLandscape
        ? {
            position: "fixed",
            left: "10px",
            top: "50%",
            transform: "translateY(-50%)",
            "display":"flex",
            "flex-direction": "column",
            width: "20vw",
            height:"100vh",
            "border-radius": "25px",
            padding: "10px",
            background: "rgba(0, 0, 0, 0.5)",
          }
        : {};
    },
  },
  mounted() {
    // 默认加载第一张图片
    if (this.images.length > 0) {
      this.currentImage = this.images[0];
    }
    // 初始检测屏幕方向
    this.checkOrientation();

    // 监听屏幕方向变化
    window.addEventListener("resize", this.checkOrientation);
    window.addEventListener("orientationchange", this.handleOrientationChange);

    this.$nextTick(this.centerImage);
  },
  methods: {
    checkOrientation() {
      // 通过宽高比判断横竖屏
      this.isLandscape = window.innerWidth > window.innerHeight;
    },
    handleOrientationChange() {
      // 屏幕方向变化时重新调整布局
      setTimeout(() => {
        this.checkOrientation();
        this.reset();

        // 如果是横屏且图片已加载，重新设置位置
        if (this.isLandscape && this.imageLoaded) {
          this.$nextTick(this.centerImage);
        }
      }, 300);
    },
    selectImage(img) {
      this.currentImage = img;
      this.reset();
    },
    centerImage() {
      const viewer = document.querySelector(".image-viewer");
      if (!viewer) return;

      const viewerRect = viewer.getBoundingClientRect();
      this.posX = (viewerRect.width - viewerRect.width) / 2;
      this.posY = (viewerRect.height - viewerRect.height) / 2;
    },
    onTouchStart(e) {
      if (e.touches.length === 1) {
        // 单指触摸
        this.isDragging = true;
        this.startX = e.touches[0].clientX - this.posX;
        this.startY = e.touches[0].clientY - this.posY;
      } else if (e.touches.length === 2) {
        // 双指触摸
        this.isPinching = true;
        this.initialDistance = this.getDistance(e.touches[0], e.touches[1]);
      }
    },
    onTouchMove(e) {
      if (this.isPinching && e.touches.length === 2) {
        // 双指缩放
        const currentDistance = this.getDistance(e.touches[0], e.touches[1]);
        const scaleChange = currentDistance / this.initialDistance;

        const newScale = this.scale * scaleChange;
        if (newScale >= 4) {
          return;
        }
        this.scale = Math.max(this.minScale, Math.min(this.maxScale, newScale));

        // 调整位置以保持缩放中心
        const viewer = document.querySelector(".image-viewer");
        if (!viewer) return;

        const viewerRect = viewer.getBoundingClientRect();
        const midpoint = this.getMidpoint(e.touches[0], e.touches[1]);

        const viewerCenterX = viewerRect.width / 2;
        const viewerCenterY = viewerRect.height / 2;

        const offsetX =
          (midpoint.x - viewerRect.left - viewerCenterX - this.posX) *
          (scaleChange - 1);
        const offsetY =
          (midpoint.y - viewerRect.top - viewerCenterY - this.posY) *
          (scaleChange - 1);

        this.posX -= offsetX;
        this.posY -= offsetY;

        this.initialDistance = currentDistance;
      } else if (this.isDragging && e.touches.length === 1) {
        // 单指拖动
        const touch = e.touches[0];
        this.posX = touch.clientX - this.startX;
        this.posY = touch.clientY - this.startY;
      }
    },
    onTouchEnd() {
      this.isDragging = false;
      this.isPinching = false;
      this.initialDistance = null;
      this.checkBoundaries();
    },
    getDistance(touch1, touch2) {
      const dx = touch2.clientX - touch1.clientX;
      const dy = touch2.clientY - touch1.clientY;
      return Math.sqrt(dx * dx + dy * dy);
    },
    // 获取触摸中心点
    getMidpoint(touch1, touch2) {
      return {
        x: (touch1.clientX + touch2.clientX) / 2,
        y: (touch1.clientY + touch2.clientY) / 2,
      };
    },
    // 边界检查
    checkBoundaries() {
      const viewer = document.querySelector(".image-viewer");
      if (!viewer) return;

      const viewerRect = viewer.getBoundingClientRect();
      const img = viewer.querySelector("img");
      if (!img) return;

      const imgRect = img.getBoundingClientRect();
      // 获取原始尺寸而非当前视图尺寸
      const naturalWidth = imgRect.width;
      const naturalHeight = imgRect.height;

      // 计算缩放后的实际尺寸
      const scaledWidth = naturalWidth * this.scale;
      const scaledHeight = naturalHeight * this.scale;

      // 计算边界位置 - X轴
      let minX = 0;
      let maxX = 0;

      // 计算边界位置 - Y轴
      let minY = 0;
      let maxY = 0;
      this.msg = scaledWidth;
      if (scaledWidth > viewerRect.width) {
        // 图片宽度大于容器 - 允许左右移动
        minX = -(scaledWidth - viewerRect.width) / 2;
        maxX = (scaledWidth - viewerRect.width) / 2;
      } else {
        // 图片宽度小于容器 - 居中显示
        this.posX = 0;
      }

      if (scaledHeight > viewerRect.height) {
        // 图片高度大于容器 - 允许上下移动
        minY = -(scaledHeight - viewerRect.height) / 2;
        maxY = (scaledHeight - viewerRect.height) / 2;
      } else {
        // 图片高度小于容器 - 居中显示
        this.posY = 0;
      }

      // 应用边界约束
      this.posX = Math.max(
        minX / this.scale,
        Math.min(maxX / this.scale, this.posX)
      );
      this.posY = Math.max(
        minY / this.scale,
        Math.min(maxY / this.scale, this.posY)
      );

      // 特殊情况：当放大倍数较大时，完全居中
      //   if (this.scale > 2) {
      //     this.posX = 0;
      //     this.posY = 0;
      //   }
    },
    zoomIn() {
      this.scale = Math.min(this.maxScale, this.scale + 0.3);
      this.checkBoundaries();
    },
    zoomOut() {
      this.scale = Math.max(this.minScale, this.scale - 0.3);
      this.checkBoundaries();
    },
    reset() {
      this.scale = 1;
      this.posX = 0;
      this.posY = 0;
      this.$nextTick(this.centerImage);
    },
  },
  beforeDestroy() {
    // 移除事件监听器
    window.removeEventListener("resize", this.checkOrientation);
    window.removeEventListener(
      "orientationchange",
      this.handleOrientationChange
    );
  },
};
</script>
<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
}

h1 {
  font-size: 2.2rem;
  margin-bottom: 10px;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.subtitle {
  font-size: 1.1rem;
  opacity: 0.9;
  max-width: 90%;
  margin: 0 auto;
  line-height: 1.6;
}

.image-gallery {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(110px, 1fr));
  gap: 12px;
  margin-bottom: 30px;
}

.thumb {
  height: 100px;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
  cursor: pointer;
  transition: all 0.3s ease;
  opacity: 0.85;
}

.thumb:hover {
  opacity: 1;
  transform: scale(1.05);
}

.thumb img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.image-viewer {
  position: relative;
  height: 65vh;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  border-radius: 25px;
  background: rgba(0, 0, 0, 0.15);
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.4);
  margin-bottom: 20px;
}

.image-container {
  position: relative;
  transition: transform 0.15s ease-out;
  will-change: transform;
}

.image-container img {
  display: block;
  max-width: 100%;
  max-height: 65vh;
  border-radius: 15px;
  box-shadow: 0 12px 35px rgba(0, 0, 0, 0.5);
  user-select: none;
  -webkit-user-drag: none;
}

.zoom-indicator {
  position: absolute;
  top: 15px;
  left: 15px;
  background: rgba(0, 0, 0, 0.5);
  color: white;
  padding: 10px 20px;
  border-radius: 25px;
  font-size: 1rem;
  z-index: 10;
  backdrop-filter: blur(5px);
}

.controls {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  box-sizing: border-box;
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 15px;
  padding: 15px;
  background: rgba(0, 0, 0, 0.3);
  border-top-left-radius: 20px;
  border-top-right-radius: 20px;
}

.btn {
  padding: 14px 28px;
  background: rgba(255, 255, 255, 0.15);
  border: 2px solid rgba(255, 255, 255, 0.3);
  color: white;
  border-radius: 50px;
  font-size: 1.1rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  gap: 8px;
}

.btn i {
  font-size: 1.3rem;
}

.btn:active {
  background: rgba(255, 255, 255, 0.3);
  transform: scale(0.95);
}

.instructions {
  text-align: center;
  padding: 20px;
  font-size: 1rem;
  opacity: 0.85;
  margin-top: 10px;
  background: rgba(0, 0, 0, 0.2);
  border-radius: 15px;
}

.feature-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
  margin-top: 30px;
}

.feature-card {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 15px;
  padding: 20px;
  text-align: center;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

.feature-icon {
  font-size: 2.5rem;
  margin-bottom: 15px;
  color: #ffd700;
}

.feature-title {
  font-size: 1.3rem;
  margin-bottom: 10px;
  font-weight: 600;
}

.feature-desc {
  font-size: 0.95rem;
  opacity: 0.9;
}

@media (max-width: 768px) {
  h1 {
    font-size: 1.8rem;
  }

  .subtitle {
    font-size: 1rem;
  }

  .image-gallery {
    grid-template-columns: repeat(auto-fill, minmax(90px, 1fr));
  }

  .thumb {
    height: 85px;
  }

  .btn {
    padding: 12px 20px;
    font-size: 1rem;
  }

  .feature-list {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 480px) {
  h1 {
    font-size: 1.5rem;
  }

  .subtitle {
    font-size: 0.9rem;
  }

  .btn {
    padding: 10px 16px;
    font-size: 0.9rem;
  }

  .image-gallery {
    grid-template-columns: repeat(auto-fill, minmax(70px, 1fr));
  }

  .thumb {
    height: 70px;
  }
}
</style>
