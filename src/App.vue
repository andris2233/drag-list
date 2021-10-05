<template>
  <div id="app">
    <VDragList />
  </div>
</template>

<script>
import VDragList from '@/components/VDragList.vue';

export default {
  name: 'App',
  components: { VDragList },
};
</script>

<style lang="scss">
 #app {
   font-family: sans-serif;
 }
</style>
