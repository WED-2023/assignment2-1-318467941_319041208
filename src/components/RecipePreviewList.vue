<template>
  <b-container>
    <h3>
      {{ title }}
      <slot></slot>
    </h3>
    <b-row v-if="recipes && recipes.length !== 0">
      <b-col v-for="r in recipes" :key="r.id">
        <RecipePreview class="recipePreview" :recipe="r" />
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import RecipePreview from "./RecipePreview.vue";
import { mockGetRecipesPreview } from "../services/recipes.js";
import axios from 'axios';
export default {
  name: "RecipePreviewList",
  components: {
    RecipePreview
  },
  props: {
    title: {
      type: String,
      required: true
    },
    listType: {
      type: String,
      default: 'random' // Can be 'lastViewed' or 'random'
    }
  },
  data() {
    return {
      recipes: []
    };
  },
  mounted() {
    this.updateRecipes();
  },
  methods: {
    async updateRecipes() {
      this.recipes = [];
      if (this.listType === 'lastviewed'){
        // local
        // let viewedRecipes = JSON.parse(localStorage.getItem('lastviewed')) || [];
        // console.log('Viewed recipes:', viewedRecipes);

        
        // // Ensure we have at least 3 viewed recipes
        // if (viewedRecipes.length >= 3) {
        //   const recipePromises = [
        //     this.getrecipe(viewedRecipes[viewedRecipes.length - 1]),
        //     this.getrecipe(viewedRecipes[viewedRecipes.length - 2]),
        //     this.getrecipe(viewedRecipes[viewedRecipes.length - 3])
        //   ];

        //   // Wait for all recipe promises to resolve
        //   const recipes = await Promise.all(recipePromises);
        //   console.log('Fetched recipes:', recipes);
        //   this.recipes.push(...recipes);
        // }
        // else if (viewedRecipes.length >= 2) {
        //   const recipePromises = [
        //     this.getrecipe(viewedRecipes[viewedRecipes.length - 1]),
        //     this.getrecipe(viewedRecipes[viewedRecipes.length - 2]),
        //   ];

        //   // Wait for all recipe promises to resolve
        //   const recipes = await Promise.all(recipePromises);
        //   console.log('Fetched recipes:', recipes);
        //   this.recipes.push(...recipes);
        // }
        // else if (viewedRecipes.length >= 1) {
        //   const recipePromises = [
        //     this.getrecipe(viewedRecipes[viewedRecipes.length - 1]),
        //   ];

        //   // Wait for all recipe promises to resolve
        //   const recipes = await Promise.all(recipePromises);
        //   console.log('Fetched recipes:', recipes);
        //   this.recipes.push(...recipes);
        // }

        // remote
        try {
        this.axios.defaults.withCredentials = true;
        const response = await this.axios.get('https://recipes-heaven.cs.bgu.ac.il/users/lastviewed');
        const viewedRecipes = response.data || [];
        // Ensure we have at least 3 viewed recipe
        console.log("Testtttt " + JSON.stringify(viewedRecipes))
        if (viewedRecipes.length >= 3) {
          const recipePromises = viewedRecipes
            .slice(0,3) // Get the last three viewed recipes
            .map(recipe  => this.getrecipe(recipe.recipeId)); // Fetch each recipe
            console.log("Viewed")
          // Wait for all recipe promises to resolve
          const recipes = await Promise.all(recipePromises);
          console.log('Fetched recipes:', recipes);
          this.recipes.push(...recipes);
        } }catch (error) {
        console.error('Error fetching viewed recipes:', error);
        }
      }
      else{
        try {
        // const response = await this.axios.get(
        //   this.$root.store.server_domain + "/recipes/random",
        // );
       
        let usedIndices = new Set();
        let recipePromises = [];

        console.log("length", this.recipes.length);
        let successfulRecipes = 0;

        while (successfulRecipes < 3) {
          const randomIndex = Math.floor(100000 + Math.random() * 900000);
          console.log("index", randomIndex);
          if (!usedIndices.has(randomIndex)) {
          let temp = await this.getrecipe(randomIndex);
            console.log(temp);
            if (temp !== null) {
              recipePromises.push(Promise.resolve(temp));
              successfulRecipes++;
              usedIndices.add(randomIndex);
            }
          }
        }
        const recipes = await Promise.all(recipePromises);
        console.log('Fetched recipes:', recipes);
        this.recipes.push(...recipes);
        usedIndices.add(randomIndex);
        console.log("length", this.recipes.length);

      } catch (error) {
        console.log(error);
      }
      }
    },

  async getrecipe(id) {
    
    const recipeId = id;
    try {
      const response = await axios.get(`https://recipes-heaven.cs.bgu.ac.il/recipes/${recipeId}`);
      console.log('Recipe API response:', response);

      const {
        analyzedInstructions,
        extendedIngredients,
        aggregateLikes,
        readyInMinutes,
        image,
        title,
        vegan,
        vegetarian,
        glutenFree,
      } = response.data;
      const _instructions = analyzedInstructions
        .map(instruction => instruction.steps)
        .reduce((allSteps, steps) => [...allSteps, ...steps], []);

      return {
        _instructions,
        extendedIngredients,
        aggregateLikes,
        readyInMinutes,
        image,
        title,
        vegan,
        vegetarian,
        glutenFree,
        id
      };
    } catch (error) {
      console.log('Error fetching recipe from API:', error.response ? error.response.status : error.message);
      return null; // Return null to handle the error gracefully
    }
  }
}
}
</script>

<!-- <style lang="scss" scoped>
.container {
  min-height: 400px;
}
</style> -->
<style scoped>
.recipe-preview-list {
  margin: 20px 0;
}
.recipe-preview {
  margin-bottom: 20px;
  text-align: center;
}
.recipe-image {
  max-width: 100%;
  height: auto;
  display: block;
  margin: 0 auto 10px;
}
.recipe-title {
  font-size: 1.2em;
  margin: 10px 0;
}
.recipe-details {
  color: #666;
}
h3 {
  color: #42b983; /* Pastel green */
  align-self: center;
  text-align: center; /* Center text horizontally */
  margin: 0; /* Remove default margin */
  
  /* Flexbox adjustments */
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%; /* Ensures it takes up full height of its container */
}

</style>

