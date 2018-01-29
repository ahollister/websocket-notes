<template>
  <div class="header">
    <h1>{{ title }}</h1>
  </div>
</template>

<script>
  export default {
    name: 'header',
    props: ['title']
  }
</script>

<style scoped>
h1 {
  margin: 0 0 0 0;
  padding: 20px 0;
  text-align: center;
  text-transform: uppercase;
  color: white;
}
.header {
  margin-bottom: 10px;
  background-color: #ff7675;
}
</style>
