<template>
  <div class="title">
    <b-navbar toggleable="lg" type="dark" variant="info">
      <b-navbar-brand class="ml-2">
        <b>{{ appName }}</b>
      </b-navbar-brand>
    </b-navbar>
  </div>
</template>

<script>
import { BNavbar, BNavbarBrand } from "bootstrap-vue";

export default {
  components: {
    BNavbar,
    BNavbarBrand,
  },
  data() {
    const appName = "";

    return {
      appName,
    };
  },
  created() {
    // Implemente aqui o GET dos dados da API REST
    // para que isso ocorra na inicialização da pagina
    this.appName = "Melhor Frete";
  },
  methods: {
    analyzeFreight() {
      fetch("http://localhost:3000/transport")
        .then((response) => response.json())
        .then((data) => {
          const cotacoesFiltradas = data.filter((cotacao) => {
            const pesoFrete =
              this.weight > 100
                ? "cost_transport_heavy"
                : "cost_transport_light";
            return (
              cotacao.city.toLowerCase() === this.destination.toLowerCase() &&
              cotacao[pesoFrete]
            );
          });

          this.bestFreightEconomic = cotacoesFiltradas.reduce(
            (anterior, atual) => {
              const custoAnterior = parseFloat(
                anterior[
                  this.weight > 100
                    ? "cost_transport_heavy"
                    : "cost_transport_light"
                ].replace("R$ ", "")
              );
              const custoAtual = parseFloat(
                atual[
                  this.weight > 100
                    ? "cost_transport_heavy"
                    : "cost_transport_light"
                ].replace("R$ ", "")
              );
              return custoAtual < custoAnterior ? atual : anterior;
            }
          );

          this.bestFreightFast = cotacoesFiltradas.reduce((anterior, atual) => {
            const tempoAnterior = parseInt(anterior.lead_time.replace("h", ""));
            const tempoAtual = parseInt(atual.lead_time.replace("h", ""));
            return tempoAtual < tempoAnterior ? atual : anterior;
          });
        })
        .catch((error) => {
          console.error(
            "Ocorreu um erro ao obter as cotações de frete:",
            error
          );
        });
    },
  },
};
</script>

<style scoped>
.title .navbar {
  background-color: #00aca6 !important;
}

.title .navbar-brand {
  margin-left: 20px;
}
</style>
