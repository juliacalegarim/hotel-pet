<script setup>
import { computed, ref } from 'vue';
import { RouterLink, useRouter } from 'vue-router';

const API_URL = 'http://localhost:3000';
const router = useRouter();

const tentouSalvar = ref(false);
const salvando = ref(false);
const erroServidor = ref('');

const form = ref({
  nome: '',
  telefone: '',
  cpfCnpj: '',
  email: '',
  nascimento: '',
  rua: '',
  numero: '',
  bairro: '',
  cidade: '',
  estado: '',
  cep: '',
});

const estados = [
  { sigla: 'AC', nome: 'Acre' },
  { sigla: 'AL', nome: 'Alagoas' },
  { sigla: 'AP', nome: 'Amapá' },
  { sigla: 'AM', nome: 'Amazonas' },
  { sigla: 'BA', nome: 'Bahia' },
  { sigla: 'CE', nome: 'Ceará' },
  { sigla: 'DF', nome: 'Distrito Federal' },
  { sigla: 'ES', nome: 'Espírito Santo' },
  { sigla: 'GO', nome: 'Goiás' },
  { sigla: 'MA', nome: 'Maranhão' },
  { sigla: 'MT', nome: 'Mato Grosso' },
  { sigla: 'MS', nome: 'Mato Grosso do Sul' },
  { sigla: 'MG', nome: 'Minas Gerais' },
  { sigla: 'PA', nome: 'Pará' },
  { sigla: 'PB', nome: 'Paraíba' },
  { sigla: 'PR', nome: 'Paraná' },
  { sigla: 'PE', nome: 'Pernambuco' },
  { sigla: 'PI', nome: 'Piauí' },
  { sigla: 'RJ', nome: 'Rio de Janeiro' },
  { sigla: 'RN', nome: 'Rio Grande do Norte' },
  { sigla: 'RS', nome: 'Rio Grande do Sul' },
  { sigla: 'RO', nome: 'Rondônia' },
  { sigla: 'RR', nome: 'Roraima' },
  { sigla: 'SC', nome: 'Santa Catarina' },
  { sigla: 'SP', nome: 'São Paulo' },
  { sigla: 'SE', nome: 'Sergipe' },
  { sigla: 'TO', nome: 'Tocantins' },
];

function somenteNumeros(valor) {
  return valor.replace(/\D/g, '');
}

function aplicarMascaraTelefone(valor) {
  const numeros = somenteNumeros(valor).slice(0, 11);

  if (numeros.length <= 2) return numeros;

  if (numeros.length <= 6) {
    return `(${numeros.slice(0, 2)}) ${numeros.slice(2)}`;
  }

  if (numeros.length <= 10) {
    return `(${numeros.slice(0, 2)}) ${numeros.slice(2, 6)}-${numeros.slice(6)}`;
  }

  return `(${numeros.slice(0, 2)}) ${numeros.slice(2, 7)}-${numeros.slice(7)}`;
}

function aplicarMascaraCpfCnpj(valor) {
  const numeros = somenteNumeros(valor).slice(0, 14);

  if (numeros.length <= 11) {
    return numeros
      .replace(/^(\d{3})(\d)/, '$1.$2')
      .replace(/^(\d{3})\.(\d{3})(\d)/, '$1.$2.$3')
      .replace(/\.(\d{3})(\d)/, '.$1-$2');
  }

  return numeros
    .replace(/^(\d{2})(\d)/, '$1.$2')
    .replace(/^(\d{2})\.(\d{3})(\d)/, '$1.$2.$3')
    .replace(/\.(\d{3})(\d)/, '.$1/$2')
    .replace(/(\d{4})(\d)/, '$1-$2');
}

function aplicarMascaraCep(valor) {
  const numeros = somenteNumeros(valor).slice(0, 8);

  return numeros.replace(/^(\d{5})(\d)/, '$1-$2');
}

const nomeValido = computed(() => form.value.nome.trim().length > 0);

const telefoneValido = computed(() => {
  const quantidade = somenteNumeros(form.value.telefone).length;
  return quantidade === 10 || quantidade === 11;
});

const emailValido = computed(() => {
  if (!form.value.email.trim()) return true;

  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.value.email.trim());
});

const cpfCnpjValido = computed(() => {
  const numeros = somenteNumeros(form.value.cpfCnpj);

  if (!numeros) return true;

  return numeros.length === 11 || numeros.length === 14;
});

