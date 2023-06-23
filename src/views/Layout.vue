<template>
  <div class="main">
    <Header />
    <div class="body-content">
      <router-view />
    </div>
    <!-- <Dialog :isShow="showDialog" :buttons="buttons" @close="showDialog = false"></Dialog> -->
    <Footer/>
  </div>
</template>

<script setup>
import Header from "@/views/Header.vue";
import Footer from "@/components/Footer.vue";
const buttons = [
  {
    text: "确定",
    type: "primary",
  },
];
</script>

<style lang="scss">
.body-content {
  margin-top: 80px;
  position: relative;
  min-height: calc(100vh - 210px);
}
</style>
