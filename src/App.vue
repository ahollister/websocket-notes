<template>
  <div>
    <AppHeader title="Notes" />
    <div class="app-body">
      <MessageBox title="Log in:"
                  placeholder="Your username ..."
                  v-if="!isLoggedIn"
                  v-on:submit="login" />

      <div v-if="isLoggedIn">
        <div class="user">Logged in as: {{username}}</div>
        <a href="#"
           class="user-logout"
           v-on:click="logout(username)">
          Log out
        </a>
      </div>

      <MessageBox title="Create note:"
                  placeholder="Write your content here ..."
                  v-if="isLoggedIn && !isEditing"
                  v-on:submit="submitNote" />

      <EditNote v-if="isLoggedIn && isEditing"
                v-on:submit="submitEditedNote"
                v-bind:note="noteToEdit" />

      <div class="user-list container" v-if="isLoggedIn">
        <h2>Users:</h2>
        <ul class="users">
          <li v-for="user in users">
            {{user}}
          </li>
        </ul>
      </div>

      <ul class="notes" v-if="isLoggedIn">
        <li class="note card" v-for="note in notes">
          <ul class="actions">
            <li><EditIcon v-on:click="editNote(note)" /></li>
            <li><Trash2Icon v-on:click="deleteNote(note)" /></li>
          </ul>
          {{note.content}}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import AppHeader from './components/AppHeader.vue'
import MessageBox from './components/MessageBox.vue'
import EditNote from './components/EditNote.vue'
import {EditIcon, Trash2Icon} from 'vue-feather-icons'

export default {
  name: 'app',
  components: {
    AppHeader,
    MessageBox,
    EditNote,
    EditIcon,
    Trash2Icon
  },
  data() {
    return {
      isLoggedIn: false,
      username: '',
      users: [],
      notes: [],
      isEditing: false,
      noteToEdit: {}
    }
  },
  mounted: function() {
    // Update with notes from storage
    if (localStorage.getItem('notes') !== null) {
      this.notes = JSON.parse(localStorage.getItem('notes'))
    }

    // Open socket connection
    window.socket = new WebSocket('wss://vast-brushlands-58657.herokuapp.com')

    // Handle socket events
    socket.onopen = (event) => {
      console.log('socket opened ... ')
    }

    // Recieve messages from websocket and interpret actions
    socket.onmessage = (event) => {
      let data = JSON.parse(event.data)
      console.log('Message recieved, action: ' + data.action)
      console.log(data)

      if (data.action === 'login') {
        this.users.push(data.username)
        this.sendUpdate(this.users, this.notes)
      } else if (data.action === 'logout' && data.username !== '') {
        this.users.splice(this.users.indexOf(data.username), 1)
      } else if (data.action === 'createNote') {
        this.notes.unshift(data.note)
        this.saveNotes()
      } else if (data.action === 'deleteNote') {
        this.notes = this.notes.filter((item) => {
          if (item.timestamp !== data.note.timestamp) {
            return item
          }
        })
        this.saveNotes()
      } else if (data.action === 'update') {
        this.recieveUpdate(data)
      } else if (data.action === 'replaceNotes') {
        this.notes = data.notes
        this.saveNotes()
      }
    }

    socket.onclose = () => {
      let data = {
        action: 'logout',
        username: this.username
      }
      socket.send(JSON.stringify(data))
    }

    window.onbeforeunload = () => {
      socket.onclose = () => {} // Handling onclose here when window is closed
      let data = {
        action: 'logout',
        username: this.username
      }
      socket.send(JSON.stringify(data))
      socket.close()
    }

    socket.onerror = function(event) {
      console.log(event)
    }
  },
  methods: {
    // login - sets state and updates other users
    login: function(username) {
      console.log('login() ... ')
      this.username = username
      this.isLoggedIn = true
      let data = {action: 'login', username}
      socket.send(JSON.stringify(data))
    },

    // logout - sets state, revoking access to app, and updates other users
    logout: function(username) {
      console.log('logout() ... ')
      this.isLoggedIn = false
      let data = {action: 'logout', username}
      socket.send(JSON.stringify(data))
      this.username = ''
    },

    // submitNote - constructs note object and send to websocket
    submitNote: function(note) {
      console.log('submitNote() ... ')
      let data = {
        action: 'createNote',
        note: {
          content: note,
          user: this.username,
          timestamp: Date.now()
        }
      }
      socket.send(JSON.stringify(data))
    },

    // deleteNote - send deleteNote action
    deleteNote: function(note) {
      console.log('deleteNote() ... ')
      let data = {action: 'deleteNote', note}
      socket.send(JSON.stringify(data))
    },

    // saveNotes - save notes to localStorage
    saveNotes: function() {
      console.log('saveNotes() ... ')
      localStorage.setItem('notes', JSON.stringify(this.notes))
    },

    // editNote - set up state for editing note
    editNote: function(note) {
      console.log('editNote() ... ')
      this.isEditing = true
      this.noteToEdit = note
    },

    // submitEditedNote - update notes locally, then broadcast all notes to users
    submitEditedNote: function(note) {
      console.log('submitEditedNote() ... ')
      let editedNote = {
        content: note,
        user: this.username,
        timestamp: Date.now()
      }
      for (let i=0; i<this.notes.length; i++) {
        if (this.notes[i].timestamp === this.noteToEdit.timestamp) {
          this.notes.splice(i, 1, editedNote)
        }
      }
      this.isEditing = false
      this.noteToEdit = {}
      let data = {action: 'replaceNotes', notes: this.notes}
      socket.send(JSON.stringify(data))
    },

    // sendUpdate - send update action with users and notes
    sendUpdate: function(users, notes) {
      console.log('sendUpdate() ... ')
      let data = {action: 'update', users, notes}
      socket.send(JSON.stringify(data))
    },

    // recieveUpdate - merge users and notes with update from socket
    recieveUpdate: function(data) {
      console.log('recieveUpdate() ... ')
      this.users = this.users.concat(
        data.users.filter(item => this.users.indexOf(item) < 0)
      )
      if (data.notes.length) {
        let notesIndex = []
        for (let i=0; i<this.notes.length; i++) {
          notesIndex.push(this.notes[i].timestamp)
        }
        this.notes = this.notes.concat(
          data.notes.filter(item => notesIndex.indexOf(item.timestamp) < 0)
        )
        this.saveNotes()
      }
    }
  }
}
</script>
