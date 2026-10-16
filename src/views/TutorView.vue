
<script setup>
import { onMounted, ref } from 'vue';
import { RouterLink } from 'vue-router';

const API_URL = 'http://localhost:3000';

const tutores = ref([]);
const carregando = ref(false);
const erro = ref('');

async function carregarTutores() {
  carregando.value = true;
  erro.value = '';

  try {
    const resposta = await fetch(`${API_URL}/tutores`);

    if (!resposta.ok) {
      throw new Error('Não foi possível carregar os tutores.');
    }

    tutores.value = await resposta.json();
  } catch (error) {
    erro.value =
      'Erro ao carregar os tutores. Verifique se o JSON Server está funcionando.';
    console.error(error);
  } finally {
    carregando.value = false;
  }
}

async function excluirTutor(id) {
  const confirmar = window.confirm(
    'Deseja realmente excluir este tutor?',
  );

  if (!confirmar) {
    return;
  }

  try {
    const resposta = await fetch(`${API_URL}/tutores/${id}`, {
      method: 'DELETE',
    });

    if (!resposta.ok) {
      throw new Error('Não foi possível excluir o tutor.');
    }

    // Atualiza a listagem depois da exclusão.
    await carregarTutores();
  } catch (error) {
    erro.value = 'Não foi possível excluir o tutor.';
    console.error(error);
  }
}

onMounted(carregarTutores);
</script>

<template>
  <div>
    <header
      class="d-flex flex-column flex-md-row justify-content-between
      align-items-md-center gap-3 mb-4"
    >
      <div>
        <h1 class="h2 fw-bold">Listagem de Tutores</h1>
        <p class="text-body-secondary mb-0">
          Consulte os tutores cadastrados no sistema.
        </p>
      </div>

      <RouterLink
        to="/tutores/novo"
        class="btn btn-success"
      >
        <i class="bi bi-plus-circle me-2"></i>
        Novo tutor
      </RouterLink>
    </header>

    <div
      v-if="erro"
      class="alert alert-danger"
      role="alert"
    >
      {{ erro }}
      <button
        class="btn btn-sm btn-outline-danger ms-2"
        @click="carregarTutores"
      >
        Tentar novamente
      </button>
    </div>

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
      <p class="mt-2">Carregando tutores...</p>
    </div>

    <div
      v-else
      class="table-responsive"
    >
      <table class="table table-striped table-hover align-middle">
        <thead class="table-success">
          <tr>
            <th scope="col">ID</th>
            <th scope="col">Nome</th>
            <th scope="col">Telefone</th>
            <th scope="col">Ações</th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="tutor in tutores"
            :key="tutor.id"
          >
            <td>{{ tutor.id }}</td>
            <td>{{ tutor.nome }}</td>
            <td>{{ tutor.telefone }}</td>

            <td>
              <div class="d-flex flex-wrap gap-2">
                <RouterLink
                  :to="`/tutores/${tutor.id}`"
                  class="btn btn-primary btn-sm"
                >
                  <i class="bi bi-eye me-1"></i>
                  Visualizar
                </RouterLink>

                <button
                  type="button"
                  class="btn btn-danger btn-sm"
                  @click="excluirTutor(tutor.id)"
                >
                  <i class="bi bi-trash me-1"></i>
                  Remover
                </button>
              </div>
            </td>
          </tr>

          <tr v-if="tutores.length === 0">
            <td
              colspan="4"
              class="text-center text-body-secondary py-4"
            >
              Nenhum tutor cadastrado.
              <RouterLink to="/tutores/novo">
                Cadastrar primeiro tutor
              </RouterLink>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
