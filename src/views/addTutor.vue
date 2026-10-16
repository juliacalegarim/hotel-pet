
<script setup>
import { ref } from 'vue';
import { useRouter, RouterLink } from 'vue-router';

const API_URL = 'http://localhost:3000';
const router = useRouter();

const novoTutor = ref({
  nome: '',
  telefone: '',
});

const salvando = ref(false);
const erro = ref('');

async function salvarTutor() {
  erro.value = '';
  salvando.value = true;

  try {
    const resposta = await fetch(`${API_URL}/tutores`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        nome: novoTutor.value.nome.trim(),
        telefone: novoTutor.value.telefone.trim(),
      }),
    });

    if (!resposta.ok) {
      throw new Error('Não foi possível cadastrar o tutor.');
    }

    // O JSON Server gera o ID automaticamente.
    await resposta.json();

    // Volta para a listagem depois de salvar.
    await router.push('/tutores');
  } catch (error) {
    erro.value = 'Erro ao salvar o tutor. Tente novamente.';
    console.error(error);
  } finally {
    salvando.value = false;
  }
}
</script>

<template>
  <div>
    <header class="mb-4">
      <h1 class="h2 fw-bold">Cadastro de Tutor</h1>
      <p class="text-body-secondary mb-0">
        Preencha os dados para cadastrar um novo tutor.
      </p>
    </header>

    <div class="card shadow-sm">
      <div class="card-body p-4">
        <div
          v-if="erro"
          class="alert alert-danger"
          role="alert"
        >
          {{ erro }}
        </div>

        <form @submit.prevent="salvarTutor">
          <div class="mb-3">
            <label
              for="nome"
              class="form-label"
            >
              Nome do tutor
            </label>

            <input
              id="nome"
              v-model="novoTutor.nome"
              type="text"
              class="form-control"
              placeholder="Digite o nome completo"
              required
              maxlength="100"
            />
          </div>

          <div class="mb-4">
            <label
              for="telefone"
              class="form-label"
            >
              Telefone
            </label>

            <input
              id="telefone"
              v-model="novoTutor.telefone"
              type="tel"
              class="form-control"
              placeholder="Ex.: (47) 99999-0000"
              required
              maxlength="20"
            />
          </div>

          <div class="d-flex flex-wrap gap-2">
            <button
              type="submit"
              class="btn btn-success"
              :disabled="salvando"
            >
              <span
                v-if="salvando"
                class="spinner-border spinner-border-sm me-2"
                aria-hidden="true"
              ></span>
              {{ salvando ? 'Salvando...' : 'Salvar tutor' }}
            </button>

            <RouterLink
              to="/tutores"
              class="btn btn-outline-secondary"
            >
              Cancelar
            </RouterLink>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>
