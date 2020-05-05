<template>
    <div id="app">
        <img alt="Album" v-if="spotifyAPI" :src="spotifyAPI ? spotifyAPI.item.album.images[0].url : ''" width="280"
             class="album">
        <img alt="Tomek" :src="tomekImage" width="275" class="avatar">
        <div class="progress_bar" style="width:50%"></div>
        <Spotify :spotify="spotifyAPI"></Spotify>
        <footer>Tomasz Wosinski {{ currentYear }} © Wszelkie prawa zastrzeżone</footer>
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
                spotifyGenres: [],
                spotifyAPI: null
            }
        },
        mounted: function () {
            this.getAccessToken();
            setInterval(() => {
                this.getSpotifyPlayback()
            }, 3000);
        },
        methods: {
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
                        if (data) {
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

    .avatar {
        width: 350px;
        margin-left: auto;
        margin-right: auto;
        vertical-align: middle;
        border: 2px solid #ccc;
        z-index: 0;
    }

    .album {
        position: fixed;
        width: 350px;
        margin-left: auto;
        margin-right: auto;
        vertical-align: middle;
        border: 2px solid #ccc;
        filter: brightness(150%) blur(3px);
        z-index: -1;
    }

    footer {
        position: fixed;
        bottom: 0;
        width: 100%;
        margin: 5px;
        font-size: 10px;
    }
</style>
