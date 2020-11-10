<template>
  <div class="cut-video">
    <video ref="video" src="/v2.mp4" controls width="800" height="450"></video>
    <TimeRangeBar
      @start-change="handleStarChange"
      @end-change="handleEndChange"
      @start-drag-callback="removeVideoLoopEventListener"
    />
    <div class="oper-box">
      <div class="btn-group">
        <input type="button" value="预览" @click="handlePreview" />
      </div>
      <div class="time-info">
        <span
          >开始时间：<span class="time">{{ startTimeText }}</span></span
        >
        <span
          >结束时间：<span class="time">{{ endTimeText }}</span></span
        >
      </div>
      <div class="btn-group">
        <input type="button" value="确定" />
      </div>
    </div>
  </div>
</template>

<script>
import TimeRangeBar from "@/components/TimeRangeBar";
import dayjs from "dayjs";

export default {
  components: {
    TimeRangeBar
  },
  data() {
    return {
      videoDom: null,
      videoDuration: 0,
      startTime: 0,
      endTime: 0
    };
  },
  computed: {
    startTimeText() {
      const t = dayjs(this.startTime * 1000 + 16 * 60 * 60 * 1000);
      return t.format("HH:mm:ss");
    },
    endTimeText() {
      const t = dayjs(this.endTime * 1000 + 16 * 60 * 60 * 1000);
      return t.format("HH:mm:ss");
    }
  },
  mounted() {
    this.videoDom = this.$refs.video;
    this.videoDom.addEventListener("loadedmetadata", () => {
      this.endTime = this.videoDuration = this.videoDom.duration;
    });
  },
  methods: {
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
    },
    loopPlay() {
      if (Math.floor(this.videoDom.currentTime) === this.endTime) {
        this.videoDom.currentTime = this.startTime;
      }
    },
    // 移除video的循环播放判定
    removeVideoLoopEventListener() {
      this.videoDom.pause();
      this.videoDom.removeEventListener("timeupdate", this.loopPlay);
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
  .btn-group {
    width: 100px;
  }
  .time-info {
    flex: 1;
    text-align: center;
  }
}
.time-info {
  margin-top: 20px;
  font-size: 12px;
  color: #999;
  span {
    margin-right: 20px;
  }
}
</style>
