<template>
  <div class="canvas-wrap">
    <div v-show="!previewImage" ref="canvasRef" class="canvas-wrap-box">
      <canvas
        id="canvas-map"
        ref="canvasMapRef"
        width="100"
        height="100"
      ></canvas>
    </div>
    <div class="preview-wrap" v-show="previewImage">
      <img class="preview-image" :src="previewImage" alt="生成预览" />
    </div>
    <div class="btn-box">
      <span class="clear-btn" @click="goClear">清除</span>
      <span class="sure-btn" @click="goSure">确认</span>
    </div>
  </div>
</template>

<script>
import SignaturePad from "signature_pad";
import { rotateBase64Img } from "@/utils/common";

export default {
  name: "SignNameCanvas",
  data() {
    return {
      canvasNode: null,
      previewImage: null,
    };
  },
  mounted() {
    this.initalHandle();
    window.addEventListener("resize", this.initalHandle, false);
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.initalHandle, false);
  },
  methods: {
    initalHandle() {
      const _canvasBox = this.$refs.canvasRef;
      const _canvas = this.$refs.canvasMapRef;
      if (!_canvasBox || !_canvas) {
        return false;
      }

      _canvas.width = _canvasBox.clientWidth;
      _canvas.height = _canvasBox.clientHeight;

      this.goClear();
      this.canvasNode = new SignaturePad(_canvas, {
        minWidth: 2,
        maxWidth: 2,
        penColor: "rgb(0, 0, 0)",
      });
    },

    goClear() {
      if (this.canvasNode) {
        this.canvasNode.clear();
        this.previewImage = null;
      }
    },

    goSure() {
      const canvasNode = this.canvasNode;
      // 重新初始化画布
      if (!canvasNode) {
        this.initalHandle();
      }

      // 是否签字
      if (canvasNode.isEmpty()) {
        this.$toast("您还没有签名");
        return false;
      }

      // 图像旋转二次处理
      const _boxWidth = window.innerWidth;
      const _boxHeight = window.innerHeight;
      const _signImg = canvasNode.toDataURL("image/png", 0.6);
      if (_boxWidth < _boxHeight) {
        this.previewImage = _signImg;
      } else {
        rotateBase64Img(_signImg, -90, (imgUrlRes) => {
          this.previewImage = imgUrlRes;
        });
      }
    },
  },
};
</script>

<style lang='css' scoped>

.canvas-wrap .btn-box .clear-btn,
.canvas-wrap .btn-box .sure-btn {
  display: inline-block;
  width: 100px;
  height: 24px;
  margin: 0 10px;
  line-height: 24px;
  border-radius: 6px;
  background-color: #fff;
}

.canvas-wrap .btn-box .clear-btn {
  color: #898989;
	background: #d4d4d4;
}

.canvas-wrap .btn-box .sure-btn {
  color: #fff;
  background: linear-gradient(100deg, #ff4e01 0%, #ffbc01 100%);
}

.canvas-wrap .canvas-wrap-box {
  /* left: 22%; */
  box-sizing: border-box;
  width: 20vw;
  height: 80vh;
  border: 1px dashed #d4d4d4;
  overflow: hidden;
  background-color: #fff;
}

/* 竖屏 css */
@media screen and (orientation: portrait) {
  .canvas-wrap .canvas-wrap-box {
    width: 90%;
    height: 30vh;
    margin: 20px auto 0;
  }
  .preview-wrap {
    margin: 20px auto 0;
  }
  .canvas-wrap .btn-box {
    margin-top: 16px;
  }
}

/* 横屏 css */

@media screen and (orientation: landscape) {
  /* 横屏样式自己调试 */
}
</style>
