<script setup>
import { ref, onMounted, computed } from "vue";
import "./style.css";

const holdings = ref("");
const rates = ref({ BTC: null, ETH: null });
const loading = ref(false);
const error = ref("");

const btcUsd = computed(() => Number(holdings.value || 0) * 0.7);
const ethUsd = computed(() => Number(holdings.value || 0) * 0.3);

const btcAmount = computed(() =>
  rates.value.BTC ? btcUsd.value * rates.value.BTC : 0,
);

const ethAmount = computed(() =>
  rates.value.ETH ? ethUsd.value * rates.value.ETH : 0,
);

async function fetchRates() {
  loading.value = true;
  error.value = "";

  try {
    const response = await fetch(
      "https://api.coinbase.com/v2/exchange-rates?currency=USD",
    );
    const data = await response.json();

    rates.value = {
      BTC: Number(data.data.rates.BTC),
      ETH: Number(data.data.rates.ETH),
    };
  } catch (error) {
    error.value = `Error fetching exchange rates: ${error.message}`;
    console.log("Error fetching exchange rates:", error);
  } finally {
    loading.value = false;
  }
}

onMounted(fetchRates);

function formatCurrency(amount) {
  return new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
  }).format(amount);
}

function formatHoldings() {
  if (holdings.value) {
    holdings.value = parseFloat(holdings.value).toFixed(2);
  }
}
</script>

<template>
  <main class="page">
    <section class="header">
      <h1>Asset Allocation Calculator</h1>
      <p>
        Enter an amount in USD and we’ll calculate a 70/30 allocation using live
        Coinbase rates.
      </p>
    </section>
    <section class="input-card">
      <div class="field-group">
        <label for="investable-assets">Investable Assets ($):</label>
        <input
          id="investable-assets"
          v-model="holdings"
          type="number"
          placeholder="Enter amount"
          min="0"
          step="0.01"
          @blur="formatHoldings"
        />
      </div>
    </section>
    <section class="status-message">
      <p v-if="loading">Fetching exchange rates...</p>
      <p v-else-if="error" class="error-message">{{ error }}</p>
    </section>
    <section v-if="!loading && !error" class="results">
      <div class="allocation-card">
        <div class="field-group">
          <label for="btc-allocation">70% BTC Allocation (BTC):</label>
          <input
            id="btc-allocation"
            :value="holdings ? btcAmount.toFixed(8) : ''"
            readonly
          />
        </div>
        <div class="allocation-info">
          <p>
            <b>Allocation (USD):</b>
            {{ holdings ? formatCurrency(btcUsd) : "-" }}
          </p>
          <p>
            <b>Rate:</b> {{ rates.BTC ? rates.BTC.toFixed(8) : "—" }} BTC/USD
          </p>
        </div>
      </div>
      <div class="allocation-card">
        <div class="field-group">
          <label for="eth-allocation">30% ETH Allocation (ETH):</label>
          <input
            id="eth-allocation"
            :value="holdings ? ethAmount.toFixed(8) : ''"
            readonly
          />
        </div>
        <div class="allocation-info">
          <p>
            <b>Allocation (USD):</b>
            {{ holdings ? formatCurrency(ethUsd) : "-" }}
          </p>
          <p>
            <b>Rate:</b> {{ rates.ETH ? rates.ETH.toFixed(8) : "—" }} ETH/USD
          </p>
        </div>
      </div>
    </section>
  </main>
</template>
