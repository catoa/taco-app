<template>
  <div id="app">
      <TacoRecipe />
  </div>
</template>

<script>
import TacoRecipe from './components/TacoRecipe.vue'

export default {
  name: 'app',
  components: {
    TacoRecipe
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
}
</style>
