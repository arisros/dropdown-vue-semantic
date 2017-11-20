<template>
  <div id="app">
    <dropdown 
      name="search_pasar"
      fluid
      search-able
      placeholder="-Pasar-"
      field-name="nama"
      field-id="id"
      v-on:selected="selectChange"
      v-bind:default-selected="search_pasar"
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
      search_pasar: 0
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
