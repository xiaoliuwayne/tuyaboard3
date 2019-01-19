<template>
  <div id="wrapper">
    <case-header :navigator="navigator"/>
    <div id="main">
      <router-view @setNav="setNav($event)"></router-view>
    </div>
    <case-footer/>
  </div>
</template>

<script>
    export default {
        name: "layout"
    }
</script>

<style scoped>
  #wrapper {
    background: url('../../static/imgs/AAA.jpg') no-repeat;
    background-size: 50px 50px;
  }
</style>
