<template>
  <div>
    <h2>your Friends</h2>

    <!-- Search Bar -->
    <input
      type="text"
      v-model="searchQuery"
      @input="filterPosts"
      placeholder="Search by title..."
    />

    <table class="table">
      <thead>
        <tr>
          <th>Name</th>
          <th>Action</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="post in filteredPosts" :key="post.id">
          <td>{{ post.title }}</td>
          <td>
            <button v-on:click="deletefriend(post.id)">Delete Friend</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axiosInstance from "../services/AxiosTokenInstance";
import { mapGetters, mapMutations } from "vuex";
import {
  GET_USER_TOKEN_GETTER,
  LOADING_SPINNER_SHOW_MUTATION,
} from "../store/storeconstants";

export default {
  data() {
    return {
      posts: [],
      searchQuery: "",
    };
  },
  computed: {
    ...mapGetters("auth", {
      userName: GET_USER_TOKEN_GETTER,
    }),
    filteredPosts() {
      if (!this.searchQuery) {
        // If searchQuery is empty, return all posts
        return this.posts;
      } else {
        // Filter the posts based on searchQuery
        const query = this.searchQuery.toLowerCase();
        return this.posts.filter((post) =>
          post.title.toLowerCase().includes(query)
        );
      }
    },
  },
  mounted() {
    this.showLoading(true);
    axiosInstance
      .get(`https://vue-completecourse.firebaseio.com/posts.json`)
      .then((response) => {
        this.formatPosts(response.data);
        this.showLoading(false);
      })
      .catch(() => {
        this.showLoading(false);
      });
  },
  methods: {
    ...mapMutations({
      showLoading: LOADING_SPINNER_SHOW_MUTATION,
    }),
    formatPosts(posts) {
      this.posts = Object.keys(posts).map((key) => ({
        ...posts[key],
        id: key,
      }));
      console.log(this.posts);
    },
    deletefriend(id) {
      axiosInstance
        .delete(`https://vue-completecourse.firebaseio.com/posts/${id}.json`)
        .then(() => {
          this.posts = this.posts.filter((post) => post.id !== id);
        })
        .catch((error) => {
          console.error(error);
        });
    },
    filterPosts() {
      // The computed property "filteredPosts" will automatically handle filtering
      // based on the updated value of "searchQuery"
    },
  },
};
</script>

<style scoped></style>
