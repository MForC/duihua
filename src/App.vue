<template>
  <div id="app">
    <div class="container">
      <TryAskMe />
    </div>
  </div>
</template>

<script>
import TryAskMe from './components/TryAskMe.vue'

export default {
  name: 'App',
  components: {
    TryAskMe
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  background: linear-gradient(180deg, #f0f4f8 0%, #e8ecef 100%);
  min-height: 100vh;
}

#app {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 20px;
}

.container {
  width: 100%;
  max-width: 652px;
}
</style>
