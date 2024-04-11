<!--
 * @Author: rk
 * @Description: 首页
 * @Date: 2023-03-17 17:48:40
 * @LastEditors: rk
 * @LastEditTime: 2024-02-05 11:29:01
-->

<template>
  <div class="app-container">
    <base-three></base-three>
  </div>
</template>
<script>
export default {
  name: "Home",
  data() {
    return {
      tilte: "", // 问候语
    };
  },
  computed: {
    // sidebarRoutes() {
    //   return this.$store.state.common.sidebarRoutes;
    // },
  },
  created() {
    this.welcomeMessageinit();
  },
  methods: {
    welcomeMessageinit() {
      let date = new Date();
      let h = date.getHours();
      if (h < 12) {
        this.tilte = "上午好";
      } else if (h >= 12 && h < 13) {
        this.tilte = "中午好";
      } else if (h >= 13 && h < 19) {
        this.tilte = "下午好";
      } else {
        this.tilte = "晚上好";
      }
    },
  },
};
</script>
<style lang="scss" scoped>
h2 {
  font-size: $fs16;
}
.overview {
  line-height: 30px;
  color: $color-success;
}
</style>
