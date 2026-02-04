<template>
  <v-card class="mx-auto" elevation="4">
    <v-img 
      :src="beguda.strDrinkThumb" 
      height="200px" 
      cover
    ></v-img>

    <v-card-title>
      {{ beguda.strDrink }}
      <v-card-text class="bg-surface-light pt-4">
        {{ beguda.strDrinkThumb }}
      </v-card-text>
    </v-card-title>
  </v-card>
</template>

<script setup>
import { defineProps } from 'vue';

const props = defineProps({
  beguda: {
    type: Object,
    required: true
  }
});
</script>