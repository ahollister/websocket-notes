<template>
  <div class="container">
    <h2>Edit Note:</h2>
    <div class="card">
      <textarea name="editNote"
                maxlength="500"
                v-model="content">
        {{content}}
      </textarea>
      <button href="#" v-on:click="submit">Submit</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      content: ''
    }
  },
  name: 'editnote',
  props: ['note'],
  mounted: function() {
    this.content = this.note.content
  },
  methods: {
    submit: function() {
      if (this.content.length) {
        this.$emit('submit', this.content)
      }
      this.content = ''
    }
  }
}
</script>

<style scoped>
.card {
  text-align: right;
}
</style>