const formularioValido = computed(() => {
  return nomeValido.value &&
    telefoneValido.value &&
    emailValido.value &&
    cpfCnpjValido.value;
});

function erroNome() {
  return tentouSalvar.value && !nomeValido.value;
}

function erroTelefone() {
  return tentouSalvar.value && !telefoneValido.value;
}

function erroEmail() {
  return tentouSalvar.value && !emailValido.value;
}

function erroCpfCnpj() {
  return tentouSalvar.value && !cpfCnpjValido.value;
}

async function salvarTutor() {
  tentouSalvar.value = true;
  erroServidor.value = '';

  if (!formularioValido.value || salvando.value) return;

  salvando.value = true;

  const dados = {
    nome: form.value.nome.trim(),
    telefone: somenteNumeros(form.value.telefone),
  };

  // Os campos abaixo são opcionais.
  if (form.value.cpfCnpj.trim()) {
    dados.cpfCnpj = form.value.cpfCnpj;
  }

  if (form.value.email.trim()) {
    dados.email = form.value.email.trim();
  }

  if (form.value.nascimento) {
    dados.dataNascimento = form.value.nascimento;
  }

  if (form.value.rua.trim()) dados.rua = form.value.rua.trim();
  if (form.value.numero.trim()) dados.numero = form.value.numero.trim();
  if (form.value.bairro.trim()) dados.bairro = form.value.bairro.trim();
  if (form.value.cidade.trim()) dados.cidade = form.value.cidade.trim();
  if (form.value.estado) dados.estado = form.value.estado;
  if (form.value.cep.trim()) dados.cep = form.value.cep;

  try {
    const resposta = await fetch(`${API_URL}/tutores`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(dados),
    });

    if (!resposta.ok) {
      throw new Error('Não foi possível cadastrar o tutor.');
    }

    await router.push('/tutores');
  } catch (error) {
    erroServidor.value = error.message;
  } finally {
    salvando.value = false;
  }
}
</script>

