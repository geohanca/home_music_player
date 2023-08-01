<template>
    <div id="currentplaylist" v-if="$store.state.showCurrentPlaylist">
        <div>
            <p style="color: black; background-color: #80b3ff;">current playlist: -- {{ this.$store.state.currentPlaylistName }} --</p>
            <ul class="list-group">
                <li class="list-group-item" v-for="item in this.$store.state.currentPlaylist">
                    <span>{{item.song.title}}</span>
                </li>
            </ul>
        </div>
    </div> 
</template>

<script>
import axios from 'axios'

export default {
    methods: {
        deletePlaylistSong: async function(playlistsong) {
            let config = {
                headers : {
                'Authorization' : 'Token '+process.env.VUE_APP_SONG_API_KEY
                }
            }
            axios.defaults.xsrfCookieName = 'csrftoken'
            axios.defaults.xsrfHeaderName = "X-CSRFTOKEN"
            axios
                .delete(playlistsong.url, config)
                .then(response => {
                    this.$store.dispatch('setCurrentPlaylist', playlistsong.playlist.name)
                })
                .catch(error => {
                  console.log('deletePlaylistSong:', error)
                })
        }
    }

}
</script>

<style>
.playlist-div {
    justify-content: center;
    overflow: scroll;
    height: 300px;
}
</style>
