<template>

  <form class="selector" @submit="addIngredient">
    <input ref="searchInput" type="text" placeholder="Rechercher une huile" @input="updateOptions">
    <select ref="select">
      <option v-for="oil in selectOptions">
        {{ oil.label }}
      </option>
    </select>
    <input type="button" value="Ajouter" @click="addIngredient">
  </form>
  <table>
    <tr>
      <th>Ingredient</th>
      <th>Poid (g)</th>
      <th>Quantité actuelle</th>
      <th>Quantité max.</th>
      <th>Température max</th>
      <th>Conservation</th>
      <!-- 
      -->
    </tr>
    <tr v-for="(ingredient,i) in selectedIngredients">
      <th>
        {{ ingredient.label }}
      </th>
      <td>
        <input class="weight" type="number" placeholder="g" v-model="ingredient.value">g
      </td>
      <td>{{ (ingredient.value / totalWeight * 100).toFixed(2) || 0 }}%</td>
      <td :class="ingredient.max < ingredient.value / totalWeight ? 'danger' : ''">{{ ingredient.max * 100 }}%</td>
      <td>{{ ingredient.melt ? ingredient.melt + "°" : "-"}}</td>
      <td>{{ ingredient.conservation }}</td>
      <!-- 
      -->
      <td><input type="button" value="Retirer" @click="removeIngredient(i)"></td>
    </tr>
    <tr>
      <th>Eau</th>
      <td><input class="weight" type="number" v-model="waterValue">g</td>
      <td>{{ (waterValue / totalWeight * 100).toFixed(2) }}%</td>
      <td>100%</td>
    </tr>
    <tr>
      <th>NaH</th>
      <td>{{ totalSap.toFixed(3) }} g</td>
      <td><input type="range" v-model="superFat" min="0" max="15"> ({{superFat}}% surgras)</td>
    </tr>
    <tr>
      <th>Total</th>
      <td>{{ totalWeight.toFixed(2) }}g</td>
    </tr>
    <tr>
      <th>Savons (100g)</th>
      <table>{{ (totalWeight / 100).toFixed(2) }}</table>
    </tr>
  </table>
  <h2>Recette</h2>
  <table>
    <tr v-for="ingredient in selectedIngredients">
      <td>{{ ingredient.label }}</td>
      <td>{{ ingredient.value }}g</td>  
    </tr>
    <tr>
      <th>Eau</th>
      <td>{{waterValue}}g</td>
    </tr>
    <tr>
      <th>NaH</th>
      <td>{{ totalSap.toFixed(3) }} g</td>
    </tr>
    <tr>
      <th>Total</th>
      <td>{{ totalWeight.toFixed(2) }}g</td>
    </tr>
  </table>
</template>
<script setup>
import {ref, computed} from "vue"
import oils from "./oils.json"
const searchInput = ref(null)
const selectedIngredients = ref([])
const select = ref(null)
const selectOptions = ref(oils)
const waterValue = ref(0)
const superFat = ref(0)
const totalWeight = computed(()=> {
  let total = waterValue.value
  selectedIngredients.value.map(ingredient => total += ingredient.value)
  return total + totalSap.value
})
const totalSap = computed(()=> {
  let total = 0
  selectedIngredients.value.map(ingredient => total += ingredient.sap * ingredient.value)
  return total * (1 - superFat.value / 100)
})
const updateOptions = ()=>{
  selectOptions.value = oils.filter(oil => oil.label.toLowerCase().match(searchInput.value.value.toLowerCase()))
}
const addIngredient = (e)=>{
  e.preventDefault();
  
  const ingredient = select.value.selectedOptions[0].value || select.value.options[0].value
  if(!ingredient) return;
  const oil = oils.find(oil => oil.label == ingredient)
  selectedIngredients.value.push({...oil, value: 0})
  searchInput.value.value = ""
  updateOptions()
}
const removeIngredient = (index)=>{
  selectedIngredients.value.splice(index, 1)
}
</script>
<style scoped>
  .selector {
    display: flex;
    flex-direction: column;
  }
  input.weight {
    text-align: right;
    background-color: transparent;
    border: none;
    border-bottom: 1px solid grey;
    width: 5rem;
  }
  .danger {
    color: red;
  }
</style>
