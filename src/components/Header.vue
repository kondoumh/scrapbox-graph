<template>
  <header>
    <h2>{{ title }}</h2>
  </header>
</template>

<script>
  export default {
    name: "Header",
    data() {
      return {
        title: 'Scrapbox graph'
      }
    }
  }
</script>

<style scoped>
header {
  position:fixed;
  top:0;
  left:0;
  right:0;
  height: 50;
}

h2 {
  margin: 0;
  margin-left: 1em;
  color: #000;
  text-align: left;
}
</style>
