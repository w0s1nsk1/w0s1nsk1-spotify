<template>
    <div class="hello">
        <p>
            Tomek słucha{{ spotify && !spotify.is_playing ? 'ł ostatnio' : ' teraz'}}:
        </p>
            <h3>{{ spotify.item.artists.length > 1 ? spotify.item.artists.map((item) => { return item.name }).join(', ')
                : spotify.item.artists[0].name }}</h3>
            <h4>{{ spotify.item.name }}</h4>
        <p style="margin: 45px;"><a v-if="spotify" :href="spotify ? spotify.item.external_urls.spotify : ''">Posłuchaj
            razem z nim</a></p>
        <p style="margin: 45px;"><a v-if="spotify" class="youtube" :href="youtubeLink">Poszukaj na Youtube</a></p>
    </div>
</template>

<script>
    export default {
        name: 'Spotify',
        props: {
            spotify: Object
        },
        computed: {
            youtubeLink: function () {
                return this.spotify ?
                    'https://www.youtube.com/results?search_query=' + encodeURI(this.spotify.item.album.artists[0].name + ' - ' + this.spotify.item.name)
                    : '';
            }
        }
    }
</script>

<style>
    .hello {
        padding: 10px;
        text-align: center;
    }

    a:link, a:visited {
        color: #1DB954;
    }

    a.youtube:link, a.youtube:visited {
        color: #c4302b;
    }

</style>
