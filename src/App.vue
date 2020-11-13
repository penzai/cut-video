<template>
  <div>
    <Button type="primary" @click="visible = true">打开</Button>
    <CutVideo
      :default-cut-info="defaultCutInfo"
      url="/v2.mp4"
      :visible.sync="visible"
      @confirm="handleConfirm"
    />
    <VideoShow v-if="show" src="/v2.mp4" :options="options" />
  </div>
</template>

<script>
import CutVideo from "@/components/CutVideo";
import VideoShow from "@/components/VideoShow";

export default {
  components: {
    CutVideo,
    VideoShow
  },

  data() {
    return {
      visible: false,
      show: false,
      options: null,
      defaultCutInfo: null
    };
  },

  mounted() {
    const startText = "00:00:03";
    const endText = "00:00:12";
    const start = this.getSecondByF1(startText);
    const end = this.getSecondByF1(endText);
    this.defaultCutInfo = {
      cutAllTime: end - start,
      cutBeginTime: start,
      cutEndTime: end,
      cutBeginTimeText: startText,
      cutEndTimeText: endText
    };
  },

  methods: {
    getSecondByF1(str) {
      const regExp = /(\d{2}):(\d{2}):(\d{2})/;
      const ret = str.match(regExp);
      if (!ret) return 0;
      const h = ~~ret[1];
      const m = ~~ret[2];
      const s = ~~ret[3];
      return h * 60 * 60 + m * 60 + s;
    },
    handleConfirm(val) {
      console.log(val);
      this.show = true;
      this.options = {
        start: val.cutBeginTime,
        end: val.cutEndTime
      };
    }
  }
};
</script>

<style></style>
