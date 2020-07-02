<template>
  <div class="md-layout md-gutter">
    <div class="title">
      <div class="margin">
        <h1 class="md-title">New Releases</h1>
        <br>
      </div>
    </div>
    <br>
    <div
      class="md-layout-item md-size-25 md-medium-size-33 md-small-size-50 md-xsmall-size-100"
      v-for="(album, i) in albums"
      :key="i"
      @click="albumInfo(album.id)"
    >
      <md-card md-with-hover>
        <md-card-header>
          <div class="md-title card-text">{{album.name}}</div>
          <div class="md-subhead card-text">{{album.artists[0].name}}</div>
        </md-card-header>
        <md-card-media md-big>
          <img :src="album.images[0].url">
        </md-card-media>

        <md-card-actions>
          <md-button
            class="md-icon-button"
            :to="'/artist/' + album.artists[0].id"
            onclick="event.stopPropagation()"
          >
            <md-icon>person_search</md-icon>
          </md-button>
          <md-button
            class="md-icon-button"
            @click="likeAlbum(album.id, i)"
            onclick="event.stopPropagation()"
          >
            <md-icon v-if="albumids.includes(i)">favorite</md-icon>
            <md-icon v-else>favorite_outline</md-icon>
          </md-button>
        </md-card-actions>
      </md-card>
      <br>
    </div>
    <div class="margin">
      <md-button class="md-accent md-raised" @click="loadMore()">Load more</md-button>
      <md-progress-spinner v-if="loading" :md-diameter="30" :md-stroke="3" md-mode="indeterminate"></md-progress-spinner>
    </div>
  </div>
</template>

<script>
import DataService from "../dataservice";

export default {
  data: function() {
    return {
      user: [localStorage.getItem("name"), localStorage.getItem("surname")],
      albums: [],
      ids: [], //id degli album piaciuti
      offset: 0,
      loading: false,
      albumids: [], //id delle card degli album piaciuti
      dblikes: [],
      dbdocs: []
    };
  },
  created: function() {
    this.loading = true;
    DataService.getToken().then(response => {
      let newToken = response.data.access_token;
      DataService.getNewReleases(this.offset, newToken).then(data => {
        this.albums = data.data.albums.items;

        //check like agli album
        this.albums.forEach(element => {
          this.ids.push(element.id);
        });
        DataService.getLikes("album").then(data => {
          data.forEach(element => {
            if (this.ids.indexOf(element) >= 0) {
              this.albumids.push(this.ids.indexOf(element));
            }
          });
          this.loading = false;
        });
      });
    });
  },
  methods: {
    albumInfo(id) {
      this.$router.push({ path: "/album/" + id });
    },
    loadMore() {
      this.loading = true;
      this.offset += 20;
      DataService.getToken().then(response => {
        let newToken = response.data.access_token;
        DataService.getNewReleases(this.offset, newToken).then(data => {
          data.data.albums.items.forEach(element => {
            this.ids.push(element.id);
          });
          DataService.getLikes("album").then(data => {
            data.forEach(element => {
              if (this.ids.indexOf(element) >= 0) {
                this.albumids.push(this.ids.indexOf(element));
              }
            });
          });
          this.albums = this.albums.concat(data.data.albums.items);
          this.loading = false;
        });
      });
    },
    likeAlbum(id, i) {
      if (this.albumids.includes(i) === false) {
        DataService.setLike("album", id);
        this.albumids.push(i);
      } else {
        let ids = this.albumids.indexOf(i);
        this.albumids.splice(ids, 1);
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