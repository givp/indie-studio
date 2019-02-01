<template>
  <div id="wrapper">
    <main>
      <h1>Landing</h1>

      <ul>
        <li v-for="(project, index) in projects">
          <router-link :to="{ name: 'project-page', params: {id: project._id}}">{{ project.project_name }}</router-link> {{ project.created_at | moment("from")}}
        </li>
      </ul>

      <button @click="newProject()">New Project</button>

    </main>
  </div>
</template>

<script>
  export default {
    name: 'landing-page',
    data () {
      return {
        projects: []
      }
    },
    mounted: function () {
      this.loadProjects()
    },
    methods: {
      loadProjects() {
        var vm = this
        vm.$db.find({}).sort({ created_at: -1 }).exec(function (err, docs) {
          vm.projects = docs
        });
      },
      newProject() {
        var vm = this
        vm.$db.insert({project_name: "My cool film", created_at: Date.now()})
        vm.loadProjects()
      }
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
