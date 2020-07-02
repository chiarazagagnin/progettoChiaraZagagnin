<template>
  <div>
    <!-- info playlist -->
    <div class="title">
      <div class="margin">
      <h1 class="md-title">Playlist: {{playlist}}</h1>
      <span class="md-subheader">{{description}}</span>
      <br>
    </div>
    </div>

    <div>
      <md-card class="playlist">
        <md-card-media-cover md-solid>
          <md-card-media md-ratio="1:1">
            <img :src="img">
          </md-card-media>

          <md-card-area>
            <md-card-actions>
              <div>
                <md-button class="md-icon-button" @click="likePlaylist(playlistId)">
                  <md-icon v-if="like">favorite</md-icon>
                  <md-icon v-else>favorite_outline</md-icon>
                </md-button>
              </div>
            </md-card-actions>
          </md-card-area>
        </md-card-media-cover>
      </md-card>
      <br>
    </div>
    <br>
    <!-- tracce -->
    <md-table>
      <md-table-row>
        <md-table-head style="width:20px"></md-table-head>
        <md-table-head>Tracks in "{{playlist}}"</md-table-head>
        <md-table-head class="md-small-hide">Duration</md-table-head>
        <md-table-head class="md-medium-hide">Artist</md-table-head>
      </md-table-row>

      <md-table-row v-for="(song, i) in songs" :key="i" @click="songInfo(song.track.id)">
        <md-table-cell style="width:20px">
          <md-button
            class="md-icon-button md-list-button"
            @click="likeSong(song.track.id, i)"
            onclick="event.stopPropagation()"
          >
            <md-icon v-if="songids.includes(i)">favorite</md-icon>
            <md-icon v-else>favorite_outline</md-icon>
          </md-button>
        </md-table-cell>
        <md-table-cell>{{song.track.name}}</md-table-cell>
        <md-table-cell class="md-small-hide">{{durationMinutes(song.track.duration_ms)}}</md-table-cell>
        <md-table-cell class="md-medium-hide">{{song.track.artists[0].name}}</md-table-cell>
        <md-table-cell class="md-xsmall-hide">
          <md-button
            v-if="song.track.preview_url != null"
            onclick="event.stopPropagation()"
            class="md-icon-button md-list-button"
            @click="playAudio(song.track.preview_url)"
          >
            <md-icon v-if="url == song.track.preview_url">pause</md-icon>
            <md-icon v-else>play_arrow</md-icon>
          </md-button>
        </md-table-cell>
        <md-table-cell class="md-small-hide">
          <md-button
            class="md-icon-button"
            :to="'/artist/' + song.track.artists[0].id"
            onclick="event.stopPropagation()"
          >
            <md-icon>person_search</md-icon>
          </md-button>
        </md-table-cell>
      </md-table-row>
    </md-table>
  </div>
</template>

<script>
import DataService from "../dataservice";

export default {
  data: function() {
    return {
      playlistId: this.$route.params.id,
      playlist: "",
      description: "",
      img: "",
      songs: [],
      ids: [], //id delle canzoni piaciute
      show: true,
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
      DataService.getPlaylistInfo(this.playlistId, newToken).then(data => {
        this.songs = data.data.tracks.items;

        //canzoni piaciute
        this.songs.forEach(element => {
          this.ids.push(element.track.id);
        });
        DataService.getLikes("track").then(data => {
          data.forEach(element => {
            if (this.ids.indexOf(element) >= 0) {
              this.songids.push(this.ids.indexOf(element));
            }
          });
        });
        this.playlist = data.data.name;
        this.description = data.data.description;
        this.img = data.data.images[0].url;
      });
    });
    DataService.getLikes("playlist").then(data => {
      if (data.indexOf(this.playlistId) >= 0) {
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
    likePlaylist(id) {
      if (this.like === false) {
        DataService.setLike("playlist", id);
      } else {
        DataService.getLikes("playlist").then(data => {
          this.dblikes = data;
          DataService.getDocs("playlist").then(data => {
            this.dbdocs = data;
            let index = this.dblikes.indexOf(id);
            let doc = this.dbdocs[index];
            DataService.removeLike("playlist", doc);
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
.playlist {
  max-width: 400px;
}
</style>
