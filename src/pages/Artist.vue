<template>
  <div>
    <!-- informazioni artista -->
    <div class="title">
      <div class="margin">
    <h1 class="md-title">Artist: {{name}}</h1>
    </div>
    </div>
    <div>
      <md-card class="artist">
        <md-card-media md-big>
          <img :src="img">
        </md-card-media>
        <md-card-actions>
          <md-button class="md-icon-button" @click="likeArtist(id)">
            <md-icon v-if="like">favorite</md-icon>
            <md-icon v-else>favorite_outline</md-icon>
          </md-button>
        </md-card-actions>
      </md-card>
    </div>
    <br>
    <!-- generi musicali, tag -->
    <div>
      <md-chip class="md-accent" v-for="genre in genres" :key="genre">{{genre}}</md-chip>
    </div>

    <br>
    <md-table>
      <md-table-row>
        <md-table-head>
          <b>Followers</b>
        </md-table-head>
        <md-table-head>
          <b>Popularity</b>
        </md-table-head>
      </md-table-row>
      <md-table-row>
        <md-table-cell>{{followers}}</md-table-cell>
        <md-table-cell>
          <div style="float:left">
            {{popularity}}/100
            <span style="color:white">__</span>
          </div>

          <div style="width:70%" class="md-small-hide">
            <md-progress-bar
              style="height:15px"
              class="md-accent"
              md-mode="determinate"
              :md-value="popularity"
            ></md-progress-bar>
          </div>
        </md-table-cell>
      </md-table-row>
    </md-table>
    <md-divider></md-divider>
    <br>
    <!-- canzoni più famose -->
    <md-list class="md-double-line">
      <span class="md-text">
        <b>Top tracks</b>
      </span>
      <br>
      <md-divider></md-divider>
      <div v-for="(track,i) in tracks" :key="i">
        <md-list-item @click="songInfo(track.id)">
          <md-button
            class="md-icon-button md-list-button"
            @click="likeSong(track.id, i)"
            onclick="event.stopPropagation()"
          >
            <md-icon v-if="idLikedSongs.includes(i)">favorite</md-icon>
            <md-icon v-else>favorite_outline</md-icon>
          </md-button>
          <div class="md-list-item-text">
            <span>{{track.name}}</span>
            <span>{{durationMinutes(track.duration_ms) }} minutes</span>
          </div>
          <md-button
            v-if="track.preview_url != null"
            onclick="event.stopPropagation()"
            class="md-icon-button md-list-button"
            @click="playAudio(track.preview_url)"
          >
            <md-icon v-if="url == track.preview_url">pause</md-icon>
            <md-icon v-else>play_arrow</md-icon>
          </md-button>
        </md-list-item>
        <md-divider></md-divider>
      </div>
      <md-divider></md-divider>
    </md-list>

    <br>
    <!-- album più famosi -->
    <span class="md-text">
      <b>Top albums</b>
    </span>
    <br>
    <br>
    <div class="md-layout md-gutter">
      <div
        class="md-layout-item md-size-25 md-medium-size-33 md-small-size-50 md-xsmall-size-100"
        v-for="(album, i) in albums"
        :key="i"
        @click="albumInfo(album.id)"
      >
        <md-card md-with-hover>
          <md-card-header>
            <div class="md-title card-text">{{album.name}}</div>
          </md-card-header>
          <md-card-media md-big>
            <img :src="album.images[0].url">
          </md-card-media>
          <md-card-actions>
            <md-button
              class="md-icon-button"
              @click="likeAlbum(album.id, i)"
              onclick="event.stopPropagation()"
            >
              <md-icon v-if="idLikedAlbums.includes(i)">favorite</md-icon>
              <md-icon v-else>favorite_outline</md-icon>
            </md-button>
          </md-card-actions>
        </md-card>
        <br>
      </div>
    </div>
    <md-divider></md-divider>

    <br>
    <br>
    <!-- artisti correlati -->
    <span class="md-text">
      <b>Related Artists</b>
    </span>
    <br>
    <br>
    <div class="md-layout md-gutter">
      <div
        class="md-layout-item md-size-25 md-medium-size-33 md-small-size-50 md-xsmall-size-100"
        v-for="(relate, i) in related"
        :key="i"
        @click="artistInfo(relate.id)"
      >
        <md-card md-with-hover>
          <md-card-media-cover md-solid>
            <md-card-media md-ratio="4:3">
              <img :src="relate.images[1].url">
            </md-card-media>

            <md-card-area>
              <md-card-header>
                <span class="md-title card-text">{{relate.name}}</span>
              </md-card-header>
              <md-card-actions>
                <md-button
                  class="md-icon-button"
                  @click="likeArtists(relate.id, i)"
                  onclick="event.stopPropagation()"
                >
                  <md-icon v-if="idLikedArtists.includes(i)">favorite</md-icon>
                  <md-icon v-else>favorite_outline</md-icon>
                </md-button>
              </md-card-actions>
            </md-card-area>
          </md-card-media-cover>
        </md-card>
        <br>
      </div>
    </div>
  </div>
</template>

