<template>
    <div class="hello">
        <p>
            Tomek słucha{{ spotify && !spotify.is_playing ? 'ł ostatnio' : ' teraz'}}:
        </p>
        <h1>
            <h2>{{ spotify.item.artists.length > 1 ? spotify.item.artists.map((item) => { return item.name }).join(', ')
            : spotify.item.artists[0].name }}</h2>
            <h4>{{ spotify.item.name }}</h4>
        </h1>
        <a v-if="spotify" :href="spotify ? spotify.item.external_urls.spotify : ''">Posłuchaj razem z nim</a>
    </div>
</template>

<script>
    export default {
        name: 'Spotify',
        props: {
            spotify: Object
        }
    }
</script>

<style>
    .hello {
        padding: 15px;
        text-align: center;
    }

    a:link, a:visited {
        color: #1DB954;
    }

</style>
