<template>
  <div>
    <div class="title">
      <div class="margin">
        <h1 class="md-title" v-if="posts != 0">Your Concerts</h1>
        <br>
      </div>
    </div>
    <!-- dialog nuovo post -->
    <md-dialog :md-active.sync="active" class="dialog" style="z-index: 6">
      <md-dialog-title>New post</md-dialog-title>
      <md-dialog-content>
        <md-field>
          <label>Add text</label>
          <md-input v-model="text"></md-input>
        </md-field>
        <md-chips class="md-accent" v-model="tags" md-placeholder="Add tags"></md-chips>
        <div style="width: 100%">
          <md-autocomplete
            v-model="selectedItem"
            :md-options="searchOptions"
            @md-changed="searchItem"
            @md-selected="select"
            :md-open-on-focus="false"
          >
            <label>Add artist</label>
          </md-autocomplete>
        </div>
        <md-field>
          <label>Add image url</label>
          <md-input v-model="image"></md-input>
        </md-field>
      </md-dialog-content>
      <md-dialog-actions>
        <md-button class="md-primary" @click="active = false">Close</md-button>
        <md-button
          class="md-primary"
          :disabled="!text || !tags || !image || !artist"
          @click="active = false, addPost()"
        >Save</md-button>
      </md-dialog-actions>
    </md-dialog>

    <!-- empty state -->
    <div v-if="posts == 0">
      <md-empty-state md-rounded md-icon="post_add" md-label="No posts found">
        <br>
        <md-button class="md-raised md-primary" @click="active = true">Add post</md-button>
      </md-empty-state>
    </div>

    <!-- post esistenti -->
    <div v-else class="md-layout md-gutter">
      <div
        class="md-layout-item md-size-25 md-medium-size-33 md-small-size-50 md-xsmall-size-100"
        v-for="post in posts"
        :key="post.id"
      >
        <!--nei methods devi farne uno che cambia a seconda del type -->
        <md-card>
          <md-card-header>
            <div class="md-title card-text">{{post.artist}}</div>
            <br>
            <div>
              <md-chip class="md-accent" v-for="tag in post.tags" :key="tag">{{tag}}</md-chip>
            </div>
            <br>
            <div class="md-subhead card-text">{{post.text}}</div>
          </md-card-header>
          <md-card-media>
            <img style="height:150px" :src="post.image">
          </md-card-media>
          <md-card-actions>
            <md-button
              class="md-icon-button"
              :to="'/artist/' + post.id"
              onclick="event.stopPropagation()"
            >
              <md-icon>person_search</md-icon>
            </md-button>
            <md-button class="md-icon-button" @click="dialog(post)">
              <md-icon>delete</md-icon>
            </md-button>
          </md-card-actions>
        </md-card>

        <!-- dialog eliminazione post -->
        <md-dialog :md-active.sync="active2" class="dialog">
          <md-dialog-title>Are you sure you want to delete this post?</md-dialog-title>
          <md-dialog-actions>
            <md-button class="md-primary" @click="active2 = false">Cancel</md-button>
            <md-button class="md-primary" @click="active2 = false, deletePost(selected)">Delete</md-button>
          </md-dialog-actions>
        </md-dialog>
        <br>
      </div>

      <md-button class="md-fab md-fab-bottom-right md-primary" @click="active=true">
        <md-icon>add</md-icon>
      </md-button>
      <div>
        <md-progress-spinner
          style="margin:3%"
          v-if="loading"
          :md-diameter="30"
          :md-stroke="3"
          md-mode="indeterminate"
        ></md-progress-spinner>
      </div>
    </div>
  </div>
</template>

<script>
import DataService from "../dataservice";
export default {
  data: function() {
    return {
      text: "",
      active: false,
      tags: [],
      artist: "",
      image: "",
      posts: [],
      active2: false,
      selected: null,
      selectedItem: null,
      searchOptions: [],
      searchIds: [],
      artistId: "",
      loading: false
    };
  },
  created: function() {
    this.load();
  },
  methods: {
    load() {
      this.text = "";
      this.image = "";
      this.tags = [];
      this.artist = "";
      this.artistId = "";
      this.selectedItem = null;
      DataService.getPosts().then(data => {
        this.posts = data;
      });
    },
    addPost() {
      DataService.setPost(
        this.text,
        this.image,
        this.tags,
        this.artist,
        this.artistId
      );

      this.load();
    },
    dialog(el) {
      this.selected = el;
      this.active2 = true;
    },
    deletePost(docum) {
      this.loading = true;
      DataService.getPosts().then(data => {
        DataService.getDocs("post").then(data => {
          this.dbdocs = data;
          let index = this.posts.indexOf(docum);
          let doc = this.dbdocs[index];
          DataService.removePost(doc);
          setTimeout(() => {
            this.load();
            this.loading = false;
          }, 700);
        });
      });
    },
    searchItem: function(term) {
      DataService.getToken().then(response => {
        let newToken = response.data.access_token;
        DataService.search(term, newToken, "artist").then(data => {
          this.searchOptions = data.map(element => element.name);
          this.searchIds = data.map(element => element.id);
        });
      });
    },
    select: function(selected) {
      let indexId = this.searchOptions.indexOf(selected);
      this.artistId = this.searchIds[indexId];
      this.artist = selected;
      this.selectedItem = selected;
      this.searchOptions = [];
    }
  }
};
</script>

<style lang="scss" scoped>
.md-dialog /deep/.md-dialog-container {
  width: 768px;
  margin: 4%;
}
.cover {
  object-fit: cover;
  width: 250px;
  height: 100px;
}
.md-autocomplete + strong {
  margin-top: 36px;
  display: block;
}
</style>
