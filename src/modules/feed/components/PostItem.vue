<template>
  <v-card elevation="2" @click="openPost" class="mt-2">
    <v-card-title>
      {{ post.title }}
    </v-card-title>
    <v-card-subtitle
      >Posted {{ createdFormatted }} ago by {{ post.author.username }} in
      {{ post.community.name }}</v-card-subtitle
    >
    <v-card-actions>
      <v-btn icon @click.stop="upvote(post)" :color="getUpvoteIconColor(post)">
        <v-progress-circular
          :width="3"
          indeterminate
          color="primary"
          v-if="upvoting === post.id"
        />
        <v-icon v-else>mdi-arrow-up</v-icon>
      </v-btn>
      <v-btn icon @click.stop="downvote(post)">
        <v-icon>mdi-arrow-down</v-icon>
      </v-btn>
      <v-btn text @click.stop="gotoPost" color="orange">Comments</v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
import Post from "../models/post";
import FormatDistance from "date-fns/formatDistance";
import { mapActions, mapGetters } from "vuex";

export default {
  name: "PostItem",
  props: {
    post: {
      type: Post,
      required: true,
    },
  },

  data: () => ({
    upvoting: null,
  }),

  methods: {
    ...mapActions("$feed", ["upvotePost", "downvotePost"]),

    openPost() {
      if (this.post.href) {
        window.open(this.post.href).focus();

        return false;
      }

      this.gotoPost();
    },

    gotoPost() {
      this.$router.push({
        name: "Post",
        params: {
          id: this.post.id,
        },
      });
    },

    upvote(post) {
      if (this.loggedIn && post && !this.upvoting) {
        this.upvoting = post.id;

        if (!post.your_vote) {
          this.upvotePost(post).finally(() => (this.upvoting = null));
        } else {
          this.downvotePost(post).finally(() => (this.upvoting = null));
        }
      }
    },

    downvote(post) {
      if (this.loggedIn) {
        this.downvotePost(post);
      }
    },

    /** @param {Post} post */
    getUpvoteIconColor(post) {
      return post.your_vote ? "orange" : "";
    },
  },

  computed: {
    ...mapGetters("$auth", ["loggedIn"]),

    /** @returns {string} */
    createdFormatted() {
      return FormatDistance(this.post.created, new Date());
    },
  },
};
</script>

<style scoped></style>