<script>
import DataService from "../dataservice";
export default {
  data: function() {
    return {
      id: "",
      name: "",
      img: "",
      genres: [],
      albums: [],
      tracks: [],
      related: [],
      like: false,
      dblikes: [],
      dbdocs: [],
      play: false,
      audio: null,
      url: "",
      idAllSongs: [], //id delle canzoni piaciute
      idLikedSongs: [], //id delle card delle canzoni piaciute
      idAllAlbums: [], //id degli album piaciuti
      idLikedAlbums: [], //id delle card degli album piaciuti
      idAllArtists: [], //id degli artisti correlati piaciuti
      idLikedArtists: [], //id delle card degli artisti correlati piaciuti
      followers: 0,
      popularity: 0
    };
  },
  watch: {
    $route: function() {
      this.load();
    }
  },
  created: function() {
    this.load();
  },
  methods: {
    load() {
      this.idAllSongs = [];
      this.idLikedSongs = [];
      this.idAllAlbums = [];
      this.idLikedAlbums = [];
      this.idAllArtists = [];
      this.idLikedArtists = [];
      this.id = this.$route.params.id;
      // informazioni dell'artista
      DataService.getToken().then(response => {
        let newToken = response.data.access_token;
        DataService.getArtistInfo(this.id, newToken).then(data => {
          this.genres = data.data.genres;
          this.name = data.data.name;
          this.img = data.data.images[1].url;
          this.followers = data.data.followers.total;
          this.popularity = data.data.popularity;
        });

        //tracce più famose (informazioni + like o meno dal database)
        DataService.getTopTracks(this.id, newToken).then(el => {
          this.tracks = el.data.tracks;
          this.tracks.forEach(element => {
            this.idAllSongs.push(element.id);
          });
          DataService.getLikes("track").then(data => {
            data.forEach(element => {
              if (this.idAllSongs.indexOf(element) >= 0) {
                this.idLikedSongs.push(this.idAllSongs.indexOf(element));
              }
            });
          });
        });

        //album più famosi (informazioni + like o meno dal database)
        DataService.getArtistAlbums(this.id, newToken).then(al => {
          this.albums = al.data.items;
          this.albums.forEach(element => {
            this.idAllAlbums.push(element.id);
          });
          DataService.getLikes("album").then(data => {
            data.forEach(element => {
              if (this.idAllAlbums.indexOf(element) >= 0) {
                this.idLikedAlbums.push(this.idAllAlbums.indexOf(element));
              }
            });
          });
        });

        //artisti correlati (informazioni + like o meno dal database)
        DataService.getRelatedArtists(this.id, newToken).then(data => {
          this.related = data.data.artists;
          this.related.forEach(element => {
            this.idAllArtists.push(element.id);
          });
          DataService.getLikes("artist").then(data => {
            data.forEach(element => {
              if (this.idAllArtists.indexOf(element) >= 0) {
                this.idLikedArtists.push(this.idAllArtists.indexOf(element));
              }
            });
          });
        });
      });

      //like per l'artista selezionato
      DataService.getLikes("artist").then(data => {
        if (data.indexOf(this.id) >= 0) {
          this.like = true;
        } else {
          this.like = false;
        }
      });
    },
    albumInfo(id) {
      this.$router.push({ path: "/album/" + id });
    },
    songInfo(id) {
      this.$router.push({ path: "/track/" + id });
    },
    artistInfo(id) {
      this.$router.push({ path: "/artist/" + id });
    },
    likeArtist(id) {
      //like all'artista selezionato
      if (this.like === false) {
        DataService.setLike("artist", id);
      } else {
        DataService.getLikes("artist").then(data => {
          this.dblikes = data;
          DataService.getDocs("artist").then(data => {
            this.dbdocs = data;
            let index = this.dblikes.indexOf(id);
            let doc = this.dbdocs[index];
            DataService.removeLike("artist", doc);
          });
        });
      }
      this.like = !this.like;
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
      if (this.idLikedSongs.includes(i) === false) {
        DataService.setLike("track", id);
        this.idLikedSongs.push(i);
      } else {
        let ids = this.idLikedSongs.indexOf(i);
        this.idLikedSongs.splice(ids, 1);
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
    },
    likeAlbum(id, i) {
      if (this.idLikedAlbums.includes(i) === false) {
        DataService.setLike("album", id);
        this.idLikedAlbums.push(i);
      } else {
        let ids = this.idLikedAlbums.indexOf(i);
        this.idLikedAlbums.splice(ids, 1);
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
    },
    likeArtists(id, i) {
      if (this.idLikedArtists.includes(i) === false) {
        DataService.setLike("artist", id);
        this.idLikedArtists.push(i);
      } else {
        let ids = this.idLikedArtists.indexOf(i);
        this.idLikedArtists.splice(ids, 1);
        DataService.getLikes("artist").then(data => {
          this.dblikes = data;
          DataService.getDocs("artist").then(data => {
            this.dbdocs = data;
            let index = this.dblikes.indexOf(id);
            let doc = this.dbdocs[index];
            DataService.removeLike("artist", doc);
          });
        });
      }
    }
  }
};
</script>

<style>
.card-text {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.artist {
  max-width: 400px;
}
</style>