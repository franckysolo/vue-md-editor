<template lang="html">
  <div class="ui-loading" v-show="loading">
    <div class="loader"></div>
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: 'Loading',
  props: ['loading']
}
</script>

<style lang="less">
.ui-loading {
  width: 100%;
  padding:10px;
  text-align: center;
}

.loader {
  border: 5px solid #ccc;
  border-top: 5px solid #333;
  border-bottom: 5px solid #333;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 2s linear infinite;
  margin: 10px auto;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
