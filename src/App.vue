<template>
  <div id="app">
    <nav class="navbar navbar-dark bg-dark">
      <div class="container">
          <span class="navbar-brand" href="#">Game of life</span>
      </div>
    </nav>
    <div class="container">
      <Grid />
    </div>
    
  </div>
</template>

<script>
import Grid from './components/Grid.vue'

export default {
  name: 'app',
  components: {
    Grid
  }
}
</script>

<style>

</style>
