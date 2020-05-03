<template>
    <div class="hello">
        <h5>
            Tomek słucha{{ !spotify.is_playing ? 'ł ostatnio' : ' teraz'}}:
        </h5>
        <br>
        <h2>
            <a :href="spotify.item.external_urls.spotify">
                <h5>{{ spotify.item.artists[0].name }}</h5>
                <h4>{{ spotify.item.name }}</h4>
            </a>
        </h2>
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
