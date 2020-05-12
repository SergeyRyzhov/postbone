<template>
  <v-container class="fill-height" fluid>
    <v-row align="center" justify="center">
      <v-col cols="12">
        <v-text-field
          color="success"
          :loading="!tag"
          hint="введите хэштег"
          persistent-hint
          v-model="tag"
        ></v-text-field>
        <v-btn type="button" v-on:click.prevent="selectRandomPost" v-if="!image">Выбрать публикацию</v-btn>
      </v-col>
    </v-row>
    <v-row align="center" justify="center" v-if="!!image && !!tag">
      <v-col cols="12" sm="10" md="8">
        <a v-bind:href="detailsUrl">
          <img v-bind:src="image" height="100%" width="100%" />
        </a>
        <!-- <p v-text="userName"></p> -->
      </v-col>
    </v-row>
    <v-row align="center" justify="center">
      <v-col cols="12">
        <v-btn type="button" v-on:click.prevent="resetForm" v-if="!!image">Начать заново</v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
const ig = require("instagram-scraping");
const confetti = require("canvas-confetti");

var storage = {};

function SchoolPride(canvas, end) {
  var colors = ["#bb0000", "#ffffff"];
  var myConfetti = confetti.create(canvas, {
    resize: true,
    useWorker: true
  });
  myConfetti({
    particleCount: 2,
    angle: 60,
    spread: 55,
    origin: { x: 0 },
    //colors: colors,
    disableForReducedMotion :true
  });
  myConfetti({
    particleCount: 2,
    angle: 120,
    spread: 55,
    origin: { x: 1 },
    //colors: colors,
    disableForReducedMotion :true
  });

  if (Date.now() < end) {
    requestAnimationFrame(SchoolPride.bind(this, canvas, end));
  }
}

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
      if (this.tag) {
        this.selectRandomPost();
      } else {
        this.$set(this, "image", null);
        this.$set(this, "tag", null);
      }
    },
    selectRandomPost() {
      // alert(this.tag);
      if (!this.tag) {
        // var end = Date.now() + 1 * 1000;
        // SchoolPride(this.$refs.canvas, end);
        return;
      }

      if (storage.hasOwnProperty(this.tag)) {
        startSelection.call(this, storage[this.tag]);
        return;
      }

      ig.scrapeTag(this.tag).then(posts => {
        var reduced = posts.medias.filter(p => p.owner_id != "4365503338");
        posts = {
          total: reduced.length,
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

  var end = Date.now() + 1 * 1000;
  SchoolPride(this.$refs.canvas, end);

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
