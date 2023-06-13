<template>
  <div class="title">
    <b-navbar toggleable="md" type="dark" variant="dark">
      <b-navbar-brand class="ml-2">
        <b>{{ appName }}</b>
      </b-navbar-brand>
    </b-navbar>

    <div class="best-transport">
      <div class="form-group">
        <label for="weight">Peso</label>
        <input type="number" id="weight" v-model="weight" min="0" step="1" />
      </div>

      <div class="form-group">
        <label for="destination">Destino:</label>
        <select v-model="destination" id="destination">
          <option
            v-for="option in destinationOptions"
            :value="option.city"
            :key="option.id"
          >
            {{ option.city }}
          </option>
        </select>
      </div>
      <button @click="analyzeFreight">Analisar</button>

      <div v-if="bestFreightEconomic">
        <h3>Melhor Frete Econômico</h3>
        <p>Transportadora: {{ bestFreightEconomic.name }}</p>
        <p>Custo Total: {{ bestFreightEconomicCost }}</p>
        <p>Tempo de Entrega: {{ bestFreightEconomic.lead_time }}</p>
      </div>

      <div v-if="bestFreightFast">
        <h3>Melhor Frete Rápido</h3>
        <p>Transportadora: {{ bestFreightFast.name }}</p>
        <p>Custo Total: {{ bestFreightFastCost }}</p>
        <p>Tempo de Entrega: {{ bestFreightFast.lead_time }}</p>
      </div>
    </div>
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
    return {
      appName: "Melhor Frete",
      weight: 0,
      destination: "",
      destinationOptions: [],
      bestFreightEconomic: null,
      bestFreightFast: null,
    };
  },
  created() {
    fetch("http://localhost:3000/transport")
      .then((response) => response.json())
      .then((data) => {
        if (Array.isArray(data)) {
          this.destinationOptions = data;
        } else {
          console.error(
            "A resposta da API não contém um array válido de transportadoras"
          );
        }
      })
      .catch((error) => {
        console.error("Ocorreu um erro ao obter os dados da API:", error);
      });
  },

  methods: {
    analyzeFreight() {
      fetch("http://localhost:3000/data.json")
        .then((response) => response.json())
        .then((data) => {
          if (data && data.transport && Array.isArray(data.transport)) {
            const cotacoesFiltradas = data.transport.filter((cotacao) => {
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

            this.bestFreightFast = cotacoesFiltradas.reduce(
              (anterior, atual) => {
                const tempoAnterior = parseInt(
                  anterior.lead_time.replace("h", "")
                );
                const tempoAtual = parseInt(atual.lead_time.replace("h", ""));
                return tempoAtual < tempoAnterior ? atual : anterior;
              }
            );
          } else {
            console.error(
              "A resposta da API não contém um array válido de transportadoras"
            );
          }
        })
        .catch((error) => {
          console.error(
            "Ocorreu um erro ao obter as cotações de frete:",
            error
          );
        });
    },

    calculateTotalCost(freight) {
      if (!freight) return 0;

      const pesoFrete =
        this.weight > 100 ? "cost_transport_heavy" : "cost_transport_light";
      const cost = parseFloat(
        freight[pesoFrete].replace("R$ ", "").replace(",", ".")
      );
      return cost;
    },
  },
  computed: {
    bestFreightEconomicCost() {
      return this.calculateTotalCost(this.bestFreightEconomic);
    },
    bestFreightFastCost() {
      return this.calculateTotalCost(this.bestFreightFast);
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
