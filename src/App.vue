<!--
 * @Author: rk
 * @Description: 根组件
 * @Date: 2023-03-17 17:48:40
 * @LastEditors: rk
 * @LastEditTime: 2024-04-10 16:17:11
-->

<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: "App",
  metaInfo() {
    return {
      title: this.$store.state.settings.title,
      titleTemplate: (title) => {
        return title
          ? `${title} - ${process.env.VUE_APP_TITLE}`
          : process.env.VUE_APP_TITLE;
      },
    };
  },
};
</script>
<style>
* {
  padding: 0;
  margin: 0;
}
</style>
