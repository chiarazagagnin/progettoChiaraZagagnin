<template>
  <div>
    <!-- informazioni dell'album -->
    <div class="title">
      <div class="margin">
      <h1 class="md-title">Album: {{album}} - {{artist.name}}</h1>
    </div>
    </div>
    <br>
    <div>
      <md-card class="album">
        <md-card-media md-big>
          <img :src="img">
        </md-card-media>
        <md-card-actions>
          <md-button
            class="md-icon-button"
            :to="'/artist/' + artist.id"
            onclick="event.stopPropagation()"
          >
            <md-icon>person_search</md-icon>
          </md-button>
          <md-button class="md-icon-button" @click="likeAlbum(albumId)">
            <md-icon v-if="like">favorite</md-icon>
            <md-icon v-else>favorite_outline</md-icon>
          </md-button>
        </md-card-actions>
      </md-card>
      <br>
    </div>
    <br>
    <!-- tracce -->
    <md-list class="md-double-line">
      <span class="md-text">
        <b>Tracks in "{{album}}"</b>
      </span>
      <br>
      <md-divider></md-divider>
      <div v-for="(song, i) in songs" :key="i">
        <md-list-item @click="songInfo(song.id)">
          <md-button
            class="md-icon-button md-list-button"
            @click="likeSong(song.id, i)"
            onclick="event.stopPropagation()"
          >
            <md-icon v-if="songids.includes(i)">favorite</md-icon>
            <md-icon v-else>favorite_outline</md-icon>
          </md-button>
          <div class="md-list-item-text">
            <span>{{song.name}}</span>
            <span>{{durationMinutes(song.duration_ms) }} minutes</span>
          </div>
          <md-button
            v-if="song.preview_url != null"
            onclick="event.stopPropagation()"
            class="md-icon-button md-list-button"
            @click="playAudio(song.preview_url)"
          >
            <md-icon v-if="url == song.preview_url">pause</md-icon>
            <md-icon v-else>play_arrow</md-icon>
          </md-button>
        </md-list-item>
        <md-divider></md-divider>
      </div>
    </md-list>
  </div>
</template>

<script>
import DataService from "../dataservice";

export default {
  data: function() {
    return {
      albumId: this.$route.params.id,
      album: "",
      artist: "",
      img: "",
      songs: [],
      ids: [], //id delle canzoni piaciute
      like: false,
      dblikes: [],
      dbdocs: [],
      play: false,
      audio: null,
      url: "",
      songids: [] //id delle card delle canzoni piaciute
    };
  },
  created: function() {
    DataService.getToken().then(response => {
      let newToken = response.data.access_token;
      DataService.getAlbumInfo(this.albumId, newToken).then(data => {
        this.songs = data.data.tracks.items;

        //canzoni piaciute
        this.songs.forEach(element => {
          this.ids.push(element.id);
        });
        DataService.getLikes("track").then(data => {
          data.forEach(element => {
            if (this.ids.indexOf(element) >= 0) {
              this.songids.push(this.ids.indexOf(element));
            }
          });
        });
        this.album = data.data.name;
        this.artist = data.data.artists[0];
        this.img = data.data.images[0].url;
      });
    });

    // check like album selezionato
    DataService.getLikes("album", this.user).then(data => {
      if (data.indexOf(this.albumId) >= 0) {
        this.like = true;
      } else {
        this.like = false;
      }
    });
  },
  methods: {
    songInfo(id) {
      this.$router.push({ path: "/track/" + id });
    },
    durationMinutes(duration) {
      let value = "";
      if (parseInt((duration / 1000) % 60, 10) <= 9) {
        value =
          parseInt((duration / (1000 * 60)) % 60, 10) +
          ":0" +
          parseInt((duration / 1000) % 60, 10);
      } else {
        value =
          parseInt((duration / (1000 * 60)) % 60, 10) +
          ":" +
          parseInt((duration / 1000) % 60, 10);
      }
      return value;
    },
    likeAlbum(id) {
      if (this.like === false) {
        DataService.setLike("album", id);
      } else {
        DataService.getLikes("album").then(data => {
          this.dblikes = data;
          DataService.getDocs("album").then(data => {
            this.dbdocs = data;
            let index = this.dblikes.indexOf(id);
            let doc = this.dbdocs[index];
            DataService.removeLike("album", doc);
          });
        });
      }
      this.like = !this.like;
    },
    playAudio(url) {
      if (this.play === false) {
        this.url = url;
        this.audio = new Audio(url);
        this.audio.play();
        this.play = true;
      } else {
        this.url = "";
        this.audio.pause();
        this.play = false;
      }
    },
    likeSong(id, i) {
      if (this.songids.includes(i) === false) {
        DataService.setLike("track", id);
        this.songids.push(i);
      } else {
        let ids = this.songids.indexOf(i);
        this.songids.splice(ids, 1);
        DataService.getLikes("track").then(data => {
          this.dblikes = data;
          DataService.getDocs("track").then(data => {
            this.dbdocs = data;
            let index = this.dblikes.indexOf(id);
            let doc = this.dbdocs[index];
            DataService.removeLike("track", doc);
          });
        });
      }
    }
  }
};
</script>

<style>
.album {
  max-width: 400px;
}
</style>
