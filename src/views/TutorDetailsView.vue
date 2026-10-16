
<script setup>
import { onMounted, ref } from 'vue';
import { RouterLink, useRoute } from 'vue-router';

const API_URL = 'http://localhost:3000';
const route = useRoute();

const tutor = ref(null);
const carregando = ref(true);
const erro = ref('');

async function carregarTutor() {
  carregando.value = true;
  erro.value = '';

  try {
    const resposta = await fetch(
      `${API_URL}/tutores/${route.params.id}`,
    );

    if (!resposta.ok) {
      throw new Error('Tutor não encontrado.');
    }

    tutor.value = await resposta.json();
  } catch (error) {
    erro.value =
      'Não foi possível encontrar esse tutor. Verifique o ID informado.';
    console.error(error);
  } finally {
    carregando.value = false;
  }
}

onMounted(carregarTutor);
</script>

<template>
  <div>
    <header class="mb-4">
      <h1 class="h2 fw-bold">Detalhes do Tutor</h1>
      <p class="text-body-secondary mb-0">
        Informações do tutor selecionado.
      </p>
    </header>

    <div
      v-if="carregando"
      class="text-center py-4"
    >
      <div
        class="spinner-border text-success"
        role="status"
      >
        <span class="visually-hidden">Carregando...</span>
      </div>
    </div>

    <div
      v-else-if="erro"
      class="alert alert-danger"
      role="alert"
    >
      {{ erro }}
    </div>

    <div
      v-else-if="tutor"
      class="card shadow-sm"
    >
      <div class="card-header bg-success text-white">
        Dados do tutor
      </div>

      <div class="card-body">
        <p>
          <strong>ID:</strong>
          {{ tutor.id }}
        </p>

        <p>
          <strong>Nome:</strong>
          {{ tutor.nome }}
        </p>

        <p>
          <strong>Telefone:</strong>
          {{ tutor.telefone }}
        </p>

        <RouterLink
          to="/tutores"
          class="btn btn-outline-secondary"
        >
          <i class="bi bi-arrow-left me-1"></i>
          Voltar para a listagem
        </RouterLink>
      </div>
    </div>
  </div>
</template>
