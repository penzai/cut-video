<template>
  <div ref="root" class="time-range-bar trb-wrapper">
    <div
      class="content"
      :style="{
        left: `${contentLeft}px`,
        right: `${contentRight}px`
      }"
    ></div>
    <div
      ref="left"
      class="left-edge"
      :style="{
        left: `${contentLeft - 7}px`
      }"
    ></div>
    <div
      ref="right"
      class="right-edge"
      :style="{
        right: `${contentRight - 7}px`
      }"
    ></div>
  </div>
</template>

<script>
export default {
  props: {
    width: {
      type: Number,
      default: 800
    }
  },
  data() {
    return {
      rootStart: 0, //容器开始x
      rootEnd: 0, //容器终止x
      contentLeft: 0, //内容条左向距离
      contentRight: 0 //内容条右向距离
    };
  },
  watch: {
    contentLeft() {
      this.$emit("start-change", this.contentLeft / this.width);
    },
    contentRight() {
      this.$emit("end-change", (this.width - this.contentRight) / this.width);
    }
  },
  mounted() {
    const $root = this.$refs.root;
    this.rootStart = $root.getBoundingClientRect().x;
    this.rootEnd = this.rootStart + this.width;
    this.bindLeftEvent();
    this.bindRightEvent();
  },
  methods: {
    // 绑定左滑块
    bindLeftEvent() {
      const $left = this.$refs.left;
      const docMoveFunc = e => {
        window.requestAnimationFrame(() => {
          this.$emit("start-drag-callback"); //暂停视频播放，移除循环播放事件
          const x = e.pageX;
          const len = x - this.rootStart;
          this.contentLeft = len;
        });
      };
      $left.addEventListener("mousedown", () => {
        window.document.addEventListener("mousemove", docMoveFunc);
        window.document.addEventListener(
          "mouseup",
          () => window.document.removeEventListener("mousemove", docMoveFunc),
          {
            once: true
          }
        );
      });
    },
    // 绑定右滑块
    bindRightEvent() {
      const $right = this.$refs.right;
      const docMoveFunc = e => {
        window.requestAnimationFrame(() => {
          this.$emit("start-drag-callback"); //暂停视频播放，移除循环播放事件
          const x = e.pageX;
          const len = this.rootEnd - x;
          this.contentRight = len;
        });
      };
      $right.addEventListener("mousedown", () => {
        window.document.addEventListener("mousemove", docMoveFunc);
        window.document.addEventListener(
          "mouseup",
          () => window.document.removeEventListener("mousemove", docMoveFunc),
          {
            once: true
          }
        );
      });
    }
  }
};
</script>

<style lang="less" scoped>
@theme-color: #2e7fdd;
@bar-height: 40px;
.trb-wrapper {
  position: relative;
  width: 100%;
  height: @bar-height;
  background: #f7f7f7;
  .left-edge,
  .right-edge {
    position: absolute;
    width: 14px;
    height: 100%;
    background: @theme-color;
    cursor: ew-resize;
    &::before,
    &::after {
      display: inline-block;
      content: "";
      position: absolute;
      top: 50%;
      -webkit-transform: translateY(-50%);
      transform: translateY(-50%);
      width: 1px;
      height: 10px;
      background: #fff;
    }
    &::before {
      left: 5px;
    }
    &::after {
      left: 8px;
    }
  }
  .left-edge {
    border-top-left-radius: 4px;
    border-bottom-left-radius: 4px;
    // left: -7px;
  }
  .right-edge {
    border-top-right-radius: 4px;
    border-bottom-right-radius: 4px;
    // left: calc(~"100% - 7px");
  }
  .content {
    position: absolute;
    box-sizing: border-box;
    // width: 400px;
    height: @bar-height;
    border-top: 4px solid @theme-color;
    border-bottom: 4px solid @theme-color;
    background: transparent;
    // cursor: move;
    // left: 0;
    // right: 0;
  }
}
</style>
