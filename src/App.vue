<template>
  <div id="app">
    <my-table></my-table>
  </div>
</template>

<script>
import MyTable from './components/MyTable.vue'

export default {
  name: 'App',
  components: {
    MyTable
  }
}
</script>

<style>
@import url('https://cdn.jsdelivr.net/npm/bootstrap@4.5.3/dist/css/bootstrap.min.css');

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
