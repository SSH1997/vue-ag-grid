<template>
  <div id="app">
    <defaultGrid />
  </div>
</template>

<script>
import defaultGrid from './components/defaultGrid.vue'

export default {
  name: 'app',
  components: {
    defaultGrid
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  @import "../node_modules/ag-grid-community/dist/styles/ag-grid.css";
  @import "../node_modules/ag-grid-community/dist/styles/ag-theme-balham.css";
}

.ag-header-cell-label{justify-content: center;}
</style>
