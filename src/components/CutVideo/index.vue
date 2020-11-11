<template>
  <Modal
    :styles="{
      top: top + 'px'
    }"
    @input="val => $emit('update:visible', val)"
    width="900"
    title="裁剪视频"
    :value="visible"
  >
    <Content ref="content" />
    <template v-slot:footer>
      <Button type="text" @click="$emit('update:visible', false)">取消</Button>
      <Button type="primary" @click="handleConfirm">保存</Button>
    </template>
  </Modal>
</template>

<script>
import Content from "./Content";

export default {
  components: {
    Content
  },
  provide() {
    return {
      dialog: this
    };
  },
  props: {
    visible: Boolean,
    url: String
  },
  created() {
    this.top = (window.innerHeight - 720) / 2;
  },
  data() {
    return {
      top: 0,
      loading: false
    };
  },
  methods: {
    handleConfirm() {
      const { cutAllTime, startTime, endTime } = this.$refs.content;
      const all = Math.floor(cutAllTime);
      if (all < 7 || all > 48) {
        this.$Message.error("裁剪视频总时长需为7秒到48秒");
      } else {
        this.$emit("update:visible", false);
        this.$emit("confirm", {
          cutBeginTime: Math.floor(startTime),
          cutEndTime: Math.floor(endTime)
        });
      }
    }
  }
};
</script>

<style></style>
