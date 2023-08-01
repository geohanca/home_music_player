<template>
  <div id="addtoplaylist" :key="addToPlaylistKey" v-if="$store.state.showAddSongToPlaylistComponent">
    <transition name="modal-playlist">
      <div class="modal-mask-playlist">
        <div class="modal-wrapper-playlist">
          <div class="modal-container-playlist">
            <div class="modal-body-playlist">
              <slot name="body-playlist">
              </slot>
            </div>
            <hr>

            <div>
            
            <select v-model="selectedPlaylist">
              <option  v-for="playlist in allplaylists" :value=playlist>
                {{playlist.name}}
              </option>
            </select>
            <p></p>
            <button type="button" class="btn btn-secondary" v-on:click="addPlaylistSongEntry()">Add to Playlist</button>
            </div>
            <p></p>
            <div>{{message}}</div>
            <p></p>
            <div class="modal-footer-playlist">
              <slot name="footer-playlist">
                <button class="modal-default-button-playlist" @click="closeModal()">
                  OK
                </button>
              </slot>
            </div>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data: function() {
    return {
      addToPlaylistKey: 0,
      message: "---",
      allplaylists: [],
      selectedPlaylist: {},
    }
  },
  mounted: function() {
    this.getPlaylistNames(); 
  },
  methods: {
    getPlaylistNames: function() {
      let config = {
          headers : {
          'Authorization' : 'Token '+process.env.VUE_APP_SONG_API_KEY
        }
      }
      axios
      .get('http://'+process.env.VUE_APP_SONG_API_IPADDRESS+'/songapi/playlist/', config)
      .then(response => {
          this.allplaylists = response.data;
          //console.log(this.playList);
      })
      .catch(error => {
          console.log(error);
          this.errored = true;
      })            
   },
   addPlaylistSongEntry: function() {
     //console.log(this.song)
      let config = {
          headers : {
          'Content-Type' : 'multipart/form-data',
          'Authorization' : 'Token '+process.env.VUE_APP_SONG_API_KEY
      }
      }
      var upload = new FormData();
      upload.append('playlist_id', this.selectedPlaylist.id);
      upload.append('song_id', this.$store.state.songToAddToPlaylist.id);
      axios.defaults.xsrfCookieName = 'csrftoken'
      axios.defaults.xsrfHeaderName = "X-CSRFTOKEN"
      axios
          .post('http://'+process.env.VUE_APP_SONG_API_IPADDRESS+'/songapi/playlistsong/', upload, config)
          .then(response => {
            this.message = 'added'
            this.$store.dispatch('setSongToAddToPlaylist', {});
            this.forceRerender();
          })
          .catch(error => {
            this.message = 'error - see log'
            this.$store.dispatch('setSongToAddToPlaylist', {});
            // console.log('addPlaylistSongEntry:', error)  
          })
    },
    closeModal() {
      this.$store.dispatch('setShowAddSongToPlaylistComponent', false)
    },
    forceRerender: function() {
      this.addToPlaylistKey += 1; //updating the key will force Vue to rerender component, including computed values
    }
}

}
</script>

<style>
@import '../assets/css/modal-playlist.css';
</style>
