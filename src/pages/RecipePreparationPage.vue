<template>
    <div class="container mt-5">
      <div v-if="recipe">
        <div class="text-center mb-4">
          <h1 class="display-4">{{ recipe.title }}</h1>
          <img :src="recipe.image" alt="Recipe Image" class="img-fluid rounded" />
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
          <div class="col-md-6 mb-4">
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
  export default {
    name: "recipeprep",
    data() {
      return {
        recipe: null,
        multipliedIngredients: [],
        stepCompleted: [],
      };
    },
    methods: {
      async fetchRecipe() {
        const recipeId = this.$route.params.id;
        const localRecipes = JSON.parse(localStorage.getItem("userRecipes") || "[]");
        let data;
        
        const localRecipe = localRecipes.find(r => r.id === recipeId);
        if (localRecipe) {
          data = localRecipe;
        } else {
          const response = await fetch(`https://api.spoonacular.com/recipes/${recipeId}/information?apiKey=b1a72f1616ff413e984ea8dc1377d964`);
          data = await response.json();
        }
        
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
        this.loadProgress();
      },
      multiplyIngredients() {
        this.multipliedIngredients = this.multipliedIngredients.map(ingredient => ({
          ...ingredient,
          amount: ingredient.amount * 2
        }));
      },
      saveProgress() {
        localStorage.setItem(`recipeProgress-${this.recipe.id}`, JSON.stringify(this.stepCompleted));
      },
      loadProgress() {
        const savedProgress = localStorage.getItem(`recipeProgress-${this.recipe.id}`);
        if (savedProgress) {
          this.stepCompleted = JSON.parse(savedProgress);
        } else {
          this.stepCompleted = Array(this.recipe.instructions.length).fill(false);
        }
      },
      logout() {
        // Implement the logout logic and clear the progress
        localStorage.removeItem(`recipeProgress-${this.recipe.id}`);
        this.stepCompleted = Array(this.recipe.instructions.length).fill(false);
      }
    },
    mounted() {
      this.fetchRecipe();
    }
  };
  </script>
  
  <style scoped>
  .recipe-image {
    max-width: 100%;
    height: auto;
  }
  </style>
  