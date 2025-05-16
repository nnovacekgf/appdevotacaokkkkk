<template>
  <div>
    <h1>Sistema de Contagem de Votos</h1>
    <input type="file" @change="handleFileUpload" />
    <div v-if="error" class="error">{{ error }}</div>
    <div v-if="cooperativas.length">
      <h2>Lista de Cooperativas</h2>
      <ul>
        <li v-for="coop in cooperativas" :key="coop.nome">
          {{ coop.nome }} - 
          <strong>{{ coop.filiacao }}</strong>
        </li>
      </ul>
      <div class="resultado">
        <h3>Resultado</h3>
        <p>Favoráveis: {{ favoraveis.length }}</p>
        <p>Contrárias: {{ contrarias.length }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import * as XLSX from "xlsx";
import { ref } from "vue";

const cooperativas = ref([]);
const favoraveis = ref([]);
const contrarias = ref([]);
const error = ref("");

function handleFileUpload(e) {
  const file = e.target.files[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = function (evt) {
    try {
      let data = evt.target.result;
      let workbook = XLSX.read(data, { type: "binary" });
      let sheet = workbook.Sheets[workbook.SheetNames[0]];
      let json = XLSX.utils.sheet_to_json(sheet);
      processCooperativas(json);
    } catch (err) {
      error.value = "Erro ao ler o arquivo. Verifique o formato.";
    }
  };
  reader.readAsBinaryString(file);
}

function processCooperativas(data) {
  cooperativas.value = data.map((item) => ({
    nome: item["Nome"] || item["nome"] || "Sem nome",
    filiacao: item["Filiação"] || item["filiacao"] || "Não informada",
    voto: item["Voto"] || item["voto"] || "Abstenção"
  }));
  favoraveis.value = cooperativas.value.filter((c) => c.voto === "Sim");
  contrarias.value = cooperativas.value.filter((c) => c.voto === "Não");
}
</script>

<style scoped>
.error {
  color: red;
}
.resultado {
  margin-top: 1rem;
}
</style>