<template>
  <div id="wrapper">
    <main>
      <h1>Landing</h1>
      <router-link :to="{ name: 'project-page', params: {}}">Project</router-link>
    </main>
  </div>
</template>

<script>
  export default {
    name: 'landing-page',
    data () {
      return {

      }
    },
    mounted: function () {

    },
    methods: {

    }
  }
</script>

<style>
  @import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro');

  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  body {
    font-family: 'Source Sans Pro', sans-serif;
    background-color: #e6e6e6;
    font-color: #333;
    padding: 50px;
  }

  #wrapper {

  }
  main > div { flex-basis: 50%; }
</style>
