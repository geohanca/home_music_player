<template>
  <div id="app">

    <div class="wrapper">
      <!-- Sidebar  -->
      <nav id="sidebar" v-bind:class="{active: isActive}">
          <div class="sidebar-header">
              <h3>Music App</h3>
          </div>

          <ul class="list-unstyled components">
              <li>
                  <a class="navbar-brand" @click="$store.dispatch('showSection', 'artists')" href="#">Artists</a>
              </li>
              <li>
                  <a class="navbar-brand" @click="$store.dispatch('showSection', 'playlists')" href="#">Playlists</a>
              </li>
              <li>
                  <a class="navbar-brand" @click="$store.dispatch('showSection', 'musicplayer')" href="#">Music Player</a>
              </li>
              <li>
                  <a class="navbar-brand" @click="$store.dispatch('showSection', 'settings')" href="#">Settings</a>
              </li>
          </ul>

      </nav>

      <!-- Page Content  -->
      <div id="content">

          <nav class="navbar navbar-expand-lg navbar-light bg-light">
              <div class="container-fluid">

                  <button type="button" id="sidebarCollapse" class="btn btn-info" @click="sidebarCollapse()">
                      <font-awesome-icon icon="align-left" />
                  </button>

              </div>
          </nav>

          <current-artist />
          <artists />
          <playlists />
          <music-player />
          <settings />
          
      </div>
    </div>

  </div>

</template>

<script>
import Playlists from './components/Playlists.vue';
import CurrentArtist from './components/CurrentArtist.vue';
import Artists from './components/Artists.vue';
import MusicPlayer from './components/MusicPlayer.vue';
import Settings from './components/Settings.vue';

export default {
  name: 'App',
  data: function() {
    return {
      isActive: false
    }
  },
  methods: {
    sidebarCollapse: function() {
      this.isActive = !this.isActive;
    }
  },
  components: {
    Playlists,
    CurrentArtist,
    Artists,
    MusicPlayer,
    Settings
  }
}
</script>

<style>
@import './assets/css/GooglePoppins.css';
@import './assets/css/App.css';

.bottom {
  margin-bottom: 75px;
}
</style>