<template>
  <div class="cadastro-tutor">
    <header class="mb-4">
      <h1 class="titulo-pagina">Cadastrar Tutor</h1>
      <p class="subtitulo-pagina">
        Preencha os dados para cadastrar um novo tutor.
      </p>
    </header>

    <form @submit.prevent="salvarTutor" novalidate>
      <section class="mb-4">
        <h2 class="titulo-secao">Dados do Tutor</h2>

        <div class="row g-3">
          <div class="col-md-4">
            <label for="idTutor" class="form-label">ID</label>
            <input
              id="idTutor"
              type="text"
              class="form-control campo-id"
              value="Gerado automaticamente"
              disabled
            />
          </div>

          <div class="col-md-8">
            <label for="nome" class="form-label">Nome Completo</label>
            <input
              id="nome"
              v-model="form.nome"
              type="text"
              class="form-control"
              :class="{ 'is-invalid': erroNome() }"
              placeholder="Digite o nome completo"
              required
            />
            <div v-if="erroNome()" class="invalid-feedback">
              Ops, campo obrigatório, por favor preencha-o!
            </div>
          </div>

          <div class="col-md-6">
            <label for="cpfCnpj" class="form-label">CPF / CNPJ</label>
            <input
              id="cpfCnpj"
              :value="form.cpfCnpj"
              @input="form.cpfCnpj = aplicarMascaraCpfCnpj($event.target.value)"
              type="text"
              class="form-control"
              :class="{ 'is-invalid': erroCpfCnpj() }"
              placeholder="Digite o CPF ou CNPJ"
            />
            <div v-if="erroCpfCnpj()" class="invalid-feedback">
              Informe um CPF com 11 dígitos ou CNPJ com 14 dígitos.
            </div>
            <div class="form-text">
              A máscara muda conforme a quantidade de dígitos.
            </div>
          </div>

          <div class="col-md-6">
            <label for="telefone" class="form-label">
              Telefone / Celular / WhatsApp
            </label>
            <input
              id="telefone"
              :value="form.telefone"
              @input="form.telefone = aplicarMascaraTelefone($event.target.value)"
              type="tel"
              class="form-control"
              :class="{ 'is-invalid': erroTelefone() }"
              placeholder="(00) 00000-0000"
              required
            />
            <div v-if="erroTelefone()" class="invalid-feedback">
              Ops, campo obrigatório, por favor preencha-o!
            </div>
            <div class="form-text">Formato: (00) 00000-0000</div>
          </div>

          <div class="col-md-6">
            <label for="email" class="form-label">E-mail</label>
            <input
              id="email"
              v-model="form.email"
              type="email"
              class="form-control"
              :class="{ 'is-invalid': erroEmail() }"
              placeholder="nome@example.com"
            />
            <div v-if="erroEmail()" class="invalid-feedback">
              Informe um e-mail válido.
            </div>
          </div>

          <div class="col-md-6">
            <label for="nascimento" class="form-label">
              Data de Nascimento
            </label>
            <input
              id="nascimento"
              v-model="form.nascimento"
              type="date"
              class="form-control"
            />
          </div>
        </div>
      </section>

      <section class="mb-4">
        <h2 class="titulo-secao">Endereço</h2>

        <div class="row g-3">
          <div class="col-md-8">
            <label for="rua" class="form-label">Rua</label>
            <input
              id="rua"
              v-model="form.rua"
              type="text"
              class="form-control"
              placeholder="Digite o nome da rua"
            />
          </div>

          <div class="col-md-4">
            <label for="numero" class="form-label">Número</label>
            <input
              id="numero"
              v-model="form.numero"
              type="text"
              class="form-control"
              placeholder="Número"
            />
          </div>

          <div class="col-md-6">
            <label for="bairro" class="form-label">Bairro</label>
            <input
              id="bairro"
              v-model="form.bairro"
              type="text"
              class="form-control"
              placeholder="Digite o bairro"
            />
          </div>

          <div class="col-md-6">
            <label for="cidade" class="form-label">Cidade</label>
            <input
              id="cidade"
              v-model="form.cidade"
              type="text"
              class="form-control"
              placeholder="Digite a cidade"
            />
          </div>

          <div class="col-md-6">
            <label for="estado" class="form-label">Estado</label>
            <select
              id="estado"
              v-model="form.estado"
              class="form-select"
            >
              <option value="">Selecione</option>
              <option
                v-for="estado in estados"
                :key="estado.sigla"
                :value="estado.sigla"
              >
                {{ estado.sigla }} - {{ estado.nome }}
              </option>
            </select>
          </div>

          <div class="col-md-6">
            <label for="cep" class="form-label">CEP</label>
            <input
              id="cep"
              :value="form.cep"
              @input="form.cep = aplicarMascaraCep($event.target.value)"
              type="text"
              class="form-control"
              placeholder="00000-000"
            />
          </div>
        </div>
      </section>

      <div class="alert alert-info py-2 d-flex align-items-center gap-2">
        <i class="bi bi-info-circle-fill"></i>
        <span>
          Preencha corretamente Nome e Telefone para habilitar o cadastro.
        </span>
      </div>

      <div v-if="erroServidor" class="alert alert-danger">
        {{ erroServidor }}
      </div>

      <div class="d-flex justify-content-end gap-2">
        <RouterLink to="/tutores" class="btn btn-outline-secondary">
          Cancelar
        </RouterLink>

        <button
          type="submit"
          class="btn btn-primary"
          :disabled="!formularioValido || salvando"
        >
          {{ salvando ? 'Cadastrando...' : 'Cadastrar Tutor' }}
        </button>
      </div>
    </form>
  </div>
</template>

<style scoped>
.cadastro-tutor {
  color: #172033;
}

.titulo-pagina {
  margin-bottom: 0.2rem;
  font-size: 2.2rem;
  font-weight: 700;
}

.subtitulo-pagina {
  margin-bottom: 0;
  color: #687386;
  font-size: 1.05rem;
}

.titulo-secao {
  margin-bottom: 1rem;
  padding-bottom: 0.45rem;
  border-bottom: 1px solid #dee4eb;
  font-size: 1.25rem;
  font-weight: 700;
}

.cadastro-tutor .form-label {
  margin-bottom: 0.4rem;
  font-weight: 600;
}

.cadastro-tutor .form-control,
.cadastro-tutor .form-select {
  min-height: 43px;
  border-color: #d5deea;
}

.cadastro-tutor .form-control:focus,
.cadastro-tutor .form-select:focus {
  border-color: #86b7fe;
  box-shadow: 0 0 0 0.2rem rgb(13 110 253 / 15%);
}

.cadastro-tutor .campo-id {
  background-color: #edf0f4;
  color: #7b8798;
}

.cadastro-tutor .btn-primary {
  background-color: #0d6efd;
}

.cadastro-tutor .form-text {
  color: #758195;
}
</style>