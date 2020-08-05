<template>
    <div id="app">
        <canvas class="album" id="canvas" width="250px" height="250px"/>
        <Spotify class="spotify" v-if="spotifyAPI" :spotify="spotifyAPI"></Spotify>
        <div class="lds-ring" v-else><div></div><div></div><div></div><div></div></div>
        <footer v-if="spotifyAPI">Tomasz Wosinski {{ currentYear }} © Wszelkie prawa zastrzeżone</footer>
    </div>
</template>

<script>
    import Spotify from './components/Spotify.vue'
    import * as querystring from "querystring";

    require('dotenv').config()

    export default {
        name: 'App',
        components: {
            Spotify
        },
        data: () => {
            return {
                spotifyToken: '',
                vueCanvas: null,
                spotifyGenres: [],
                spotifyAPI: null
            }
        },
        mounted: function () {
            let c = document.getElementById("canvas");
            let ctx = c.getContext("2d");
            this.vueCanvas = ctx;
            this.getAccessToken();
            setInterval(() => {
                this.getSpotifyPlayback();
            }, 5000);
        },
        methods: {
            refreshImage: function () {
                this.vueCanvas.clearRect(0, 0, 300, 300);
                let base_image = new Image();
                base_image.src = this.spotifyAPI.item ? this.spotifyAPI.item.album.images[0].url : '';
                console.log(base_image.src)
                let base_image1 = new Image();
                base_image1.src = this.tomekImage;
                console.log(this.tomekImage);
                setTimeout(() => {
                    this.vueCanvas.drawImage(base_image, 0, 0, 250, 250);
                }, 500);
                setTimeout(() => {
                    this.vueCanvas.drawImage(base_image1, 0, 0, 250, 250);
                }, 500);
            },
            getAccessToken: function () {
                let that = this;
                const axios = require('axios').default;
                axios.post('https://accounts.spotify.com/api/token', querystring.stringify({
                    'grant_type': 'refresh_token',
                    'redirect_uri': 'http://localhost:8080',
                    'refresh_token': process.env.VUE_APP_SPOTIFY,
                }), {
                    headers: {
                        'Content-Type': 'application/x-www-form-urlencoded',
                        'Authorization': `Basic YzBkMmRhYThjZDU4NGY0ZWI4MDU0YWUyODljZTNjMzY6MTc4ZGM3MjQwMGJlNDBlYmEzOWY2MTc0ZWVkZTRhZDQ=`
                    }
                })
                    .then(({data}) => {
                        that.spotifyToken = data.access_token;
                        that.getSpotifyPlayback();
                    })
                    .catch(function (error) {
                        // handle error
                        console.log(error);
                    })
            },
            getArtist: function () {
                if (this.spotifyAPI) {
                    var that = this;
                    const axios = require('axios').default;
                    axios.get(`https://api.spotify.com/v1/artists/${this.spotifyAPI ? this.spotifyAPI.item.artists[0].id : ''}`, {
                        headers: {
                            'Authorization': `Bearer ${this.spotifyToken}`
                        }
                    })
                        .then(({data}) => {
                            that.spotifyGenres = data.genres;
                            that.refreshImage()
                        })
                        .catch(function () {
                            // handle error
                            that.getAccessToken();
                        })
                }
            },
            getSpotifyPlayback: function () {
                var that = this;
                const axios = require('axios').default;
                axios.get('https://api.spotify.com/v1/me/player', {
                    headers: {
                        'Authorization': `Bearer ${this.spotifyToken}`
                    }
                })
                    .then(({data}) => {
                        if (!that.spotifyAPI || (data && that.spotifyAPI && (data.item.id !== that.spotifyAPI.item.id))) {
                            that.spotifyAPI = data;
                            that.getArtist();
                        }
                    })
                    .catch(function () {
                        // handle error
                        that.getAccessToken();
                    })
            },
        },
        computed: {
            tomekImage: function () {
                const genres = this.spotifyGenres.join(',');
                return '/' + (this.isListening ?
                    (genres.includes('hardcore') || genres.includes('rock') || genres.includes('metal') ?
                        'rock.png' : 'music.png') :
                    'still.png');
            },
            isListening() {
                return this.spotifyAPI ? this.spotifyAPI.is_playing : false;
            },
            currentYear() {
                const d = new Date();
                return d.getFullYear();
            }
        }
    }
</script>

<style>
    html {
        height: 100%;
        background-color: #191414;
        box-sizing: border-box;
    }

    #app {
        display: block;
        align-items: center;
        vertical-align: middle;
        font-family: 'Montserrat', sans-serif;
        color: white;
        box-sizing: border-box;
        text-align: center;
    }

    .album {
        border: 2px solid #ccc;
    }

    footer {
        text-align: center;
        width: 100%;
        font-size: 10px;
    }

    .lds-ring {
        display: inline-block;
        position: relative;
        width: 80px;
        height: 80px;
    }

    .lds-ring div {
        box-sizing: border-box;
        display: block;
        position: absolute;
        width: 64px;
        height: 64px;
        margin: 8px;
        border: 8px solid #fff;
        border-radius: 50%;
        animation: lds-ring 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
        border-color: #fff transparent transparent transparent;
    }

    .lds-ring div:nth-child(1) {
        animation-delay: -0.45s;
    }

    .lds-ring div:nth-child(2) {
        animation-delay: -0.3s;
    }

    .lds-ring div:nth-child(3) {
        animation-delay: -0.15s;
    }

    @keyframes lds-ring {
        0% {
            transform: rotate(0deg);
        }
        100% {
            transform: rotate(360deg);
        }
    }

</style>
