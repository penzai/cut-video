<template>
  <div ref="wrapper" class="time-range-bar trb-wrapper">
    <div class="video-frame">
      <!-- <div v-for="item in frameCurrentTimes" :key="item">
        <video
          preload="metadata"
          :currentTime="item"
          :src="dialog.url"
          width="100%"
          height="40"
          :oncanplay="`this.currentTime = ${item}`"
        ></video>
      </div> -->
    </div>
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
        left: `${contentLeft - sliderWidth / 2}px`
      }"
    ></div>
    <div
      ref="right"
      class="right-edge"
      :style="{
        right: `${contentRight - sliderWidth / 2}px`
      }"
    ></div>
  </div>
</template>

<script>
export default {
  inject: ["root", "dialog"],
  data() {
    return {
      sliderWidth: 14, //滑块宽度
      wrapperStart: 0, //容器开始x
      wrapperEnd: 0, //容器终止x
      contentLeft: 0, //内容条左向距离
      contentRight: 0, //内容条右向距离
      isDraging: false,
      width: 0,
      frameCurrentTimes: []
    };
  },
  watch: {
    contentLeft() {
      this.$emit("start-change", this.contentLeft / this.width);
    },
    contentRight() {
      this.$emit("end-change", (this.width - this.contentRight) / this.width);
    },
    "root.videoDuration"(val) {
      const count = 20;
      const gap = val / count;
      // 生成关键帧节点
      this.frameCurrentTimes = Array(count)
        .fill(1)
        .map((v, i) => {
          return i === count - 1 ? val : gap * i;
        });
    }
  },
  mounted() {
    this.bindLeftEvent();
    this.bindRightEvent();
    this.dialog.$once("init-default-data", () => {
      setTimeout(() => {
        const defaultData = this.dialog.defaultCutInfo;
        if (!defaultData) return;

        this.calulateWrapperRect();
        const { cutBeginTime, cutEndTime } = defaultData;
        const duration = this.root.videoDuration;
        this.contentRight = ((duration - cutEndTime) / duration) * this.width;
        this.contentLeft = (cutBeginTime / duration) * this.width;
      }, 1000);
    });
  },
  methods: {
    calulateWrapperRect() {
      const $wrapper = this.$refs.wrapper;
      const rect = $wrapper.getBoundingClientRect();
      this.width = rect.width;
      this.wrapperStart = rect.x;
      this.wrapperEnd = this.wrapperStart + this.width;
    },
    // 绑定左滑块
    bindLeftEvent() {
      const $left = this.$refs.left;
      const docMoveFunc = e => {
        window.requestAnimationFrame(() => {
          if (!this.isDraging) return; //已经停止拖动了，不再触发事件
          this.$emit("start-drag-callback"); //暂停视频播放，移除循环播放事件\

          // 获取wrapper位置
          if (!this.wrapperStart) {
            this.calulateWrapperRect();
          }

          const len = e.pageX - this.wrapperStart;
          const min = 0;
          const max = this.width - this.contentRight - this.sliderWidth;
          if (len < min) {
            this.contentLeft = min;
          } else if (len > max) {
            this.contentLeft = max;
          } else {
            this.contentLeft = len;
          }
        });
      };
      this.bindMoveEvent($left, docMoveFunc);
    },
    // 绑定右滑块
    bindRightEvent() {
      const $right = this.$refs.right;
      const docMoveFunc = e => {
        window.requestAnimationFrame(() => {
          if (!this.isDraging) return; //已经停止拖动了，不再触发事件
          this.$emit("start-drag-callback"); //暂停视频播放，移除循环播放事件

          // 获取wrapper位置
          if (!this.wrapperStart) {
            this.calulateWrapperRect();
          }

          const len = this.wrapperEnd - e.pageX;
          const min = 0;
          const max = this.width - this.contentLeft - this.sliderWidth;
          if (len < min) {
            this.contentRight = min;
          } else if (len > max) {
            this.contentRight = max;
          } else {
            this.contentRight = len;
          }
        });
      };
      this.bindMoveEvent($right, docMoveFunc);
    },
    bindMoveEvent(el, fn) {
      el.addEventListener("mousedown", () => {
        this.isDraging = true;
        window.document.addEventListener("mousemove", fn);
        window.document.addEventListener(
          "mouseup",
          () => {
            this.isDraging = false;
            window.document.removeEventListener("mousemove", fn);
          },
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
@theme-color: #2d8cf0;
@bar-height: 40px;
.trb-wrapper {
  position: relative;
  width: 100%;
  height: @bar-height;
  background: #f7f7f7
    url("data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBzdGFuZGFsb25lPSJubyI/PjwhRE9DVFlQRSBzdmcgUFVCTElDICItLy9XM0MvL0RURCBTVkcgMS4xLy9FTiIgImh0dHA6Ly93d3cudzMub3JnL0dyYXBoaWNzL1NWRy8xLjEvRFREL3N2ZzExLmR0ZCI+PHN2ZyB0PSIxNjAwNDE5NzE5MTI2IiBjbGFzcz0iaWNvbiIgdmlld0JveD0iMCAwIDQ1NTYgMTAyNCIgdmVyc2lvbj0iMS4xIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHAtaWQ9IjcxMjAiIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIiB3aWR0aD0iODg5Ljg0Mzc1IiBoZWlnaHQ9IjIwMCI+PGRlZnM+PHN0eWxlIHR5cGU9InRleHQvY3NzIj48L3N0eWxlPjwvZGVmcz48cGF0aCBkPSJNMTY2NCAwYzE0LjEzMTIgMCAyNS42IDYuOTYzMiAyNS42IDE1LjUxMzZ2OTkyLjk3MjhjMCA4LjU1MDQtMTEuNDY4OCAxNS41MTM2LTI1LjYgMTUuNTEzNnMtMjUuNi02Ljk2MzItMjUuNi0xNS41MTM2VjE1LjUxMzZjMC04LjU1MDQgMTEuNDY4OC0xNS41MTM2IDI1LjYtMTUuNTEzNnogbS00MDkuNiAxMDIuNGMxNC4xMzEyIDAgMjUuNiA2Ljg2MDggMjUuNiAxNS4zMDg4djgzOS43ODI0YzAgOC40NDgtMTEuNDY4OCAxNS4zMDg4LTI1LjYgMTUuMzA4OHMtMjUuNi02Ljg2MDgtMjUuNi0xNS4zMDg4VjExNy43MDg4YzAtOC40NDggMTEuNDY4OC0xNS4zMDg4IDI1LjYtMTUuMzA4OHogbTIwNC44IDBjMTQuMTMxMiAwIDI1LjYgNi44NjA4IDI1LjYgMTUuMzA4OHY4MzkuNzgyNGMwIDguNDQ4LTExLjQ2ODggMTUuMzA4OC0yNS42IDE1LjMwODhzLTI1LjYtNi44NjA4LTI1LjYtMTUuMzA4OFYxMTcuNzA4OGMwLTguNDQ4IDExLjQ2ODgtMTUuMzA4OCAyNS42LTE1LjMwODh6IG04MTkuMiAwYzE0LjEzMTIgMCAyNS42IDYuOTEyIDI1LjYgMTUuMzZ2Nzg4LjQ4YzAgOC40NDgtMTEuNDY4OCAxNS4zNi0yNS42IDE1LjM2cy0yNS42LTYuOTEyLTI1LjYtMTUuMzZWMTE3Ljc2YzAtOC40NDggMTEuNDY4OC0xNS4zNiAyNS42LTE1LjM2eiBtMjA0LjggMGMxNC4xMzEyIDAgMjUuNiA2LjkxMiAyNS42IDE1LjM2djc4OC40OGMwIDguNDQ4LTExLjQ2ODggMTUuMzYtMjUuNiAxNS4zNnMtMjUuNi02LjkxMi0yNS42LTE1LjM2VjExNy43NmMwLTguNDQ4IDExLjQ2ODgtMTUuMzYgMjUuNi0xNS4zNnogbTEwMjQgMGMxNC4xMzEyIDAgMjUuNiA2LjkxMiAyNS42IDE1LjM2djc4OC40OGMwIDguNDQ4LTExLjQ2ODggMTUuMzYtMjUuNiAxNS4zNnMtMjUuNi02LjkxMi0yNS42LTE1LjM2VjExNy43NmMwLTguNDQ4IDExLjQ2ODgtMTUuMzYgMjUuNi0xNS4zNnogbS0yNDU3LjYgMTAyLjRjMTQuMTMxMiAwIDI1LjYgNy4wNjU2IDI1LjYgMTUuNzY5NnY2MzQuMDYwOGMwIDguNzA0LTExLjQ2ODggMTUuNzY5Ni0yNS42IDE1Ljc2OTZzLTI1LjYtNy4wMTQ0LTI1LjYtMTUuNzY5NlYyMjAuNTY5NmMwLTguNzA0IDExLjQ2ODgtMTUuNzY5NiAyNS42LTE1Ljc2OTZ6IG0xNjM4LjQgMGMxNC4xMzEyIDAgMjUuNiA3LjE2OCAyNS42IDE1LjkyMzJWODAzLjMyOGMwIDguODA2NC0xMS40Njg4IDE1LjkyMzItMjUuNiAxNS45MjMycy0yNS42LTcuMTY4LTI1LjYtMTUuOTIzMlYyMjAuNjcyYzAtOC44MDY0IDExLjQ2ODgtMTUuOTIzMiAyNS42LTE1LjkyMzJ6IG0tNjE0LjQgNTEuMmMxNC4xMzEyIDAgMjUuNiA2Ljg2MDggMjUuNiAxNS4zNnY1MzIuNDhjMCA4LjQ5OTItMTEuNDY4OCAxNS4zNi0yNS42IDE1LjM2cy0yNS42LTYuODYwOC0yNS42LTE1LjM2VjI3MS4zNmMwLTguNDk5MiAxMS40Njg4LTE1LjM2IDI1LjYtMTUuMzZ6IG0tMjA0LjggMGMxNC4xMzEyIDAgMjUuNiA2Ljg2MDggMjUuNiAxNS4zNnY1MzIuNDhjMCA4LjQ5OTItMTEuNDY4OCAxNS4zNi0yNS42IDE1LjM2cy0yNS42LTYuODYwOC0yNS42LTE1LjM2VjI3MS4zNmMwLTguNDk5MiAxMS40Njg4LTE1LjM2IDI1LjYtMTUuMzZ6IG0xNDMzLjYtNTEuMmMxNC4xMzEyIDAgMjUuNiA3LjE2OCAyNS42IDE1LjkyMzJWODAzLjMyOGMwIDguODA2NC0xMS40Njg4IDE1LjkyMzItMjUuNiAxNS45MjMycy0yNS42LTcuMTY4LTI1LjYtMTUuOTIzMlYyMjAuNjcyYzAtOC44MDY0IDExLjQ2ODgtMTUuOTIzMiAyNS42LTE1LjkyMzJ6IG02MTQuNCAwYzE0LjEzMTIgMCAyNS42IDcuMTY4IDI1LjYgMTUuOTIzMlY4MDMuMzI4YzAgOC44MDY0LTExLjQ2ODggMTUuOTIzMi0yNS42IDE1LjkyMzJzLTI1LjYtNy4xNjgtMjUuNi0xNS45MjMyVjIyMC42NzJjMC04LjgwNjQgMTEuNDY4OC0xNS45MjMyIDI1LjYtMTUuOTIzMnogbS0zNDgxLjYgNTEuMmMxNC4xMzEyIDAgMjUuNiA2LjkxMiAyNS42IDE1LjQ2MjR2NDgxLjA3NTJjMCA4LjU1MDQtMTEuNDY4OCAxNS40NjI0LTI1LjYgMTUuNDYyNHMtMjUuNi02LjkxMi0yNS42LTE1LjQ2MjRWMjcxLjQ2MjRDNDA5LjYgMjYyLjkxMiA0MjEuMDY4OCAyNTYgNDM1LjIgMjU2eiBtMjA0LjggNTEuMmMxNC4xMzEyIDAgMjUuNiA3LjAxNDQgMjUuNiAxNS42NjcydjQyOS40NjU2YzAgOC43MDQtMTEuNDY4OCAxNS42NjcyLTI1LjYgMTUuNjY3MnMtMjUuNi02Ljk2MzItMjUuNi0xNS42NjcyVjMyMi44NjcyYzAtOC42NTI4IDExLjQ2ODgtMTUuNjY3MiAyNS42LTE1LjY2NzJ6IG0yMDQuOCAwYzE0LjEzMTIgMCAyNS42IDcuMDE0NCAyNS42IDE1LjY2NzJ2NDI5LjQ2NTZjMCA4LjcwNC0xMS40Njg4IDE1LjY2NzItMjUuNiAxNS42Njcycy0yNS42LTYuOTYzMi0yNS42LTE1LjY2NzJWMzIyLjg2NzJjMC04LjY1MjggMTEuNDY4OC0xNS42NjcyIDI1LjYtMTUuNjY3MnogbTIyNTIuOC0xMDIuNGMxNC4xMzEyIDAgMjUuNiA3LjE2OCAyNS42IDE1LjkyMzJWNzUyLjEyOGMwIDguODA2NC0xMS40Njg4IDE1LjkyMzItMjUuNiAxNS45MjMycy0yNS42LTcuMTY4LTI1LjYtMTUuOTIzMlYyMjAuNjcyYzAtOC44MDY0IDExLjQ2ODgtMTUuOTIzMiAyNS42LTE1LjkyMzJ6IG02MTQuNCA1MS4yYzE0LjEzMTIgMCAyNS42IDYuOTEyIDI1LjYgMTUuNDYyNHY0ODEuMDc1MmMwIDguNTUwNC0xMS40Njg4IDE1LjQ2MjQtMjUuNiAxNS40NjI0cy0yNS42LTYuOTEyLTI1LjYtMTUuNDYyNFYyNzEuNDYyNGMwLTguNTUwNCAxMS40Njg4LTE1LjQ2MjQgMjUuNi0xNS40NjI0eiBtLTgxOS4yIDUxLjJjMTQuMTMxMiAwIDI1LjYgNy4xNjggMjUuNiAxNS45MjMydjM3Ny43NTM2YzAgOC44MDY0LTExLjQ2ODggMTUuOTIzMi0yNS42IDE1LjkyMzJzLTI1LjYtNy4xNjgtMjUuNi0xNS45MjMyVjMyMy4wNzJjMC04LjgwNjQgMTEuNDY4OC0xNS45MjMyIDI1LjYtMTUuOTIzMnogbTEyMjguOCAwYzE0LjEzMTIgMCAyNS42IDcuMTY4IDI1LjYgMTUuODcydjM3Ny44NTZjMCA4LjcwNC0xMS40Njg4IDE1Ljg3Mi0yNS42IDE1Ljg3MnMtMjUuNi03LjE2OC0yNS42LTE1Ljg3MlYzMjMuMDcyYzAtOC43MDQgMTEuNDY4OC0xNS44NzIgMjUuNi0xNS44NzJ6IG0tMzg5MS4yIDUxLjJjMTQuMTMxMiAwIDI1LjYgNy40NzUyIDI1LjYgMTYuNjR2MjczLjkyYzAgOS4yMTYtMTEuNDY4OCAxNi42NC0yNS42IDE2LjY0cy0yNS42LTcuNDc1Mi0yNS42LTE2LjY0di0yNzMuOTJjMC05LjIxNiAxMS40Njg4LTE2LjY0IDI1LjYtMTYuNjR6IG00MDk2IDBjMTQuMTMxMiAwIDI1LjYgNy40NzUyIDI1LjYgMTYuNjR2MjczLjkyYzAgOS4yMTYtMTEuNDY4OCAxNi42NC0yNS42IDE2LjY0cy0yNS42LTcuNDc1Mi0yNS42LTE2LjY0di0yNzMuOTJjMC05LjIxNiAxMS40Njg4LTE2LjY0IDI1LjYtMTYuNjR6IG0yMDQuOCA1MS4yYzE0LjEzMTIgMCAyNS42IDcuMDY1NiAyNS42IDE1Ljc2OTZ2MTczLjI2MDhjMCA4LjcwNC0xMS40Njg4IDE1Ljc2OTYtMjUuNiAxNS43Njk2cy0yNS42LTcuMDY1Ni0yNS42LTE1Ljc2OTZWNDI1LjM2OTZjMC04LjcwNCAxMS40Njg4LTE1Ljc2OTYgMjUuNi0xNS43Njk2ek0yNS42IDQwOS42YzE0LjEzMTIgMCAyNS42IDcuMDY1NiAyNS42IDE1Ljc2OTZ2MTczLjI2MDhjMCA4LjcwNC0xMS40Njg4IDE1Ljc2OTYtMjUuNiAxNS43Njk2cy0yNS42LTcuMDY1Ni0yNS42LTE1Ljc2OTZWNDI1LjM2OTZDMCA0MTYuNjY1NiAxMS40Njg4IDQwOS42IDI1LjYgNDA5LjZ6IiBmaWxsPSIjODA4QTg3IiBwLWlkPSI3MTIxIj48L3BhdGg+PC9zdmc+")
    center/8% repeat-x;
  border-radius: 4px;
  .left-edge,
  .right-edge {
    position: absolute;
    top: 0;
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
  }
  .right-edge {
    border-top-right-radius: 4px;
    border-bottom-right-radius: 4px;
  }
  .content {
    position: absolute;
    top: 0;
    box-sizing: border-box;
    height: @bar-height;
    border-top: 2px solid @theme-color;
    border-bottom: 2px solid @theme-color;
    background: transparent;
  }
}
.video-frame {
  display: flex;
  height: 100%;
  div {
    flex: 1;
  }
  video {
    object-fit: cover;
  }
}
</style>
