<template>
  <v-app id="inspire">
    <router-link v-bind:to="{ name: 'Home' }"> </router-link>

    <v-toolbar color="grey darken-4" dark fixed app>
      <v-toolbar-side-icon>
        <v-img
          src="http://www.bcslogic.com/wp-content/uploads/2018/06/2018.06.06-Brett-feature.png"
        />
      </v-toolbar-side-icon>

      <v-spacer></v-spacer>

      <v-toolbar-items class="hidden-sm-and-down">
          <v-btn v-bind:to="{ name: 'Admin' }" flat>
          <v-list-tile>
            <v-list-tile-content id="Admin">Admin</v-list-tile-content>
          </v-list-tile>
        </v-btn>

        <v-btn
          class="btn btn-warning  "
          id="adminLogout_btn"
          flat
          v-if="isLoggedIn"
          @click="handleSignOut"
          >Admin Logout</v-btn
        >

        <!-- <button
          @click="handleSignOut"
          v-if="isLoggedIn"
          type="button"
          class="btn btn-warning btn-lg "
        >
          Sign out
        </button> -->

        <v-btn v-bind:to="{ name: 'Home' }" flat>
          <v-list-tile>
            <v-list-tile-content id="Home">Home</v-list-tile-content>
          </v-list-tile>
        </v-btn>

        <v-btn v-bind:to="{ name: 'Movies' }" flat v-if="current_user">
          <v-list-tile>
            <v-list-tile-content id="Movies">Movies</v-list-tile-content>
          </v-list-tile>
        </v-btn>

        <v-btn v-bind:to="{ name: 'Contact' }" flat v-if="current_user">
          <v-list-tile>
            <v-list-tile-content id="contact">Contact</v-list-tile-content>
          </v-list-tile>
        </v-btn>

        <v-btn id="user_email" flat v-if="current_user">{{
          current_user.email
        }}</v-btn>
        <v-btn flat v-bind:to="{ name: 'Register' }" v-if="!current_user">
          Register
        </v-btn>
        <v-btn flat v-bind:to="{ name: 'Login' }" v-if="!current_user"
          >Login</v-btn
        >
        <v-btn id="logout_btn" flat v-if="current_user" @click="logout"
          >Logout</v-btn
        >

        </v-toolbar-items>

        <v-menu class="hidden-md-and-up">
          <v-toolbar-side-icon slot="activator"></v-toolbar-side-icon>
          <v-list>

            <v-list-tile>
              <v-list-tile-content>
                <v-btn v-bind:to="{ name: 'Admin' }" flat ><v-list-tile-content id="Admin">Admin</v-list-tile-content></v-btn>
              </v-list-tile-content>
            </v-list-tile>

            <v-list-tile v-if="isLoggedIn">
              <v-list-tile-content>
                <v-btn  @click="handleSignOut"  class="btn btn-warning  " flat ><v-list-tile-content >Admin Logout</v-list-tile-content></v-btn>
              </v-list-tile-content>
            </v-list-tile>

            <v-list-tile>
              <v-list-tile-content>
                <v-btn v-bind:to="{ name: 'Home' }" flat><v-list-tile-content id="Home">Home</v-list-tile-content></v-btn>
              </v-list-tile-content>
            </v-list-tile>

            <v-list-tile v-if="current_user">
              <v-list-tile-content>
                <v-btn v-bind:to="{ name: 'Movies' }" flat ><v-list-tile-content id="Movies">Movies</v-list-tile-content></v-btn>
              </v-list-tile-content>
            </v-list-tile>

            <v-list-tile v-if="current_user">
              <v-list-tile-content>
                <v-btn v-bind:to="{ name: 'Contact' }" flat ><v-list-tile-content id="Contact">Contact</v-list-tile-content></v-btn>
              </v-list-tile-content>
            </v-list-tile>

             <v-list-tile v-if="current_user">
              <v-list-tile-content>
                <v-btn flat ><v-list-tile-content >{{current_user.email}}</v-list-tile-content></v-btn>
              </v-list-tile-content>
            </v-list-tile>

            <v-list-tile v-if="!current_user">
              <v-list-tile-content>
                <v-btn v-bind:to="{ name: 'Register' }" v-if="!current_user" flat ><v-list-tile-content >Register</v-list-tile-content></v-btn>
              </v-list-tile-content>
            </v-list-tile>

             <v-list-tile v-if="!current_user">
              <v-list-tile-content>
                <v-btn v-bind:to="{ name: 'Login' }" flat ><v-list-tile-content >Login</v-list-tile-content></v-btn>
              </v-list-tile-content>
            </v-list-tile>

            <v-list-tile v-if="current_user">
              <v-list-tile-content>
                <v-btn @click="logout" flat ><v-list-tile-content >Logout</v-list-tile-content></v-btn>
              </v-list-tile-content>
            </v-list-tile>

          </v-list>
        </v-menu>
      </v-toolbar>

    <v-content>
      <v-container fluid>
        <div id="app">
          <router-view />
        </div>
      </v-container>
    </v-content>
    <v-footer
      default
      padless
      tile
      class="grey darken-4 white--text text-center"
    >
      <v-spacer></v-spacer>
      <v-btn v-for="icon in icons" :key="icon" class="mx-4 white--text" icon>
        <v-icon size="24px">
          {{ icon }}
        </v-icon>
      </v-btn>
      <v-spacer></v-spacer>
      <div>&copy; {{ new Date().getFullYear() }}</div>
    </v-footer>
  </v-app>
</template>

<script>
import { getAuth, onAuthStateChanged, signOut } from "firebase/auth";
import axios from "axios";
import "./assets/stylesheets/main.css";
import bus from "./bus";

let auth;
export default {
  name: "app",
  data: () => ({
    isLoggedIn: null,
    drawer: null,
    current_user: null,
    icons: ["mdi-facebook", "mdi-twitter", "mdi-linkedin", "mdi-instagram"]
  }),

  props: {
    source: String
  },
  mounted() {
    this.fetchUser();
    this.listenToEvents();

    auth = getAuth();
    onAuthStateChanged(auth, user => {
      if (user) {
        this.isLoggedIn = true;
      } else {
        this.isLoggedIn = false;
      }
    });
  },
  methods: {
    listenToEvents() {
      bus.$on("refreshUser", () => {
        this.fetchUser();
      });
    },
    async fetchUser() {
      return axios({
        method: "get",
        url: "/api/current_user"
      })
        .then(response => {
          this.current_user = response.data.current_user;
        })
        .catch(() => {});
    },
    logout() {
      return axios({
        method: "get",
        url: "/api/logout"
      })
        .then(() => {
          bus.$emit("refreshUser");
          this.$router.push({ name: "Home" });
        })
        .catch(() => {});
    },

    handleSignOut() {
      signOut(auth).then(() => {
        this.$router.push({ name: "AdminLogin" });
      });
    }
  }
};
</script>
