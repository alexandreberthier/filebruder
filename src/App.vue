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
  width: min(760px, calc(100% - 24px));
  margin: 0 auto;
  padding: 24px 0 48px;
}

main {
  flex: 1;
}
</style>
