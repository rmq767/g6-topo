<template>
  <div class="pdf-view">
    <aside class="pdf-list" ref="pdf-list" :class="{ collapse: !showNav }">
      <div
        class="pdf-item"
        v-for="(url, index) in imgList"
        :key="index"
        @click="changeImage(url, index)"
      >
        <el-image :src="url" lazy fit="cover"></el-image>
      </div>
    </aside>
    <main class="pdf-content" id="pdf-content">
      <div class="handle-btn">
        <el-button type="primary" size="mini" @click="showNavigator">{{
          showNav ? "隐藏导航栏" : "展示导航栏"
        }}</el-button>
      </div>
      <div class="canvas-content">
        <canvas id="pdf-canvas"></canvas>
        <div class="tools">
          <el-button
            type="primary"
            size="mini"
            icon="el-icon-zoom-in"
            @click="toBig"
          ></el-button>
          <el-button
            type="primary"
            size="mini"
            icon="el-icon-zoom-out"
            @click="toSmall"
          ></el-button>
        </div>
      </div>
      <div class="jump">
        <div class="jump-btn">
          <el-button
            type="primary"
            size="mini"
            icon="el-icon-arrow-left"
            :disabled="activeImageIndex === 0"
            @click="preImage"
          ></el-button>
          <el-button
            type="primary"
            size="mini"
            icon="el-icon-arrow-right"
            :disabled="activeImageIndex === imgList.length"
            @click="nextImage"
          ></el-button>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
let ctx = null;
export default {
  name: "test",
  data() {
    return {
      imgList: [],
      activeImageIndex: 0,
      canvas: {
        width: 0,
        height: 0,
        padding: [20, 20, 20, 20],
        scale: 1,
      },
      showNav: true,
      observer: null,
    };
  },
  mounted() {
    this.getImage();
    this.initCanvas();
    const canvas = document.getElementById("pdf-canvas");
    canvas.addEventListener("contextmenu", this.disabledContextMenu, true);
  },
  beforeDestroy() {
    const canvas = document.getElementById("pdf-canvas");
    canvas.removeEventListener("contextmenu", this.disabledContextMenu);
  },
  methods: {
    getImage() {
      this.imgList = [
        "https://fuss10.elemecdn.com/a/3f/3302e58f9a181d2509f3dc0fa68b0jpeg.jpeg",
        "https://fuss10.elemecdn.com/1/34/19aa98b1fcb2781c4fba33d850549jpeg.jpeg",
        "https://fuss10.elemecdn.com/0/6f/e35ff375812e6b0020b6b4e8f9583jpeg.jpeg",
        "https://fuss10.elemecdn.com/9/bb/e27858e973f5d7d3904835f46abbdjpeg.jpeg",
        "https://fuss10.elemecdn.com/d/e6/c4d93a3805b3ce3f323f7974e6f78jpeg.jpeg",
        "https://fuss10.elemecdn.com/3/28/bbf893f792f03a54408b3b7a7ebf0jpeg.jpeg",
        "https://fuss10.elemecdn.com/2/11/6535bcfb26e4c79b48ddde44f4b6fjpeg.jpeg",
      ];
    },
    /**
     * @description 初始化canvas
     */
    initCanvas() {
      const canvasParent = document.querySelector(".pdf-view");
      const canvas = document.getElementById("pdf-canvas");
      canvas.width = canvasParent.offsetWidth;
      canvas.height = canvasParent.offsetHeight - 100 - 40;
      this.canvas.width = canvas.width;
      this.canvas.height = canvas.height;
      ctx = canvas.getContext("2d");
      this.drawImage();
    },
    /**
     * @description 画图片
     */
    drawImage(src, scale = this.canvas.scale) {
      const img = new Image();
      img.onload = () => {
        const imgWidth = img.width;
        const imgHeight = img.height;
        const startX = Math.floor(imgWidth / 2 - imgWidth / 2 / scale);
        const startY = Math.floor(imgHeight / 2 - imgHeight / 2 / scale);
        const endX = imgWidth - startX * 2;
        const endY = imgHeight - startY * 2;
        if (imgWidth <= this.canvas.width) {
          const left = (this.canvas.width - imgWidth) / 2;
          ctx.drawImage(
            img,
            startX, //中心往左上x距离
            startY, //中心往左上y距离
            endX, //中心往右下x距离
            endY, //中心往右下y距离
            left,
            this.canvas.padding[0],
            imgWidth,
            imgHeight - this.canvas.padding[3]
          );
        } else {
          const scale1 = this.canvas.width / imgWidth;
          ctx.drawImage(
            img,
            startX, //中心往左上x距离
            startY, //中心往左上y距离
            endX, //中心往右下x距离
            endY, //中心往右下y距离
            0,
            this.canvas.padding[0],
            (startX + endX) * scale1,
            (startY + endY) * scale1
          );
        }
      };
      if (src) {
        img.src = src;
      } else {
        img.src = this.imgList[this.activeImageIndex];
      }
    },
    /**
     * @description 选择图片展示
     */
    changeImage(src, index) {
      this.activeImageIndex = index;
      this.canvas.scale = 1;
      ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      this.drawImage(src);
    },
    /**
     * @description 展示侧边
     */
    showNavigator() {
      this.showNav = !this.showNav;
    },
    /**
     * @description 禁止右键菜单
     */
    disabledContextMenu(e) {
      e.preventDefault();
      return false;
    },
    /**
     * @description 放大
     */
    toBig() {
      this.canvas.scale += 0.5;
      ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      this.drawImage(this.imgList[this.activeImageIndex]);
    },
    /**
     * @description 缩小
     */
    toSmall() {
      this.canvas.scale -= 0.5;
      ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      this.drawImage(this.imgList[this.activeImageIndex]);
    },
    /**
     * @description 上一个图片
     */
    preImage() {
      if (this.activeImageIndex === 0) {
        return;
      } else {
        this.activeImageIndex--;
        this.changeImage(
          this.imgList[this.activeImageIndex],
          this.activeImageIndex
        );
      }
    },
    /**
     * @description 下一个图片
     */
    nextImage() {
      if (this.activeImageIndex === this.imgList.length) {
        return;
      } else {
        this.activeImageIndex++;
        this.changeImage(
          this.imgList[this.activeImageIndex],
          this.activeImageIndex
        );
      }
    },
  },
};
</script>

<style scoped lang="scss">
.pdf-view {
  display: flex;
  width: 100%;
  height: 100%;
  .pdf-list {
    width: 240px;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    padding: 20px 0;
    background: #000;
    box-sizing: border-box;
    transition: all 0.3s ease-in-out;
    &::-webkit-scrollbar {
      width: 0px;
    }
    .pdf-item {
      display: inline-flex;
      justify-content: center;
      align-items: center;
      cursor: pointer;
      &:hover {
        ::v-deep img {
          transition: all 0.5s ease-in-out;
          transform: scale(1.1);
        }
      }
      ::v-deep .el-image {
        pointer-events: none;
      }
    }
  }
  .collapse {
    width: 0;
  }
  .pdf-content {
    flex: 1;
    height: 100vh;
    background: #000;
    .handle-btn {
      height: 40px;
    }
    canvas {
      width: 100%;
    }
    .jump {
      height: 60px;
      padding-bottom: 20px;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .canvas-content {
      position: relative;
      .tools {
        position: absolute;
        bottom: 0;
        left: 0;
      }
    }
  }
}
.pdf-item .el-image {
  display: block;
  height: 200px;
  width: 200px;
  object-fit: cover;
  margin-bottom: 10px;
}
</style>
