<template>
  <div>
    <div class="title">
      <div class="margin">
        <h1 class="md-title"  v-if="posts != 0">Community</h1>
        <br>
      </div>
    </div>
    <div v-if="posts == 0">
      <md-empty-state md-rounded md-icon="post_add" md-label="No posts found">
        <br>
      </md-empty-state>
    </div>

    <div class="md-layout md-gutter">
      <div
        class="md-layout-item md-size-33 md-medium-size-33 md-small-size-50 md-xsmall-size-100"
        v-for="post in posts"
        :key="post.id"
      >
        <md-card>
          <md-card-header>
            <md-toolbar class="md-transparent" md-elevation="0">
              <div class="md-toolbar-row">
                <div class="md-toolbar-section-start">
                  <div class="md-title card-text">{{post.artist}}</div>
                </div>
                <div class="md-toolbar-section-end">
                  <md-button
                    class="md-icon-button"
                    :to="'/artist/' + post.id"
                    onclick="event.stopPropagation()"
                  >
                    <md-icon>person_search</md-icon>
                  </md-button>
                </div>
              </div>
            </md-toolbar>
            <br>
            <div>
              <md-chip class="md-accent" v-for="tag in post.tags" :key="tag">{{tag}}</md-chip>
            </div>
            <br>
            <div class="md-subhead">
              <b>{{post.user[0]}} {{post.user[1]}}</b>
            </div>
            <div class="md-subhead">{{post.text}}</div>
            <br>
          </md-card-header>
          <md-card-media>
            <div class="post">
              <img style="height:150px" :src="post.image">
            </div>
          </md-card-media>
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
      posts: []
    };
  },
  created: function() {
    DataService.getAllPosts().then(data => {
      this.posts = data
    });
  },
  methods: {}
};
</script>

<style>
.dialog {
  margin: 3%;
}
.inputImg button {
  color: grey;
}
img {
  width: 200px;
  height: 100px;
}

</style>
