<template>
  <div id="app" class="row">
  <div class="col-6">
    <select-username
      v-if="!usernameAlreadySelected"
      @input="onUsernameSelection"
    />

    <chat v-else />
  </div>
  <div class="col-6">

<Login />
  </div>


  </div>
</template>

<script>
import SelectUsername from "./components/SelectUsername";
import Chat from "./components/Chat";
import Login from "./components/Login";
import socket from "./socket";

export default {
  name: "App",
  components: {
    Chat,
    SelectUsername,
    Login
  },
  data() {
    return {
      usernameAlreadySelected: false,
    };
  },
  methods: {
    onUsernameSelection(username) {
      this.usernameAlreadySelected = true;
      socket.auth = { username };
      socket.connect();
    },
  },
  created() {
    const sessionID = localStorage.getItem("sessionID");

    if (sessionID) {
      this.usernameAlreadySelected = true;
      socket.auth = { sessionID };
      socket.connect();
    }

    socket.on("session", ({ sessionID, userID }) => {
      // attach the session ID to the next reconnection attempts
      socket.auth = { sessionID };
      // store it in the localStorage
      localStorage.setItem("sessionID", sessionID);
      // save the ID of the user
      socket.userID = userID;
    });

    socket.on("connect_error", (err) => {
      if (err.message === "invalid username") {
        this.usernameAlreadySelected = false;
      }
    });
  },
  destroyed() {
    socket.off("connect_error");
  },
};
</script>

<style>
body {
  margin: 10;
}

@font-face {
  font-family: Lato;
  src: url("/fonts/Lato-Regular.ttf");
}

#app {
  font-family: Lato, Arial, sans-serif;
  font-size: 14px;
}
</style>
