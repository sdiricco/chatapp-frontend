<template>
  <v-app id="inspire">
    <v-app-bar app clipped-right flat height="72">
      <v-spacer></v-spacer>
    </v-app-bar>

    <v-navigation-drawer app clipped right>
      <v-list>
        <v-list-item>
          <v-list-item-content>
            Online Users ({{ this.usersId.length }})
          </v-list-item-content>
        </v-list-item>
        <v-list-item v-for="uId in usersId" :key="uId" link>
          <v-list-item-content>
            <v-list-item-title v-if="uId === id" class="blue--text">{{
              toShortId(uId)
            }}</v-list-item-title>
            <v-list-item-title v-else>{{ toShortId(uId) }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <div v-if="messages.length" :ref="this.uuid" class="section">
        <template v-for="m in messages">
          <div
            :class="[
              m.id === id ? 'align-right' : 'align-left',
              'mb-2',
              'item',
            ]"
            :key="m.timestamp + '_ul'"
          >
            <div class="item-header">
              {{ toShortId(m.id) }} - {{ formatData(m.timestamp) }}
            </div>
            <div class="item-content">
              {{ m.message }}
            </div>
          </div>
          <v-spacer :key="m.timestamp + '_spacer'"></v-spacer>
        </template>
      </div>
    </v-main>

    <v-footer app height="72" inset>
      <v-container fluid>
        <v-row>
          <v-col cols="12">
            <v-text-field
              v-model="textInpt"
              @keypress.enter="onSend"
              dense
              flat
              hide-details
              rounded
              solo
            >
              <v-icon slot="append" color="primary" @click="onSend">
                mdi-send
              </v-icon>
            </v-text-field>
          </v-col>
        </v-row>
      </v-container>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  name: "App",

  data: () => ({
    textInpt: "",
    usersId: [],
    drawer: null,
    messages: [],
    id: "",
    uuid: Math.random().toString(36).substr(2, 15),
  }),
  sockets: {
    connect: function () {
      console.log("socket connected");
    },
    allMessages: function (response = { data: [] }) {
      this.messages = response.data;
    },
    newUserConnected: function () {
    },
    allUsers: function (response = { data: [] }) {
      this.usersId = response.data;
    },
    join: function (response = { id: "" }) {
      this.id = response.id;
    },
  },
  methods: {
    onSend() {
      this.$socket.emit("message", { message: this.textInpt });
      this.textInpt = "";
    },
    toShortId(id = "") {
      return id.substring(0, 5);
    },
    formatData(d) {
      const data = new Date(d);
      const hours = data.getHours();
      const minutes = data.getMinutes();
      const seconds = data.getSeconds();

      return `${hours}:${minutes}:${seconds}`;
    },
  },
  updated(){
    this.$refs[this.uuid].scrollTop = this.$refs[this.uuid].scrollHeight
  }
};
</script>

<style>

body {
  min-height: 100vh;
  min-height: -webkit-fill-available;
}
html {
  height: -webkit-fill-available;
}
/* width */
::-webkit-scrollbar {
  width: 10px;
  border-radius: 8px;
}

/* Track */
::-webkit-scrollbar-track {
  /* background: #f1f1f1;  */
}

/* Handle */
::-webkit-scrollbar-thumb {
  background: #272727;
  border-radius: 4px;
}

/* Handle on hover */
::-webkit-scrollbar-thumb:hover {
  /* background: #555;  */
}

.section {
  overflow-y: auto;
  max-height: calc(100vh - 72px - 72px);
  display: flex;
  flex-direction: column;
  padding: 8px;
}

.item {
  max-width: 80%;
  border: 1px solid #272727;
  border-radius: 16px;
  display: flex;
  flex-direction: column;
  padding: 12px;
}

.item-header {
  opacity: 0.6;
  margin-bottom: 8px;
  font-size: 0.9rem;
}

.item-content {
}

.align-right {
  align-self: flex-end;
}

.align-left {
  align-self: flex-start;
}

html {
  overflow-y: hidden !important;
}

.item {
  display: flex;
  justify-content: center;
}
</style>
