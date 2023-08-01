<template>
    <div id="musicplayer" v-if="$store.state.showMusicPlayer">
         
        <div class="container-fluid">
            <div class="row">
                <div class="col"></div>
                <div class="col-8 justify-content-center">
                    <h5>{{ songTitle.split(".")[0] }}</h5>
                </div>
                <div class="col"></div>
            </div>
            <div class="row">
                <div class="col"></div>
                <div class="col-8 justify-content-center">
                    <span >{{ songArtist }}</span>
                </div>
                <div class="col"></div>
            </div>
            <p></p>
            <div class="row">
                <div class="col-2 justify-content-end" ref="currentTime" id="currentTime">00:00</div>
                <div class="col-8 justify-content-center">
                    <input
                    class="container-fluid"
                    type="range"
                    ref="progressBar"
                    id="progress-bar"
                    min="0"
                    max=""
                    value="0"
                    v-on:change="changeProgressBar()"
                    /> 
                </div>
                <div class="col-2 justify-content-end" ref="durationTime" id="durationTime">00:00</div>
            </div>
            <p></p>
            <div class="row">
                <div class="col-2"></div>
                <div class="col-2 justify-content-end">
                    <font-awesome-icon icon="step-backward" size="2x"  v-on:click="previousSong()" />
                </div>
                <div class="col-4 text-center">
                    <font-awesome-icon :icon="playIcon" size="3x" v-on:click="play()" />
                </div>
                <div class="col-2 justify-content-start">
                    <font-awesome-icon icon="step-forward" size="2x" v-on:click="nextSong()" />
                </div>
                <div class="col-2"></div>
                <audio ref="mp3Player" preload="none"></audio>

            </div>
            <div class="row">
                <div class="col col-2"></div>
                <div class="col-6">
                    <input id="shuffle" type="checkbox" v-model="shuffle" label="shuffle">
                    <label class="custom-left-margin" for="shuffle">shuffle</label>
                </div>
                <div class="col col-2"></div>
            </div>
            
            <current-playlist />    
        
        </div>  

    </div>
</template>

<script>
import CurrentPlaylist from './CurrentPlaylist.vue';

export default {
  data: function() {
    
      return {
        musicPlayerKey: 0,
        playIcon: 'play',
        shuffle: false,
        songLoaded: false,
        songTitle: '---',
        songArtist: '---',
        playing: false,
        playOrderIndex: 0, 
        currentSong: new Audio()
      }

  },
  components: {
      CurrentPlaylist
  },
  methods: {
    forceRerender: function() {
        this.musicPlayerKey += 1; //updating the key will force Vue to rerender component, including computed values
    },
    loadSong() {

        try {

            if (this.currentPlaylist[this.playOrder[this.playOrderIndex]]) {

                this.songLoaded = true;

                let selectedItem = this.currentPlaylist[this.playOrder[this.playOrderIndex]];
                if (selectedItem) {
                    this.songTitle = selectedItem.title
                    this.songArtist = selectedItem.artist.name
                    this.currentSong.src = selectedItem.remote_url; //in the resetSong() function the audio object is reset explicitly to not have a source file
                    this.currentSong.onended = (event) => {
                        if (this.playOrderIndex == (this.currentPlaylist.length - 1) ) {
                            this.resetSong();
                        } else {
                            this.nextSong();
                        };
                    }; 
                    this.songLoaded = true;                  
                }
                
            }
            
        } catch (error) {

            console.log('loadSong:', error)
            
        }

    },
    play: function() {
        
        if (this.playing == true) {
            this.pauseSong();
        } else {
            if (this.songLoaded) {
                this.playSong();
            } else {
                this.loadSong();
                this.playSong();
            }
            
        }
    },
    playSong: function() {
        this.playIcon = 'pause';
        this.currentSong.play();
        this.playing = true;
        this.startInterval();
    },
    pauseSong: function() {
        this.playIcon = 'play';
        this.currentSong.pause();
        this.playing = false; 
        this.stopInterval();
    },
    nextSong: function() {

        this.resetSong()
        
        this.playOrderIndex++;
        
        if (this.playOrderIndex > (this.currentPlaylist.length - 1) ) {
            this.playOrderIndex = 0;
        };

        this.loadSong()
        this.play()

    },
    previousSong: function() {

        this.resetSong()

        this.playOrderIndex--;

        if (this.playOrderIndex < 0) {
            this.playOrderIndex = (this.currentPlaylist.length - 1);
        };
        
        this.loadSong()
        this.play()

    },
    resetSong: function() {
        
            this.currentSong.pause(); //first three lines reset the audio object
            this.currentSong.removeAttribute('src');
            this.currentSong.load();
            this.playIcon = 'play';
            this.playing = false;
            this.stopInterval();
            this.$refs.progressBar.value = 0;
            this.$refs.currentTime.innerHTML = (this.formatTime(Math.floor(0)));

    },
    updateProgressValue: function() {
        this.$refs.progressBar.max = this.currentSong.duration;
        this.$refs.progressBar.value = this.currentSong.currentTime;
        this.$refs.currentTime.innerHTML = (this.formatTime(Math.floor(this.currentSong.currentTime)));
        if (this.$refs.durationTime.innerHTML === "NaN:NaN") {
            this.$refs.durationTime.innerHTML = "0:00";
        } else {
            this.$refs.durationTime.innerHTML = (this.formatTime(Math.floor(this.currentSong.duration)));
        }
    },
    formatTime: function(seconds) {
        let min = Math.floor((seconds / 60));
        let sec = Math.floor(seconds - (min * 60));
        if (sec < 10){ 
            sec  = `0${sec}`;
        };
        return `${min}:${sec}`;
    },
    changeProgressBar: function() {
        this.currentSong.currentTime = this.$refs.progressBar.value;
        this.$refs.currentTime.innerHTML = (this.formatTime(Math.floor(this.currentSong.currentTime)));
    },
    startInterval: function() {
        this.intervalNum = setInterval(this.updateProgressValue, 500);            
    },
    stopInterval: function() {
        clearInterval(this.intervalNum);
    }
  },
  computed: {
    currentPlaylist() {

        //   element<Object>:
        //     album<Object>: id, name, url
        //     artist<Object>: id, name, url
        //     id
        //     remote_url
        //     title
        //     url
         
        let returnData = []
        this.$store.state.currentPlaylist.forEach(element => {
            returnData.push(element.song)
        });

        return returnData;

    },
    playOrder() {

        try {

            let returnData = []
            if (this.shuffle) {

                for (var i = 0; i <= (this.currentPlaylist.length - 1); i++) {
                    returnData.push(i);
                }

                //randomly sort shuffle array using Fisher-Yates Algorithm
                for (let i = (returnData.length - 1); i > 0; i--) {
                    const j = Math.floor(Math.random() * i);
                    const temp = returnData[i];
                    returnData[i] = returnData[j];
                    returnData[j] = temp;
                }
                
            } else {
                
                for (var i = 0; i <= (this.currentPlaylist.length - 1); i++) {
                    returnData.push(i);
                }

            }
            
            return returnData
            
        } catch (error) {

            return error
            
        }

    }
  }
    
}
</script>

<style>

.custom-left-margin {
    margin-left: 10px;
}

.custom-right-margin {
    margin-right: 10px;
}

</style>
