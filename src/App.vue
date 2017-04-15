<template>
  <div id="app">
    <nav class="navbar navbar-default" role="navigation">
      <div class="container">
        <div class="navbar-header">
          <button type="button" class="navbar-toggle" data-toggle="collapse" data-target="#navbar">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </button>
          <a class="navbar-brand" href="/">
            <img alt="Brand" :src="logo" class="img-responsive">
          </a>
        </div>
        <p class="navbar-text navbar-right">
          Vue Markdown Editor by <a href="https://github.com/franckysolo" class="navbar-link">Franck Matherat</a>
        </p>
      </div><!-- /.container-fluid -->
    </nav>
    <div class="container">
      <h1>Vue Markdown Editor - Version prototype 0.1.0</h1>
      <p>A javascript editor with syntax coloring and image manager</p>
      <md-editor></md-editor>
    </div>
    <div class="container hide">
      <h2>Dependencies</h2>
      <ul class="">
        <li>Vuejs</li>
        <li>Bootstrap</li>
        <li>jQuery</li>
        <li>Font awesome</li>
        <li>Marked</li>
        <li>Highlight.js</li>
        <li>Toastr</li>
        <li>Lodash</li>
      </ul>
    </div>
  </div>
</template>

<script>
import MdEditor from './components/MarkdownEditor'

export default {
  name: 'app',
  components: {
    MdEditor
  },
  data () {
    return {
      logo: require('@/assets/img/favicon-small.png')
    }
  }
}
</script>

<style lang="less">
 @import 'assets/less/main.less';
</style>
