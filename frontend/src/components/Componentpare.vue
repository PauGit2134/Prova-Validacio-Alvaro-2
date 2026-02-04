<template>
    <div>
    <div class="d-flex justify-center my-4">
      <v-btn @click="buscar('A')">A</v-btn>
      <v-btn @click="buscar('B')">B</v-btn>
      <v-btn @click="buscar('C')">C</v-btn>
    </div>

    <v-row>
      <v-col 
        v-for="item in llista" 
        :key="item.idDrink" 
        cols="12" md="4"
      >
        <Componentfill :beguda="item" />
      </v-col>
    </v-row>
  </div>
</template>

<script setup>
import {ref} from 'vue';
import Componentfill from './Componentfill.vue';

const llista = ref([]);

const buscar = (lletra) => {
  fetch(`https://www.thecocktaildb.com/api/json/v1/1/search.php?f=${lletra}`)
    .then(res => res.json())
    .then(data => {
      llista.value = data.drinks; 
    })
    .catch(e => console.error(e));
};
</script>