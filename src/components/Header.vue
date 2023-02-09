<template>
  <header>
    <h2>{{ title }}</h2>
    <select v-model="project">
      <option>help-jp</option>
      <option>icons</option>
      <option>comic-forum</option>
    </select>
  </header>
</template>

<script>
  export default {
    name: "Header",
    data() {
      return {
        title: 'Scrapbox graph',
        project: 'help-jp'
      }
    }
  }
</script>

<style scoped>
header {
  position: fixed;
  top: 0px;
  left: 0px;
  right: 0px;
  height: 50px;
}

h2 {
  margin: 0;
  margin-left: 1em;
  text-align: left;
}

select {
  position: fixed;
  top: 0px;
  right: 0px;
  margin: 1em;
}
</style>
