<template>
  <v-app>
    <v-toolbar color="#9D6447" style="color:white;" app>
      <v-toolbar-title class="headline text-uppercase">
        <span>Wallet</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-menu
        transition="slide-y-transition"
        bottom
      >
        <template v-slot:activator="{ on }">
          <v-btn
            flat
            dark
            v-on="on"
          >
            {{ userName }} <v-icon v-if="isLoggedIn">keyboard_arrow_down</v-icon>
          </v-btn>
        </template>
        <v-list v-if="isLoggedIn">
          <v-list-tile @click="openSettings">
            <v-icon style="margin-right:10px;">settings</v-icon>
            <v-list-tile-title>{{ $t("settings.settings") }}</v-list-tile-title>
          </v-list-tile>
          <v-list-tile v-if="!isAnonymous" @click="logout">
            <v-icon style="margin-right:10px;">power_settings_new</v-icon>
            <v-list-tile-title>{{ $t("app.logout") }}</v-list-tile-title>
          </v-list-tile>
        </v-list>
      </v-menu>
    </v-toolbar>

    <v-content>
      <transition
        name="fade"
        mode="out-in"
      >
        <router-view/>
      </transition>
      <footer>
        <p>{{ $t("general.moreAppsOn") }}
          <a href="https://www.schmuckli.dev/" target="_blank">schmuckli.dev</a></p>
      </footer>
    </v-content>

    <!-- Notification -->
    <v-snackbar
      v-model="notification"
    >
      {{ $t(notification_text) }}
      <v-btn
        color="#9D6447"
        flat
        @click="notification = ''"
      >
        {{ $t("general.close") }}
      </v-btn>
    </v-snackbar>
    <v-btn fab dark color="#9D6447" v-if="isLoggedIn" fixed right bottom>
        <v-icon @click="openNew" dark>add</v-icon>
    </v-btn>
  </v-app>
</template>

<script>
import { Store, StoreMod } from "./store.js";
import firebase from "firebase";

export default {
  name: 'App',
  data() {
    return {
      currentUser: firebase.auth().currentUser
    }
  },
  computed: {
    /* Notification */
    notification: {
      get() { return Store.notification !== "" },
      set() { StoreMod.showNotification(""); }
    },
    notification_text(){
      return Store.notification;
    },
    /* Notificatio End */
    userName(){
      if(!this.isLoggedIn){
        return "";
      }
      if(this.currentUser.isAnonymous){
        return "Guest";
      }
      if(this.currentUser.displayName == undefined){
        return this.currentUser.email;
      }else{
        return this.currentUser.displayName;
      }
    },
    isAnonymous(){
      return this.currentUser.isAnonymous;
    },
    isLoggedIn(){
      return this.currentUser != undefined;
    }
  },
  methods: {
    openSettings(){
      this.$router.replace("settings");
    },
    openNew(){
      if(navigator.onLine){
        this.$router.replace("new");
      } else {
        StoreMod.showNotification("notification.youOnlyCanAddPassesWhileOnline");
      }
    },
    logout(){
      if(navigator.onLine){
        var global_this = this;
        firebase.auth().signOut().then(function() {
          global_this.$router.replace("login");
          StoreMod.showNotification("notification.logoutWasSuccessful");
        }, function() {
          StoreMod.showNotification("notification.logoutWasNotSuccessful");
        });
      } else {
        StoreMod.showNotification("notification.youCantLogoutWhileOffline");
      }
    }
  },
  watch:{
    //$route (to, from){
    $route (){ //Call everytime when route changes
      this.currentUser = firebase.auth().currentUser;
    }
  }
}
</script>

<style>
@import "./assets/css/layout.css";
@import "./assets/css/animate.css";

footer{
  text-align:center;
  margin-top:30px;
  color:grey;
}
footer a{
  color:grey;
}

h1{
  margin-bottom: 20px;
}

.fade-enter-active,
.fade-leave-active {
  transition-duration: 0.3s;
  transition-property: opacity;
  transition-timing-function: ease;
}

.fade-enter,
.fade-leave-active {
  opacity: 0
}
</style>
