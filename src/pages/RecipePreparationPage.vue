<template>
    <div class="container mt-5">
      <div v-if="recipe">
        <div class="text-center mb-4">
          <h1 class="display-4">{{ recipe.title }}</h1>
          <img :src="recipe.image|| defaultImage" alt="Recipe Image" class="img-fluid rounded recipe-image" />
        </div>
        <div v-if="recipe.instructions && recipe.instructions.length > 0" class="text-center mt-3">
            <h5>Progress label</h5>
            <b-progress :value="progress" :max="max_len" :precision="2"></b-progress>
            <span>{{ progress }}/{{max_len}}</span>
        </div>
        <div class="row">
          <div class="col-md-6 mb-4">
            <h2 class="h4">Ingredients</h2>
            <ul class="list-group">
              <li class="list-group-item" v-for="ingredient in multipliedIngredients" :key="ingredient.id">
                {{ ingredient.amount }} {{ ingredient.unit }} {{ ingredient.name }}
              </li>
            </ul>
            <b-button variant="primary" class="mt-3" @click="multiplyIngredients">Double Ingredients</b-button>
          </div>
          <div class="col-md-6 mb-4" v-if="recipe.instructions && recipe.instructions.length">
            <h2 class="h4">Preparation Steps</h2>
            <ul class="list-group">
              <li class="list-group-item" v-for="(step, index) in recipe.instructions" :key="index">
                <b-form-checkbox v-model="stepCompleted[index]" @change="saveProgress">
                  {{ step }}
                </b-form-checkbox>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import familyrecipes from "../assets/mocks/familyrecipes.json";
  import axios from "axios";
  export default {
    name: "recipeprep",
    data() {
      return {
        recipe: null,
        multipliedIngredients: [],
        stepCompleted: [],
        defaultImage: require('@/assets/myrecipes.png'),
        progress: 0,
        max_len: 1
      };
    },
    methods: {
      getImageUrl(image) {
        try {
          return require(`@/assets/${image}`);
        } catch (e) {
          console.error(`Image not found: ${image}`);
          return '';
        }
      },
      async fetchRecipe() {
        const recipeId = this.$route.params.id;
        const collection = this.$route.query.collection;
        let data;
        let localRecipes = [];
        let localRecipe = [];

        // family recipe
        if (collection === 'family') {
          this.axios.defaults.withCredentials = true;
          try{
            const response = await this.axios.get(`https://recipes-heaven.cs.bgu.ac.il/users/familyrecipes`);
            console.log(response.data);
            data = response.data || [];
            data = data.find(r => r.id === recipeId);

          }catch(error){
            console.log("error")
          }
          console.log("Prepa: "+ data.preparation);
          
          this.recipe = {
          id: data.id,
          title: data.title,
          image: this.getImageUrl(data.image[0]),
          ingredients: data.ingredients,
          instructions: [data.preparation],
        };
        this.multipliedIngredients = this.recipe.ingredients;

        // myrecipes recipe
        } else if (collection === 'myrecipes') {
          this.axios.defaults.withCredentials = true;
          try{
            const response = await this.axios.get(`https://recipes-heaven.cs.bgu.ac.il/users/myrecipes`);
            console.log(response.data);
            this.recipes = response.data || [];
          }catch(error){
            console.log("error")
          }
          this.recipes.forEach(recipe => {
            if (!recipe.image) {
              recipe.image = this.defaultImage;
            }
          });

          console.log("ID: " + recipeId);
          data = this.recipes.find(r => r.id === recipeId);
          let ingredients = [];

      // Process each ingredient in the array
        console.log("Recipe: " + data);
        data.ingredients.forEach(ingredient => {
          ingredients.push({
            name: ingredient.name,
            amount: ingredient.amount,
            unit: ingredient.unit,
          });

        });
          
          this.recipe = {
          id: data.id,
          title: data.title,
          ingredients: ingredients,
          instructions: data.analyzedInstructions && data.analyzedInstructions[0]
            ? data.analyzedInstructions[0].steps.map(step => step.step)
            : [] // Default to an empty array if instructions are undefined
        };
          this.multipliedIngredients = [...this.recipe.ingredients];
          if (this.recipe.instructions) {
            this.max_len = this.recipe.instructions.length;
          }
          
        }

        //from api
        else{
          let response;
          try {
            response = await axios.get(`https://recipes-heaven.cs.bgu.ac.il/recipes/${recipeId}`);
          } catch (error) {
            console.error("Error fetching recipe from API:", error.response ? error.response.status : error.message);
            this.$router.replace("/NotFound");
            return;
          }
          data = response.data;

          this.recipe = {
            id: data.id,
            title: data.title,
            image: data.image,
            ingredients: data.extendedIngredients.map(ingredient => ({
              id: ingredient.id,
              name: ingredient.name,
              amount: ingredient.measures.metric.amount,
              unit: ingredient.measures.metric.unitShort
            })),
            instructions: data.analyzedInstructions[0]?.steps.map(step => step.step) || []
          };
          this.multipliedIngredients = [...this.recipe.ingredients];
          //this.loadProgress();
          if (this.recipe.instructions) {
            this.max_len = this.recipe.instructions.length;
          }
        }
        this.loadProgress(); 
      },
      multiplyIngredients() {
        this.multipliedIngredients = this.multipliedIngredients.map(ingredient => ({
          ...ingredient,
          amount: ingredient.amount * 2
        }));

        this.max_len = this.recipe.instructions.length;
      },
      /**
       * Save the progress of the recipe to the local storage
       */
      async saveProgress() {
        // localStorage.setItem(`recipeProgress-${this.recipe.id}`, JSON.stringify(this.stepCompleted));
        try{
          this.axios.defaults.withCredentials = true;
          await this.axios.post(`https://recipes-heaven.cs.bgu.ac.il/users/${this.recipe.id}/progress`, {progress: JSON.stringify(this.stepCompleted)} );
          console.log('Recipe progress saved successfully.');
        } catch (error) {
          onsole.error('Error saving recipe progress:', error);
        }
        this.updateProgress();
      },
      /**
       * Load the progress of the recipe from the local storage
       */
      async loadProgress() {
        // const savedProgress = localStorage.getItem(`recipeProgress-${this.recipe.id}`);
        try{
          this.axios.defaults.withCredentials = true;
          const response = await this.axios.get(`https://recipes-heaven.cs.bgu.ac.il/users/${this.recipe.id}/progress`);
          const savedProgress = response.data[0].progress || null;
          console.log(response.data[0].progress)
          if (savedProgress) {
            this.stepCompleted = savedProgress;
          } else {
            this.stepCompleted = Array(this.recipe.instructions.length).fill(false);
          }
        } catch (error) {
        console.error('Error fetching recipe progress:', error);
        this.stepCompleted = Array(this.recipe.instructions.length).fill(false);
        }
          this.updateProgress();        
      },
      
      updateProgress() {
        const completedSteps = this.stepCompleted.filter(step => step).length;
        if (this.max_len != 0) {
          this.progress = (completedSteps / this.max_len) * this.max_len;
        }
      },
    },
    mounted() {
      this.fetchRecipe();
      this.updateProgress();
    }
  };
  </script>
  
  <style scoped>
  .recipe-image {
    width: 400px; /* Set a specific width */
    height: 300px; /* Set a specific height */
    object-fit: cover; /* Ensures the image covers the specified dimensions while maintaining aspect ratio */
  }
 
  </style>
  