<template>
  <div class="container">
    <div v-if="recipe">
      <div class="recipe-header mt-3 mb-4">
        <h1>{{ recipe.title }}</h1>
        <img :src="recipe.image" class="center" />
        <br>
        <br>
        <br>

        <span v-if="$root.store.username">
          <span v-if="!this.favorite">
            <b-button @click="addToFavorite" variant="outline-dark">
              <b-icon-star></b-icon-star>
            </b-button>
          </span>
          <span v-else>
            <b-button
              @click="addToFavorite"
              variant="outline-success"
              disabled="false"
            >
              <b-icon-star></b-icon-star>
            </b-button>
          </span>
        </span>
      </div>
      <div class="recipe-body">
        <div class="wrapper">
          <div class="wrapped">
            <div class="mb-3">
              <div>Ready in {{ recipe.readyInMinutes }} minutes</div>
              <div>Likes: {{ recipe.aggregateLikes }} likes</div>
              <div>Servings: {{ recipe.servings }}</div>
              <div v-if="recipe.vegan">
                <b-avatar
                  variant="tranparent"
                  :src="require('@/assets/vegan.png')"
                  style="width: 2.5em; height: 3.3em"
                >
                </b-avatar>
              </div>
              <div v-if="recipe.vegetarian">
                <b-avatar
                  variant="tranparent"
                  :src="require('@/assets/vegetarian.png')"
                  style="width: 4.5em; height: 3em"
                >
                </b-avatar>
              </div>
              <div v-if="recipe.gluteFree">
                <b-avatar
                  variant="tranparent"
                  :src="require('@/assets/gluten_free.png')"
                  style="width: 3.8em; height: 3em"
                >
                </b-avatar>
              </div>
            </div>
            Ingredients:
            <ul>
              <li
                v-for="(r, index) in recipe.ingredients"
                :key="index + '_' + r.id"
              >
                {{ r.original }}
              </li>
            </ul>
          </div>
          <div class="wrapped">
            Instructions:
            <ol>
              <li v-for="s in recipe._instructions" :key="s.number">
                {{ s.step }}
              </li>
            </ol>
          </div>
        </div>
      </div>
      <!-- <pre>
      {{ $route.params }}
      {{ recipe }}
    </pre
      > -->
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      recipe: null,
      favorite: false,
    };
  },
  async created() {
    try {
      let response;
      // response = this.$route.params.response;
      console.log(this.$route.params.title);

      try {
        if (this.$route.params.title == "My Recipes") {
          response = await this.axios.get(
            // "https://test-for-3-2.herokuapp.com/recipes/info",
            this.$root.store.server_domain +
              "/recipes/myFullDetailes?recipeid=" +
              this.$route.params.recipeId,
            { withCredentials: true }
          );
        } else {
          response = await this.axios.get(
            // "https://test-for-3-2.herokuapp.com/recipes/info",
            this.$root.store.server_domain +
              "/recipes/fullDetailes?recipeid=" +
              this.$route.params.recipeId,
            { withCredentials: true }
          );
        }

        // console.log("response.status", response.status);
        if (response.status !== 200) this.$router.replace("/NotFound");
      } catch (error) {
        console.log("error.response.status", error.response.status);
        this.$router.replace("/NotFound");
        return;
      }

      let _recipe;

      if (this.$route.params.title == "My Recipes") {
        console.log(response.data);

        let {
          title,
          image,
          readyInMinutes,
          aggregateLikes,
          vegan,
          vegetarian,
          glutenFree,
          ingredients,
          instructions,
          servings,          
        } = response.data;

        let _instructions = instructions;

        _recipe = {
          title,
          image,
          readyInMinutes,
          aggregateLikes,
          vegan,
          vegetarian,
          glutenFree,
          ingredients,
          _instructions,
          servings,
        };

      } else {
        console.log(response.data);

        let {
          analyze_Instructions,
          instructions,
          ingredients,
          aggregateLikes,
          readyInMinutes,
          image,
          title,
          servings,
          vegan,
          vegetarian,
          glutenFree,
        } = response.data;

        console.log(analyze_Instructions);

        let _instructions = analyze_Instructions
          .map((fstep) => {
            fstep.steps[0].step = fstep.name + fstep.steps[0].step;
            return fstep.steps;
          })
          .reduce((a, b) => [...a, ...b], []);

        console.log(_instructions);

        _recipe = {
          instructions,
          _instructions,
          analyze_Instructions,
          ingredients,
          aggregateLikes,
          readyInMinutes,
          image,
          title,
          servings,
          vegan,
          vegetarian,
          glutenFree,
        };
      }

      this.recipe = _recipe;
      this.checkIfFavorite();
    } catch (error) {
      console.log(error);
    }
  },
  methods: {
    async addToFavorite() {
      const recipe = { recipeId: this.$route.params.recipeId };
      let response;

      try {
        response = await this.axios.post(
          this.$root.store.server_domain + "/user/favorites",
          {
            rec_id: recipe.recipeId,
            withCredentials: true,
          }
        );
        this.favorite = true;
      } catch (err) {
        console.log(err.response);
        this.form.submitError = err.response.data.message;
      }
    },

    async checkIfFavorite() {
      const recipe = { recipeId: this.$route.params.recipeId };
      let response;

      try {
        response = await this.axios.get(
          this.$root.store.server_domain +
            "/user/isAFavorites?recipeId=" +
            recipe.recipeId,
          { withCredentials: true }
        );

        if (response != undefined) {
          if (response.data == true) this.favorite = true;
        }

        console.log(reaponse);
      } catch (err) {
        console.log(err.response);
        // this.form.submitError = err.response.data.message;
      }
    },
  },
};
</script>

<style scoped>
.wrapper {
  display: flex;
}
.wrapped {
  width: 50%;
}
.center {
  display: block;
  margin-left: auto;
  margin-right: auto;
  width: 50%;
}
/* .recipe-header{

} */
</style>
