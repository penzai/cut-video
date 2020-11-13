<template>
  <Modal
    :styles="{
      top: top + 'px'
    }"
    @input="val => $emit('update:visible', val)"
    :width="dialogWidth"
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
  computed: {
    dialogWidth() {
      const clientWidth = window.innerWidth;
      const relativeWidth = clientWidth * 0.75;
      return relativeWidth < 900 ? 900 : relativeWidth;
    }
  },
  props: {
    visible: Boolean,
    url: String,
    defaultCutInfo: [Object, null]
  },
  watch: {
    visible(val) {
      if (val && this.isError) {
        this.$Message.error("视频地址有误，暂不能裁剪");
      }
      if (val) {
        this.$emit("init-default-data");
      }
      if (!val) {
        this.$refs.content.handlePreview();
      }
    }
  },
  created() {
    this.top = (window.innerHeight - 720) / 2;
  },
  data() {
    return {
      top: 0,
      loading: false,
      isError: false
    };
  },
  methods: {
    handleConfirm() {
      if (this.isError) {
        this.$Message.error("视频地址有误，暂不能裁剪");
        return;
      }
      const {
        cutAllTime,
        startTime,
        endTime,
        startTimeText,
        endTimeText
      } = this.$refs.content;
      const all = Math.floor(cutAllTime);
      if (all < 7 || all > 48) {
        this.$Message.error("裁剪视频总时长需为7秒到48秒");
      } else {
        this.$emit("update:visible", false);
        this.$emit("confirm", {
          cutBeginTimeText: startTimeText,
          cutEndTimeText: endTimeText,
          cutAllTime: Math.floor(cutAllTime),
          cutBeginTime: Math.floor(startTime),
          cutEndTime: Math.floor(endTime)
        });
      }
    }
  }
};
</script>

<style></style>
