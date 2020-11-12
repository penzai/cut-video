<template>
  <div>
    <video
      ref="video"
      :src="src"
      :width="usedOptions.width"
      :height="usedOptions.height"
    ></video>
    <div>
      <Icon
        v-show="isPaused || isPlayCompleted"
        type="ios-play"
        @click="play"
      />
      <Icon v-show="isPlaying" @click="pause" type="ios-pause" />
      <span style="margin-left: 20px;"
        >{{ currentTimeText }} / {{ videoDurationText }}</span
      >
    </div>
  </div>
</template>

<script>
import dayjs from "dayjs";

export default {
  props: {
    src: String,
    options: Object
  },

  data() {
    return {
      videoDom: null,
      usedOptions: null,
      currentTime: null,
      videoDuration: null,
      playStatus: ""
    };
  },

  computed: {
    currentTimeText() {
      return this.getTimeText(this.currentTime);
    },
    videoDurationText() {
      return this.getTimeText(this.videoDuration);
    },
    isPlaying() {
      return this.playStatus === "play";
    },
    isPaused() {
      return this.playStatus === "pause";
    },
    isPlayCompleted() {
      return this.playStatus === "complete";
    }
  },

  watch: {
    src: "loadVideo"
  },

  created() {
    this.usedOptions = {
      width: 256,
      height: 148,
      ...this.options,
      start: this.options.start || 0,
      end: this.options.end || 0
    };
  },

  mounted() {
    this.loadVideo();
  },

  methods: {
    getTimeText(t) {
      return dayjs(t * 1000 + 16 * 60 * 60 * 1000).format("HH:mm:ss");
    },
    loadVideo() {
      if (!this.src) return;

      const { start, end } = this.usedOptions;
      if (!this.videoDom) {
        this.videoDom = this.$refs.video;
        if (end) {
          this.videoDuration = end - start;
          this.videoDom.addEventListener("timeupdate", () => {
            const time = Math.floor(this.videoDom.currentTime);
            this.currentTime = time;
            if (time === end + 1) {
              this.videoDom.pause();
              this.videoDom.currentTime = start;
              this.currentTime = start;
              this.playStatus = "complete";
            }
          });
        }
      }
      this.videoDom.currentTime = start;
      this.currentTime = start;
      this.playStatus = "pause";
    },
    play() {
      this.videoDom.play();
      this.playStatus = "play";
    },
    pause() {
      this.videoDom.pause();
      this.playStatus = "pause";
    }
  }
};
</script>

<style lang="less" scoped>
.ivu-icon {
  cursor: pointer;
}
</style>
