<template>
    <div id="playlists" v-if="$store.state.showPlaylists">
        <h2>Playlists</h2>
        <button 
            type="button" 
            class="btn btn-outline-dark btn-sm custom-style" 
            @click="$store.dispatch('setShowAddPlaylistComponent', true);">
            new playlist
        </button>
        <button 
            type="button" 
            class="btn btn-outline-dark btn-sm custom-style" 
            @click="$store.dispatch('setShowAddRandomPlaylistComponent', true);">
            new random playlist
        </button>
        <p></p>
        <ul class="list-group">
            <li class="list-group-item" v-for="playlist in this.$store.state.playlistNames">
                {{playlist.name}}
                <button
                  type="button"
                  class="btn btn-secondary btn-sm custom-style"
                  @click="$store.dispatch('setShowPlaylistEditComponent', true); 
                  $store.dispatch('setPlaylistEditModalValue', playlist);
                  loadThePlaylistBeingEdited();
                  ">
                  manage
                </button>
                <button
                  type="button"
                  class="btn btn-outline-dark btn-sm custom-style"
                  @click="$store.dispatch('setCurrentPlaylist', playlist.name); $store.dispatch('showSection', 'musicplayer')">
                  load
                </button>
            </li>
        </ul>
        <EditPlaylistName />
        <AddPlaylist />
        <AddRandomPlaylist />
    </div> 
</template>

<script>
import axios from 'axios'
import EditPlaylistName from './EditPlaylistName.vue';
import AddPlaylist from './AddPlaylist.vue';
import AddRandomPlaylist from './AddRandomPlaylist.vue';
export default {
    mounted: function() { 
        this.$store.dispatch('setPlaylistNames');       
    },
    components: {
        EditPlaylistName,
        AddPlaylist,
        AddRandomPlaylist
    },
    data: function() {
        return {
            thePlaylistBeingEdited: []
        }
    },
    methods: {
        loadThePlaylistBeingEdited: function() {

            let config = {
                headers : {
                'Authorization' : 'Token '+process.env.VUE_APP_SONG_API_KEY
                }
            }
            axios
            .get('http://'+process.env.VUE_APP_SONG_API_IPADDRESS+'/songapi/playlistsong/?search='+this.$store.state.playlistToEdit.name, config)
            .then(response => {
                this.thePlaylistBeingEdited = response.data
            })
            .catch(error => {
                console.log('loadThePlaylistBeingEdited:', error);
            })            

        },
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
    },
    computed: {
        playlistToManage() {

        //   element<Object>:
        //     album<Object>: id, name, url
        //     artist<Object>: id, name, url
        //     id
        //     remote_url
        //     title
        //     url
         
        let returnData = []
        this.thePlaylistBeingEdited.forEach(element => {
            returnData.push(element.song)
        });

        return returnData;

    }
    }
}
</script>

<style>
.custom-style {
    margin-left: 10px;
}
</style>
