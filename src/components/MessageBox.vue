<template>
  <div>
    <h2>{{ title }}</h2>
    <div class="messagebox-form">
      <textarea name="messagebox"
                class="messagebox"
                maxlength="50"
                v-bind:placeholder="placeholder"
                v-model="message">
        {{ message }}
      </textarea>
      <button href="#" v-on:click="submit">Submit</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: ''
    }
  },
  name: 'messagebox',
  props: ['title', 'placeholder'],
  methods: {
    submit: function() {
      this.$emit('submit', this.message)
      this.message = ''
    }
  }
}
</script>

<style scoped>
h2 {
  padding-left: 20px;
}
textarea {
  display: block;
  width: 100%;
  margin-bottom: 10px;
  padding: 10px;
  border: 1px solid #f2f2f2;
  border-radius: 4px;
}
.messagebox-form {
  text-align: right;
  background: white;
  border: 1px solid #f2f2f2;
  border-bottom-width: 2px;
  padding: 20px
}
</style>
