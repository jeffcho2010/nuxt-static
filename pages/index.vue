<template>
  <section>
    <div>
      <Title/>
      <div class="links">
        <nuxt-link to="/about">關於</nuxt-link>
        {{posts}}
      </div>
    </div>
  </section>
</template>

<script>
import Title from "~/components/Title.vue";
import request from "axios";
const getPosts = function() {
  console.log("Request to posts");
  return new Promise((resolve, reject) => {
    request.defaults.baseURL = "https://nuxt.craftedup.com/wp-json/wp/v2";
    request.get(`posts`).then(response => {
      const data = [...response.data];
      if (response.status === 200 && response.data.length > 0) {
        const filtered = {
          total: response.headers["x-wp-total"],
          totalPages: response.headers["x-wp-totalpages"],
          data: data.map(item => ({
            id: item.id,
            title: item.title.rendered,
            content: item.content.rendered,
            excerpt: item.excerpt.rendered,
            slug: item.slug
          }))
        };
        resolve(filtered);
      } else {
        reject(response);
      }
    });
  });
};
export default {
  components: {
    Title
  },
  async asyncData({ params }) {
    // We can use async/await ES6 feature
    let { data } = await getPosts();
    return {
      posts: data
    };
  },
  methods: {
    getPost(slug) {
      return new Promise((resolve, reject) => {
        request.defaults.baseURL = "https://nuxt.craftedup.com/wp-json/wp/v2";
        request.get(`posts?slug=${slug}`).then(response => {
          const data = [...response.data][0];
          if (response.status === 200 && response.data.length > 0) {
            const filtered = {
              content: data.content.rendered,
              author: data.author,
              date: data.date,
              date_gmt: data.date_gmt,
              excerpt: data.excerpt.rendered,
              featured_media: data.featured_media,
              guid: data.guid.rendered,
              link: data.link,
              slug: data.slug,
              title: data.title.rendered
            };
            resolve(filtered);
          } else {
            reject(response);
          }
        });
      });
    }

    // moreArticles() {
    //   // this.indexInfiniteLoading.page++
    //   this.$axios
    //     .get("https://blockstudio.tw/wp-json/wp/v2/posts")
    //     .then(response => {
    //       console.log(response.data);
    //       // this.$store.commit("setArticles", response.data);
    //       // this.$refs.infiniteLoading.$emit("$InfiniteLoading:loaded");
    //     })
    //     .catch(() => {
    //       // this.$refs.infiniteLoading.$emit("$InfiniteLoading:complete");
    //     });
    // }
  }
};
</script>

<style>
.links {
  padding-top: 15px;
}
</style>
