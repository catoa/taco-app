<template>
  <b-container>
    <b-row>
      <b-col cols="6" class="header">
        <b-button @click="getNewTaco()" class="new-taco-button">New Taco</b-button>
        <h4>Today's Taco</h4>
      </b-col>
    </b-row>
    <div id="taco-recipes">
      <b-row class="recipe-row">
        <b-col cols="10" class="recipe-style-one">
          <div class="recipe-text" v-html="tacoSummary"></div>
        </b-col>
        <b-col cols="2" class="aside-style-one"></b-col>
      </b-row>
      <b-row class="recipe-row" v-for="(part, index, id) in taco" :key="id">
        <b-col
          cols="10"
          :class="{ 'recipe-style-one' : (id + 1) % 2 === 0, 'recipe-style-two' : (id + 1) % 2 !== 0 }"
        >
          <div class="recipe-text" v-html="part.recipe"></div>
        </b-col>
        <b-col
          cols="2"
          :class="{ 'aside-style-one' : (id + 1) % 2 === 0, 'aside-style-two' : (id + 1) % 2 !== 0 }"
        >
          <div class="recipe-name-column">
            <span>{{ part.name }}</span>
          </div>
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
        return `${this.taco.baseLayer.cleanName} <strong>with</strong> ${
          this.taco.mixin.cleanName
        } <strong>garnished with</strong> ${
          this.taco.condiment.cleanName
        } <strong>topped off with</strong> ${
          this.taco.seasoning.cleanName
        } <strong>and wrapped in delicious</strong> ${
          this.taco.shell.cleanName
        }`;
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
    cleanRecipe(obj) {
      obj.recipe = markdown.toHTML(obj.recipe);
      let cleanName = obj.name.replace(/ *\([^)]*\) */g, "");
      return {
        ...obj,
        cleanName
      };
    },
    createTaco(data) {
      let baseLayer = this.cleanRecipe(data["base_layer"]);
      let seasoning = this.cleanRecipe(data["seasoning"]);
      let mixin = this.cleanRecipe(data["mixin"]);
      let shell = this.cleanRecipe(data["shell"]);
      let condiment = this.cleanRecipe(data["condiment"]);

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
#taco-recipes {
  color: white;
}
.header {
  height: 200px;
}
.recipe-text {
  padding: 20px;
}
.recipe-name-column {
  transform: translateX(-50%) translateY(-50%) rotate(90deg);
  white-space: nowrap;
  font-weight: bold;
  position: absolute;
  top: 50%;
  left: 50%;
}
.recipe-row {
  min-height: 150px;
}
.new-taco-button {
  margin-top: 1em;
  margin-left: 0.5em;
  margin-bottom: 110px;
  background-color: #ff6347;
  border-radius: 10px;
  font-weight: bold;
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
