<template>
  <div class="md-layout md-gutter">
    <div class="title">
      <div class="margin">
        <h1 class="md-title">Category: {{categoryId}}</h1>
        <br>
      </div>
    </div>
    <br>
    <div
      class="md-layout-item md-size-25 md-medium-size-33 md-small-size-50 md-xsmall-size-100"
      v-for="(playlist, i) in playlists"
      :key="i"
      @click="playlistInfo(playlist.id)"
    >
      <md-card md-with-hover>
        <md-card-area>
          <md-card-media>
            <img :src="playlist.images[0].url">
          </md-card-media>

          <md-card-header>
            <div class="md-title card-text">{{playlist.name}}</div>
            <div class="md-subhead card-text">{{playlist.description}}</div>
          </md-card-header>
          <md-card-actions>
            <md-button
              class="md-icon-button"
              @click="likePlaylist(playlist.id, i)"
              onclick="event.stopPropagation()"
            >
              <md-icon v-if="idLikedPlaylists.includes(i)">favorite</md-icon>
              <md-icon v-else>favorite_outline</md-icon>
            </md-button>
          </md-card-actions>
        </md-card-area>
      </md-card>
      <br>
    </div>
  </div>
</template>

<script>
import DataService from "../dataservice";
export default {
  data: function() {
    return {
      categoryId: this.$route.params.id,
      playlists: [],
      idAllPlaylists: [], //id delle playlist piaciute
      idLikedPlaylists: [] //id delle card delle playlist piaciute
    };
  },
  created: function() {
    DataService.getToken().then(response => {
      let newToken = response.data.access_token;
      DataService.getCategoryPlaylists(this.categoryId, newToken).then(data => {
        this.playlists = data.data.playlists.items;

        // playlist piaciute
        this.playlists.forEach(element => {
          this.idAllPlaylists.push(element.id);
        });
        DataService.getLikes("playlist").then(data => {
          data.forEach(element => {
            if (this.idAllPlaylists.indexOf(element) >= 0) {
              this.idLikedPlaylists.push(this.idAllPlaylists.indexOf(element));
            }
          });
        });
      });
    });
  },
  methods: {
    playlistInfo(id) {
      this.$router.push({ path: "/playlist/" + id });
    },
    likePlaylist(id, i) {
      if (this.idLikedPlaylists.includes(i) === false) {
        DataService.setLike("playlist", id);
        this.idLikedPlaylists.push(i);
      } else {
        let ids = this.idLikedPlaylists.indexOf(i);
        this.idLikedPlaylists.splice(ids, 1);
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
</style>
