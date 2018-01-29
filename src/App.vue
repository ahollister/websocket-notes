<template>
  <div>
    <Header title="Notes" />
    <div class="body">
      <MessageBox v-if="!isLoggedIn"
                  title="Log in:"
                  placeholder="Your username ..."
                  v-on:submit="login" />

      <div class="user" v-if="isLoggedIn">
        Logged in as: {{ username }}
      </div>
      <MessageBox v-if="isLoggedIn"
                  title="Create note:"
                  placeholder="Write your content here ..."
                  v-on:submit="createNote" />
    </div>
  </div>
</template>

<script>
import Rx from 'rxjs/Rx';
import Header from './components/Header.vue'
import MessageBox from './components/MessageBox.vue'

export default {
  name: 'app',
  components: {
    Header,
    MessageBox
  },
  data () {
    return {
      isLoggedIn: false,
      username: '',
    }
  },
  methods: {
    login: function(username) {
      this.username = username
      this.isLoggedIn = true
    },
    createNote: function(note) {
      console.log('Submitted note: ' + note)
    }
  },
  created: function () {
    var socket = new WebSocket( 'wss://echo.websocket.org' )

    socket.onopen = function(event) {
      console.log(event)
    }
    socket.onmessage = function(event) {
      console.log(event)
    }
  }
}
</script>
