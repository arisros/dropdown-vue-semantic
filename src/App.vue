<template>
  <div id="app">
    <dropdown 
      name="select"
      fluid 
      search-able
      placeholder="Select First"
      field-name="nama"
      field-id="id"
      v-on:selected="selectChange"
      v-bind:default-selected="select"
      end-point="http://pdpasar.docotel.net/v1/pasars"
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
      disabled: true,
      select: 0,
      options: [{id: 1, name: 'Dropdown test'}, {id: 2, name: 'Dropdown test 2'}]
    }
  },
  methods: {
    selectChange (e) {
      this[e.name] = e.value
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
