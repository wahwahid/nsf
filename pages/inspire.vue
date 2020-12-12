<template>
  <v-row>
    <v-col class="text-center">
      <img src="/v.png" alt="Vuetify.js" class="mb-5" />
      <h2>Notif {{ notif }}</h2>
      <h2>socketConnect {{ socketConnect }}</h2>
      <v-btn v-if="!notif" @click="notifSetup">Turn On Notification</v-btn>
      <blockquote class="blockquote">
        &#8220;First, solve the problem. Then, write the code.&#8221;
        <footer>
          <small>
            <em>&mdash;John Johnson</em>
          </small>
        </footer>
      </blockquote>
    </v-col>
  </v-row>
</template>
<script>
export default {
  data() {
    return {
      notif: false,
      socket: null,
      socketConnect: false,
    }
  },
  mounted() {
    this.notifSetup()
  },
  methods: {
    async notifSetup() {
      const granted = await this.notifRequestAndShowPermission()
      if (granted) {
        this.notifConnectSocket()
      }
    },
    async notifRequestAndShowPermission() {
      this.notif = false
      if (!('Notification' in window)) {
        alert('This browser does not support desktop notification')
      } else if (Notification.permission === 'granted') {
        this.notif = true
      } else if (Notification.permission !== 'denied') {
        const permission = await Notification.requestPermission()
        if (permission === 'granted') {
          this.notif = true
        }
      }
      return this.notif
    },
    notifConnectSocket() {
      this.socket = this.$nuxtSocket({
        path: '/ws',
      })
      this.socket.on('connect_error', () => {
        this.socketConnect = false
      })
      this.socket.on('connect', () => {
        this.socketConnect = true
        this.socket.on('disconnect', () => {
          this.socketConnect = false
        })
        this.socket.on('notif-user', (payload) => {
          this.notifShow({
            title: 'Halo User',
            body: payload.body || 'Kosong',
            href: '?user=' + Date.now(),
          })
        })
        this.socket.on('notif-admin', (payload) => {
          this.notifShow({
            title: 'Halo Admin',
            body: payload.body || 'Kosong',
            href: '?admin=' + Date.now(),
          })
        })
      })
    },
    notifShow({ title, body, href }) {
      const icon = '/icon.png'
      const notification = new Notification(title, { body, icon })
      notification.onclick = () => {
        if (href) {
          window.open(href)
        } else {
          notification.close()
          window.parent.focus()
        }
      }
    },
  },
}
</script>
