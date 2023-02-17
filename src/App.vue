

<template>
  <div class="app">
    <Bmap/>
  </div>
</template>
<script setup>
import { onMounted } from "vue";
import Bmap from '@/components/Bmap.vue'
</script>
<style scoped lang="scss">
.app {
  width: 100%;
  height: 100%;
}
</style>
