<template>
  <div class="app">
    <main>
      <RouterView/>
    </main>
  </div>
</template>

<script setup lang="ts">

</script>

<style scoped>

.app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  width: calc(100% - 54px);
  margin: 0 auto;

  main {
    flex: 1;
  }
}

@media (min-width: 1200px) {
  .app {
    width: 1200px;
  }
}

</style>