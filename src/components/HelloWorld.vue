<template>
  <div>
    <div>
      <!-- <h1>{{ msg }}</h1> -->
      <h1>Случайный выбор публикаций</h1>
    </div>
    <div>
      <input placeholder="введите хэштег" v-model="tag" />
    </div>
    <div>
      <button type="button" v-on:click.prevent="selectRandomPost" v-if="!image">Выбрать публикацию</button>

      <button type="button" v-on:click.prevent="resetForm" v-if="!!image">Начать заново</button>
    </div>

    <div v-if="!!image && !!tag">
      <a v-bind:href="detailsUrl">
        <img v-bind:src="image" height="256px" width="256px" />
      </a>
      <!-- <p v-text="userName"></p> -->
    </div>

    <pre>{{tag}}</pre>
  </div>
</template>

<script>
let ig = require("instagram-scraping");
var storage = {};

export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data: function() {
    console.log(ig);

    return {
      tag: "",

      image: "",
      userName: "",
      detailsUrl: ""
    };
  },
  methods: {
    resetForm() {
      if(this.tag){
        this.selectRandomPost();
      }else{
        this.$set(this, "image", null);
        this.$set(this, "tag", null);
      }

    },
    selectRandomPost() {
      // alert(this.tag);
      if (!this.tag) return;

      if (storage.hasOwnProperty(this.tag)) {
        startSelection.call(this, storage[this.tag]);
        return;
      }

      ig.scrapeTag(this.tag).then(posts => {
        var reduced = posts.medias.filter(p=> p.owner_id != "4365503338");
        posts = {
          total:reduced.length,
          medias: reduced
        };
        storage[this.tag] = posts;
        startSelection.call(this, posts);
      });
    }
  }
};

function startSelection(posts) {
  var total = posts.total;
  var randomIndex = Math.ceil(Math.random() * (total - 1));
  var timer = setInterval(showRandomPost.bind(this, posts), 200);
  setTimeout(stopTimer.bind(this, timer), 5000);
}

function showRandomPost(posts) {
  this.$nextTick(() => {
    var total = posts.total;
    var randomIndex = Math.ceil(Math.random() * (total - 1));
    var post = posts.medias[randomIndex];

    var thumbnail = post.thumbnail;
    var image = post.display_url;
    var user = post.owner_id;

    this.$set(this, "image", thumbnail);
    this.$set(this, "userName", user);
    this.$set(this, "detailsUrl", "https://instagram.com/p/" + post.shortcode);
  });
}

function stopTimer(timer) {
  clearInterval(timer);

  /*var thumbnail = post.thumbnail;
  var image = post.display_url;
  var user = post.owner_id;

  this.$set(this, "image", image);
  this.$set(this, "userName", user);*/
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="stylus">
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
