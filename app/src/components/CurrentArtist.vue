<template>
    <div class="collapse multi-collapse" :id="divId">
        <button type="button" class="btn btn-outline-primary btn-sm" v-on:click="setCurrentArtist()">refresh artist songs</button>
        <div class="alert alert-primary" role="alert" v-if="this.refreshInProgress">
        Refresh in-progress. Please wait...
        </div>
        <p></p>
        <ul class="list-group">
            <li class="list-group-item" v-for="item in currentArtistFiles">
                <span class="custom-style">{{ item.title }}</span>
                <button
                  type="button"
                  class="btn btn-secondary btn-sm custom-style"
                  @click="$store.dispatch('setShowAddSongToPlaylistComponent', true); $store.dispatch('setSongToAddToPlaylist', item);">
                  +
                </button>
            </li>
        </ul>
        <AddSongToPlaylist />
    </div>
</template>

<script>
import axios from 'axios';
import AddSongToPlaylist from './AddSongToPlaylist.vue';

export default {
    props: ['divId', 'currentArtist'],
    components: {
        AddSongToPlaylist
    },
    data: function() {
        return {
            currentArtistKey: 0,
            currentArtistFiles: [],
            refreshInProgress: false
        }
    },
    methods: {
        forceRerender: function() {
            this.currentArtistKey += 1; //updating the key will force Vue to rerender component, including computed values
        },
        setCurrentArtist() {
            this.refreshInProgress = true;
            let config = {
                headers : {
                'Authorization' : 'Token '+process.env.VUE_APP_SONG_API_KEY
                }
            }
            axios
            .get('http://'+process.env.VUE_APP_SONG_API_IPADDRESS+'/songapi/song/?search='+this.currentArtist, config)
            .then(response => {
                this.currentArtistFiles = response.data;
                this.refreshInProgress = false;
            })
            .catch(error => {
                console.log('setCurrentArtist:', error);
            })            
        }
    }
}
</script>

<style>
.custom-style {
    margin-left: 10px;
}
</style>
