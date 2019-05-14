<template>
  <b-container>
    <b-row>
      <b-col cols="6" class="header">
        <b-button @click="getNewTaco()" class="new-taco-button">New Taco</b-button>
        <h4>Today's Taco</h4>
      </b-col>
    </b-row>
    <div>
    <b-row>
      <b-col cols="10" class="recipe-style-one">
        <div class="taco-recipe">{{ tacoSummary }}</div>
      </b-col>
      <b-col cols="2" class="aside-style-one"></b-col>
    </b-row>
    <b-row v-for="(part, index, id) in taco" :key="id" class="taco-recipe">
      <b-col
        cols="10"
        :class="{ 'recipe-style-one' : (id + 1) % 2 === 0, 'recipe-style-two' : (id + 1) % 2 !== 0 }"
      >
        <div v-html="part.md"></div>
        <!-- <div>{{ part.recipe }}</div> -->
        <!-- <p v-if="part.tags">{{ part.tags }}</p> -->
      </b-col>
      <b-col
        cols="2"
        :class="{ 'aside-style-one' : (id + 1) % 2 === 0, 'aside-style-two' : (id + 1) % 2 !== 0 }"
      >
        <div class="text-rotate rotate-90">{{ part.name }}</div>
      </b-col>
    </b-row>
    </div>
  </b-container>
</template>

<script>
import { markdown } from "markdown";
export default {
  name: "TacoRecipe",
  data() {
    return {
      taco: null,
      errors: []
    };
  },
  created() {
    if (this.taco == null) {
      this.getNewTaco();
    }
  },
  computed: {
    tacoSummary: function() {
      if (this.taco) {
        return `${this.taco.baseLayer.cleanName} with ${
          this.taco.mixin.cleanName
        } garnished with ${this.taco.condiment.cleanName} topped off with ${
          this.taco.seasoning.cleanName
        } and wrapped in delicious ${this.taco.shell.cleanName}`;
      } else {
        return "";
      }
    }
  },
  methods: {
    getNewTaco() {
      this.$http
        .get("https://taco-randomizer.herokuapp.com/random/")
        .then(response => {
          this.taco = this.createTaco(response.data);
        })
        .catch(err => {
          // eslint-disable-next-line
          console.warn(err);
        });
    },
    getTags(value) {
      let tagsIndex = value.indexOf("tags");

      return tagsIndex > -1
        ? {
            index: tagsIndex,
            tags: value.substring(tagsIndex, value.length).trim()
          }
        : { index: tagsIndex, tags: "" };
    },
    cleanRecipe(obj) {
      let { index, tags } = this.getTags(obj.recipe);
      let recipeString = obj.recipe;
      let md = markdown.toHTML(obj.recipe);
      let cleanString = recipeString.substring(
        recipeString.lastIndexOf("=") + 1,
        index
      );
      obj.recipe = cleanString;
      let cleanName = obj.name.replace(/ *\([^)]*\) */g, "");
      return {
        ...obj,
        tags,
        cleanName,
        md
      };
    },
    createTaco(data) {
      let baseLayer = this.cleanRecipe(data["base_layer"]);
      let seasoning = this.cleanRecipe(data["seasoning"]);
      let mixin = this.cleanRecipe(data["mixin"]);
      let shell = this.cleanRecipe(data["shell"]);
      let condiment = this.cleanRecipe(data["condiment"]);

      // remove string delimited with multiple equal signs
      // create new lines from either * (asterisks) or numbers (1., 2., etc.)
      return {
        baseLayer,
        seasoning,
        mixin,
        shell,
        condiment
      };
    }
  }
};
</script>
<style scoped>
.header {
  height: 150px;
}
.new-taco-button {
  margin: 1em;
  background-color: #ff6347;
  border-radius: 5px;
  font-weight: bold;
}
.taco-recipe {
  color: white;
}
.rotate-90 {
  transform: rotate(90deg);
}
.text-rotate {
  /* TODO: add padding to the content */
  /* TODO: vertically center both the recipe and name of the ingredient */
  padding-left: 20px;
  font-weight: bold;
  white-space: pre;
}
.recipe-style-one {
  background: #ff6347;
}
.recipe-style-two {
  background: #f73513;
}
.aside-style-one {
  background: #f73513;
}
.aside-style-two {
  background: #ca3c23;
}
</style>
