<template>
  <div>
    <Button type="primary" @click="visible = true">打开</Button>
    <CutVideo url="/v2.mp4" :visible.sync="visible" @confirm="handleConfirm" />
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
      options: null
    };
  },

  methods: {
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
