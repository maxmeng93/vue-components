<template>
  <div id="app">
    <ElSelectBigdataDemo />
    <ParagraphDemo />
  </div>
</template>

<script>
import ElSelectBigdataDemo from '@/components/ElSelectBigdata/Demo.vue';
import ParagraphDemo from '@/components/Paragraph/Demo.vue';

export default {
  name: 'App',
  components: {
    ElSelectBigdataDemo,
    ParagraphDemo,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
