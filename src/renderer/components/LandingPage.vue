<template>
  <div id="wrapper">
    <div id="top_nav">Tagger > <router-link :to="{ name: 'landing-page', params: {}}">Projects</router-link></div>
    <main>

      <div>
        <h1>Projects</h1>

        <ul>
          <li v-for="(project, index) in projects">
            <router-link :to="{ name: 'project-page', params: {id: project._id}}">{{ project.project_name }}</router-link> {{ project.created_at | moment("from")}}
          </li>
        </ul>

        <button @click="newProject()">New Project</button>
      </div>
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
        vm.$db.insert({project_name: "My cool film", file: null, tags: null, created_at: Date.now()})
        vm.loadProjects()
      }
    }
  }
</script>

<style lang="scss">

</style>
