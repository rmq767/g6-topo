<template>
  <div class="pdf-container" v-loading="loading">
    <input type="file" @change="getPdfFile" />
    <div class="handle-btn">
      <div class="left">
        <el-button type="primary" size="mini" @click="triggerCollapse">{{
          collapse ? "展开" : "收缩"
        }}</el-button>
      </div>
      <div class="right">
        <div class="pagination">
          <el-input-number
            v-model="currentPage"
            :min="1"
            :max="pageTotalNum"
            size="mini"
            :controls="false"
          ></el-input-number>
          /<span> {{ pageTotalNum || 0 }}</span>
        </div>
        <el-button
          type="primary"
          size="mini"
          icon="el-icon-arrow-left"
          @click="prePage"
          :disabled="currentPage === 1"
        ></el-button>
        <el-button
          type="primary"
          size="mini"
          icon="el-icon-arrow-right"
          @click="nextPage"
          :disabled="currentPage === pageTotalNum"
        ></el-button>
        <el-button
          type="primary"
          size="mini"
          icon="el-icon-zoom-in"
          @click="zoomIn"
        ></el-button>
        <el-button
          type="primary"
          size="mini"
          icon="el-icon-zoom-out"
          @click="zoomOut"
        ></el-button>
        <span class="scale">{{ scale }}%</span>
      </div>
    </div>
    <div class="pdf-viewer">
      <div class="pdf-list" :class="{ collapse: collapse }">
        <div
          class="pdf-item"
          :class="{ active: currentPage === index + 1 }"
          v-for="(url, index) in imgList"
          :key="index"
          @click="changePage(index + 1)"
        >
          <el-image :src="url">
            <!-- <div slot="placeholder" class="image-slot">
              加载中<span class="dot">...</span>
            </div> -->
          </el-image>
        </div>
      </div>
      <div class="pdf-main">
        <pdf
          ref="pdfRef"
          style="margin: 0 auto"
          :src="pdfData"
          :page="currentPage"
          @num-pages="pageTotalNum = $event"
        ></pdf>
      </div>
    </div>
  </div>
</template>

<script>
import pdf from "vue-pdf";
// var loadingTask = pdf.createLoadingTask(
//   "http://storage.xuetangx.com/public_assets/xuetangx/PDF/PlayerAPI_v1.0.6.pdf"
// );
export default {
  name: "pdfView",
  components: {
    pdf,
  },
  data() {
    return {
      loading: false, //加载
      pdfData: null, //url
      currentPage: 1, //当前页数
      pageTotalNum: 1, //总页数
      imgList: [], //base64图片列表
      collapse: false, //收缩
      scale: 100, //放大倍数
    };
  },
  watch: {
    /**
     * @description 监听当前页变化 滚动列表到顶部
     */
    currentPage(value) {
      this.$nextTick(() => {
        document.querySelector(".pdf-list").scrollTo({
          top: (value - 1) * 293,
          behavior: "smooth",
        });
      });
    },
  },
  mounted() {
    // this.loadPdf();
  },
  methods: {
    getPdfFile(e) {
      const file = e.target.files[0];
      const reader = new FileReader();
      reader.onload = (e) => {
        this.pdfData = pdf.createLoadingTask(e.target.result);
        this.loadPdf();
      };
      reader.readAsDataURL(file);
    },
    /**
     * @description 加载PDF获取canvas图片
     */
    loadPdf() {
      this.loading = true;
      this.pdfData.promise.then((pdf) => {
        this.pageTotalNum = pdf.numPages;
        let pagePromise = [];
        for (let index = 1; index <= this.pageTotalNum; index++) {
          const element = pdf.getPage(index);
          pagePromise.push(element);
        }
        this.getImageList(pagePromise);
      });
    },
    /**
     * @description 通过page 渲染canvas获取base64图片
     */
    getImageList(pagePromise) {
      let result = [];
      Promise.all(pagePromise)
        .then((res) => {
          for (let index = 0; index < res.length; index++) {
            // 当前页
            const page = res[index];
            // 绘制canvas
            const canvas = document.createElement("canvas");
            const context = canvas.getContext("2d");
            const viewport = page.getViewport({ scale: 1 });
            canvas.width = viewport.width;
            canvas.height = viewport.height;
            const renderContext = {
              canvasContext: context,
              viewport: viewport,
            };
            result.push(page.render(renderContext).promise);
            // canvas转图片
            Promise.all(result).then(() => {
              this.imgList.push(canvas.toDataURL());
              this.loading = false;
            });
          }
        })
        .catch(() => {
          this.loading = false;
        });
    },
    /**
     * @description 切换当前页
     */
    changePage(index) {
      this.currentPage = index;
    },
    /**
     * @description 切换侧边
     */
    triggerCollapse() {
      this.collapse = !this.collapse;
    },
    /**
     * @description 上一页
     */
    prePage() {
      let page = this.currentPage;
      if (page !== 1) {
        page = page > 1 ? page - 1 : this.pageTotalNum;
        this.currentPage = page;
      }
    },
    /**
     * @description 下一页
     */
    nextPage() {
      let page = this.currentPage;
      if (page !== this.pageTotalNum) {
        page = page < this.pageTotalNum ? page + 1 : 1;
        this.currentPage = page;
      }
    },
    /**
     * @description 放大
     */
    zoomIn() {
      this.scale += 25;
      const itemEl = this.$refs["pdfRef"].$el;
      itemEl.style.width = parseInt(this.scale) + "%";
    },
    /**
     * @description 缩小
     */
    zoomOut() {
      this.scale -= 25;
      const itemEl = this.$refs["pdfRef"].$el;
      itemEl.style.width = parseInt(this.scale) + "%";
    },
  },
};
</script>

<style lang="scss" scoped>
.pdf-container {
  width: 800px;
  height: 600px;
}
.pdf-viewer {
  width: 100%;
  height: calc(600px - 28px);
  position: relative;
  display: flex;
}
.pdf-list {
  width: 240px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  padding: 20px;
  background: #000;
  box-sizing: border-box;
  transition: all 0.3s ease-in-out;
  &::-webkit-scrollbar {
    width: 0px;
  }
  .pdf-item {
    height: 283px;
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
    &.active {
      border: 1px solid red;
    }
    &:not(:last-child) {
      margin-bottom: 10px;
    }
    ::v-deep .el-image {
      pointer-events: none;
      .image-slot {
        height: 283px;
      }
    }
  }
  &.collapse {
    // transform: scale(0);
    width: 0;
    padding: 0;
  }
}
.pdf-main {
  flex: 1;
  width: 100%;
  height: 100%;
  overflow-y: auto;
}
.handle-btn {
  background: #000;
  display: flex;
  color: #fff;
  font-size: 12px;
  .left {
    width: 240px;
  }
  .right {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    .pagination {
      display: flex;
      align-items: center;
      margin: 0 10px;
      font-size: 12px;
      ::v-deep .el-input__inner {
        width: 28px;
        height: 16px;
        line-height: 16px;
        padding: 0;
      }
      ::v-deep .el-input-number--mini {
        width: 28px;
      }
      span {
        display: flex;
        align-items: center;
        justify-content: center;
      }
    }
  }
}
</style>
