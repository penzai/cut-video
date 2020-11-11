<template>
  <div class="cut-video">
    <video ref="video" src="/v2.mp4" width="800" height="450"></video>
    <TimeRangeBar
      @start-change="handleStarChange"
      @end-change="handleEndChange"
      @start-drag-callback="removeVideoLoopEventListener"
    />
    <div class="oper-box">
      <div class="preview-btn-group">
        <Button type="primary" @click="handlePreview">预览</Button>
        <span style="margin-left: 10px;font-size:12px;"
          ><span v-show="isPlaying">{{ cutCurrentTimeText }} / </span
          >{{ cutAllTimeText }}</span
        >
      </div>
      <div class="time-info">
        <span
          >开始时间：<span class="time">{{ startTimeText }}</span></span
        >
        <span
          >结束时间：<span class="time">{{ endTimeText }}</span></span
        >
        <span>总时长：{{ Number.parseInt(cutAllTime) + 1 }} 秒</span>
      </div>
      <div class="btn-group" style="text-align: right;">
        <Button type="primary" @click="handleConfirm">确定</Button>
      </div>
    </div>
  </div>
</template>

<script>
import TimeRangeBar from "./TimeRangeBar";
import dayjs from "dayjs";

export default {
  components: {
    TimeRangeBar
  },
  provide() {
    return {
      root: this
    };
  },
  data() {
    return {
      videoDom: null,
      videoDuration: 0,
      startTime: 0,
      endTime: 0,

      cutCurrentTime: 0, //裁剪视频当前播放时间
      isPlaying: false //是否正在播放裁剪视频
    };
  },
  computed: {
    startTimeText() {
      return this.getTimeText(this.startTime);
    },
    endTimeText() {
      return this.getTimeText(this.endTime);
    },
    cutCurrentTimeText() {
      return this.getTimeText(this.cutCurrentTime - this.startTime);
    },
    // 裁剪视频总长
    cutAllTime() {
      return this.endTime - this.startTime;
    },
    cutAllTimeText() {
      return this.getTimeText(this.cutAllTime);
    }
  },
  mounted() {
    this.videoDom = this.$refs.video;
    this.videoDom.addEventListener("loadedmetadata", () => {
      this.endTime = this.videoDuration = this.videoDom.duration;
    });
  },
  methods: {
    getTimeText(t) {
      return dayjs(t * 1000 + 16 * 60 * 60 * 1000).format("HH:mm:ss");
    },
    handleStarChange(v) {
      this.startTime = this.videoDom.currentTime = Math.floor(
        this.videoDuration * v
      );
    },
    handleEndChange(v) {
      this.endTime = this.videoDom.currentTime = Math.floor(
        this.videoDuration * v
      );
    },
    handlePreview() {
      this.videoDom.currentTime = this.startTime;
      this.videoDom.addEventListener("timeupdate", this.loopPlay);
      this.videoDom.play();
      this.isPlaying = true;
    },
    loopPlay() {
      const time = Math.floor(this.videoDom.currentTime);
      this.cutCurrentTime = time;
      if (time === this.endTime + 1) {
        this.videoDom.currentTime = this.startTime;
      }
    },
    // 移除video的循环播放判定
    removeVideoLoopEventListener() {
      this.isPlaying = false;
      this.videoDom.pause();
      this.videoDom.removeEventListener("timeupdate", this.loopPlay);
    },
    handleConfirm() {
      const len = this.endTime - this.startTime;
      if (len < 9 || len > 45) {
        this.$Message.error("只支持裁剪9秒到45秒的视频");
      }
    }
  }
};
</script>

<style lang="less" scoped>
.cut-video {
  width: 800px;
  margin: auto;
  position: relative;
  video {
    margin-bottom: 15px;
  }
}
.oper-box {
  display: flex;
  height: 40px;
  line-height: 40px;
  margin-top: 20px;
  .preview-btn-group {
    width: 200px;
  }
  .btn-group {
    width: 130px;
  }
  .time-info {
    flex: 1;
    text-align: center;
  }
  .time-info {
    font-size: 12px;
    color: #999;
    span {
      margin-right: 20px;
    }
    .time {
      padding: 10px;
      background: #f7f7f7;
      border-radius: 5px;
      font-size: 16px;
      color: #222;
    }
  }
}
</style>
