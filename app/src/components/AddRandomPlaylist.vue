<template>
  <div v-if="$store.state.showAddRandomPlaylistComponent">
    <transition name="modal-playlist">
        <div class="modal-mask-playlist">
        <div class="modal-wrapper-playlist">
            <div class="modal-container-playlist">
            <div class="modal-body-playlist">
                <slot name="body-playlist">
                </slot>
            </div>
            <div>
            
                <form v-on:submit.prevent="addRandomPlaylist()">
                    <div class="form-group">
                        <div class="col-auto my-1">
                            <input
                                type="text"
                                class="form-control"
                                ref="randomplaylistname"
                                placeholder="Enter Playlist Name"
                                required="required" >
                        </div>
                        <div class="col-auto my-1">
                            <select class="custom-select mr-sm-2" ref="randomplaylistamount">
                                <option selected>amount...</option>
                                <option value="10">10</option>
                                <option value="20">20</option>
                                <option value="50">50</option>
                                <option value="100">100</option>
                            </select>
                        </div>
                    </div>
                    <button type="submit" class="btn btn-primary">Submit</button>
                </form>

            </div>

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
import axios from 'axios'

export default {

    methods: {
        closeModal() {
        this.$store.dispatch('setShowAddRandomPlaylistComponent', false)
        },
        addRandomPlaylist: function() {
        let config = {
            headers : {
            'Content-Type' : 'multipart/form-data',
            'Authorization' : 'Token '+process.env.VUE_APP_SONG_API_KEY
        }
        }
        var upload = new FormData();
        upload.append('name', this.$refs.randomplaylistname.value);
        upload.append('amount', this.$refs.randomplaylistamount.value);
        axios.defaults.xsrfCookieName = 'csrftoken'
        axios.defaults.xsrfHeaderName = "X-CSRFTOKEN"
        axios
            .post('http://'+process.env.VUE_APP_SONG_API_IPADDRESS+'/songapi/randomplaylist/', upload, config)
            .then(response => {
                this.$store.dispatch('setPlaylistNames');
            })
            .catch(error => {
                console.log('addRandomPlaylist:', error)
                
            })
        }
    }

}
</script>

<style>
@import '../assets/css/modal-playlist.css';
</style>