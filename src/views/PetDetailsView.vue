<script setup>
import { computed, onMounted, ref } from 'vue';
import { RouterLink, useRoute, useRouter } from 'vue-router';

const API_URL = 'http://localhost:3000';
const route = useRoute();
const router = useRouter();

const tutorId = route.params.id;

const tutor = ref(null);
const pets = ref([]);
const nomePet = ref('');
const especiePet = ref('');
const carregando = ref(true);
const salvandoPet = ref(false);
const erro = ref('');

const especies = ['Cachorro', 'Gato'];

const tutorPossuiPets = computed(() => pets.value.length > 0);

async function carregarTutor() {
  carregando.value = true;
  erro.value = '';

  try {
    const [respostaTutor, respostaPets] = await Promise.all([
      fetch(`${API_URL}/tutores/${tutorId}`),
      fetch(
        `${API_URL}/pets?tutorId=${encodeURIComponent(tutorId)}`,
      ),
    ]);

    if (!respostaTutor.ok) {
      throw new Error('Tutor não encontrado.');
    }

    if (!respostaPets.ok) {
      throw new Error('Não foi possível carregar os pets.');
    }

    tutor.value = await respostaTutor.json();
    pets.value = await respostaPets.json();
  } catch (error) {
    erro.value = error.message;
  } finally {
    carregando.value = false;
  }
}

async function cadastrarPet() {
  erro.value = '';

  if (!nomePet.value.trim() || !especiePet.value || salvandoPet.value) {
    return;
  }

  salvandoPet.value = true;

  try {
    const resposta = await fetch(`${API_URL}/pets`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        nome: nomePet.value.trim(),
        especie: especiePet.value,
        tutorId: String(tutorId),
      }),
    });

    if (!resposta.ok) {
      throw new Error('Não foi possível cadastrar o pet.');
    }

    nomePet.value = '';
    especiePet.value = '';

    await carregarTutor();
  } catch (error) {
    erro.value = error.message;
  } finally {
    salvandoPet.value = false;
  }
}

async function removerTutor() {
  if (tutorPossuiPets.value) {
    window.alert(
      'Este tutor possui pets vinculados. Remova ou transfira os pets antes de excluir o tutor.',
    );
    return;
  }

  const confirmar = window.confirm(
    'Deseja realmente remover este tutor?',
  );

  if (!confirmar) return;

  try {
    const resposta = await fetch(`${API_URL}/tutores/${tutorId}`, {
      method: 'DELETE',
    });

    if (!resposta.ok) {
      throw new Error('Não foi possível remover o tutor.');
    }

    await router.push('/tutores');
  } catch (error) {
    erro.value = error.message;
  }
}

onMounted(carregarTutor);
</script>

<template>
  <div>
    <header
      class="d-flex flex-wrap justify-content-between align-items-start gap-3 mb-4"
    >
      <div>
        <h1 class="h2 fw-bold">Detalhes do Tutor</h1>
        <p class="text-body-secondary">
          Visualize os dados do tutor e cadastre pets vinculados.
        </p>
      </div>

      <RouterLink to="/tutores" class="btn btn-outline-primary">
        <i class="bi bi-arrow-left me-1"></i>
        Voltar para Tutores
      </RouterLink>
    </header>

    <div v-if="carregando" class="text-body-secondary">
      Carregando dados...
    </div>

    <div v-else-if="erro && !tutor" class="alert alert-danger">
      {{ erro }}
    </div>

    <template v-else-if="tutor">
      <section class="card mb-4">
        <div class="card-header fw-bold">
          Dados do Tutor
        </div>

        <div class="card-body">
          <div class="row g-3">
            <div class="col-md-4">
              <strong>ID</strong>
              <div>{{ tutor.id }}</div>
            </div>

            <div class="col-md-4">
              <strong>Nome completo</strong>
              <div>{{ tutor.nome }}</div>
            </div>

            <div class="col-md-4">
              <strong>Telefone / WhatsApp</strong>
              <div>{{ tutor.telefone }}</div>
            </div>
          </div>
        </div>
      </section>

      <section class="card mb-4">
        <div class="card-header fw-bold">
          Pets vinculados
          <span class="badge text-bg-primary ms-2">
            {{ pets.length }} pet(s)
          </span>
        </div>

        <div class="table-responsive">
          <table class="table table-striped table-hover align-middle mb-0">
            <thead>
              <tr>
                <th>ID</th>
                <th>Nome</th>
                <th>Espécie</th>
                <th>Ações</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="pet in pets" :key="pet.id">
                <td>{{ pet.id }}</td>
                <td>{{ pet.nome }}</td>
                <td>{{ pet.especie }}</td>
                <td>
                  <RouterLink
                    :to="`/pets/${pet.id}`"
                    class="btn btn-outline-primary btn-sm"
                  >
                    Visualizar Pet
                  </RouterLink>
                </td>
              </tr>

              <tr v-if="pets.length === 0">
                <td colspan="4" class="text-center py-4">
                  Nenhum pet vinculado a este tutor.
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <section class="card mb-4">
        <div class="card-header fw-bold">
          Cadastrar novo pet
        </div>

        <div class="card-body">
          <p class="text-body-secondary">
            O pet será vinculado automaticamente a
            <strong>{{ tutor.nome }}</strong>.
          </p>

          <form @submit.prevent="cadastrarPet">
            <div class="mb-3">
              <label for="nomePet" class="form-label">
                Nome do pet
              </label>

              <input
                id="nomePet"
                v-model="nomePet"
                type="text"
                class="form-control"
                placeholder="Digite o nome do pet"
                required
              />
            </div>

            <div class="mb-3">
              <label for="especiePet" class="form-label">
                Espécie
              </label>

              <select
                id="especiePet"
                v-model="especiePet"
                class="form-select"
                required
              >
                <option value="" disabled>
                  Selecione a espécie
                </option>

                <option
                  v-for="especie in especies"
                  :key="especie"
                  :value="especie"
                >
                  {{ especie }}
                </option>
              </select>
            </div>

            <button
              type="submit"
              class="btn btn-success"
              :disabled="
                !nomePet.trim() || !especiePet || salvandoPet
              "
            >
              {{ salvandoPet ? 'Cadastrando...' : 'Cadastrar pet' }}
            </button>
          </form>
        </div>
      </section>

      <div v-if="erro" class="alert alert-danger">
        {{ erro }}
      </div>

      <button
        type="button"
        class="btn btn-outline-danger"
        @click="removerTutor"
      >
        Remover Tutor
      </button>
    </template>
  </div>
</template>