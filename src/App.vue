<template>
  <div>
    <Button type="primary" @click="visible = true">打开</Button>
    <CutVideo url="/v2.mp4" :visible.sync="visible" @confirm="handleConfirm" />
  </div>
</template>

<script>
import CutVideo from "@/components/CutVideo";

export default {
  components: {
    CutVideo
  },

  data() {
    return {
      visible: false
    };
  },

  methods: {
    handleConfirm(val) {
      console.log(val);
    }
  }
};
</script>

<style></style>
