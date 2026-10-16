<script setup>
import { onMounted, ref } from 'vue';
import { RouterLink, useRoute } from 'vue-router';

// chamando minha UseRouter

const route = useRoute();

// chamando nossa api
const API_URL = 'http://localhost:3000';
// criando variáveis reativas para armazenar os dados do pet e do tutor
const pet = ref({});
const tutor = ref({});

// função para carregar os dados do pet e do tutor
async function carregarPet() {
  try {
    const respostaPet = await fetch(`${API_URL}/pets/${route.params.id}`);
    if (!respostaPet.ok) {
      console.log('Opees, Pet Não encontrado!');
    }

    pet.value = await respostaPet.json();
    console.log(pet.value);

    const respostaTutor = await fetch(
      `${API_URL}/tutores/${pet.value.tutorId}`,
    );
    tutor.value = respostaTutor.ok
      ? await respostaTutor.json()
      : { nome: 'Tutor não encontrado' };
  } catch (erro) {
    console.error('Erro ao carregar os dados do pet e do tutor:', erro);
  }
}

onMounted(carregarPet);
</script>

<template>
  <h1>Nome: {{ pet.nome }}</h1>
  <p>Espécie: {{ pet.especie }}</p>
  <!-- <p>Tutor: {{ nomeDoTutor(pet.tutorId) }}</p> -->
</template>