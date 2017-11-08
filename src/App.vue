<template>
  <div id="app">
    <dropdown 
      name="select"
      fluid 
      placeholder="Select First"
      field-name="name"
      field-id="id"
      v-bind:default-selected="select"
      v-bind:options="options"
      ></dropdown>
  </div>
</template>

<script>
import dropdown from './Dropdown-vue-semantic'

export default {
  name: 'app',
  components: {
    dropdown
  },
  data () {
    return {
      select: 0,
      options: [{id: 1, name: 'Dropdown test'}, {id: 2, name: 'Dropdown test 2'}]
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
