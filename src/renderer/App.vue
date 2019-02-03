<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
  export default {
    name: 'script-tagger'
  }
</script>

<style lang="scss">
  @import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro');

  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  body {
    font-family: 'Source Sans Pro', sans-serif;
    background-color: #EBEFF2;
    color: #4C5359;
    margin: 0;
    padding: 0;
  }

  #top_nav {
    background-color: #fff;
    height: 50px;
    line-height: 50px;
    padding-left: 20px;
  }

  main {
    display: flex;
    width: 100%;
    margin-top: 50px;
    margin-left: 50px;
  }

  a, a:hover {
    color: #11A0F8;
    text-decoration: none;
  }
</style>
