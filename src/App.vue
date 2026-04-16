<script setup>
import { ref, onMounted, computed } from "vue";
import AllocationCard from "./components/AllocationCard.vue";
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

    if (!response.ok) {
      throw new Error(`Request failed with status ${response.status}`);
    }

    const data = await response.json();

    const btcRate = Number(data?.data?.rates?.BTC);
    const ethRate = Number(data?.data?.rates?.ETH);

    if (!Number.isFinite(btcRate) || !Number.isFinite(ethRate)) {
      throw new Error("Invalid exchange rate data received");
    }

    rates.value = {
      BTC: Number(data.data.rates.BTC),
      ETH: Number(data.data.rates.ETH),
    };
  } catch (err) {
    error.value = `Error fetching exchange rates: ${err.message}`;
    console.log("Error fetching exchange rates:", err);
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
    const holdingsValue = Math.max(0, parseFloat(holdings.value));
    holdings.value = holdingsValue.toFixed(2);
  }
}
</script>

<template>
  <main class="page">
    <section class="header">
      <h1>Asset Allocation Calculator</h1>
      <p id="description">
        Enter an amount in USD and we’ll calculate a 70/30 allocation using live
        Coinbase rates.
      </p>
    </section>

    <section class="input-card" aria-label="input-card">
      <div class="field-group">
        <label id="investable-assets-label" for="investable-assets-input">
          Investable Assets ($):
        </label>
        <input
          id="investable-assets-input"
          v-model="holdings"
          type="number"
          placeholder="Enter amount"
          min="0"
          step="0.01"
          @blur="formatHoldings"
          inputmode="decimal"
        />
        <p id="investable-assets-help" class="helper-text">
          Enter the total amount in USD you want to split between Bitcoin and
          Ethereum.
        </p>
      </div>
    </section>

    <section class="status-message" aria-live="polite">
      <p v-if="loading">Fetching exchange rates...</p>
      <p v-else-if="error" class="error-message" role="alert">{{ error }}</p>
    </section>

    <section
      v-if="!loading && !error"
      class="results"
      aria-label="allocation-results"
    >
      <AllocationCard
        input-id="btc-allocation"
        label="70% BTC Allocation (BTC):"
        :value="holdings ? btcAmount.toFixed(8) : ''"
        :usd-allocation="holdings ? formatCurrency(btcUsd) : '-'"
        :rate="rates.BTC ? rates.BTC.toFixed(8) : '—'"
        rate-label="BTC/USD"
      />

      <AllocationCard
        input-id="eth-allocation"
        label="30% ETH Allocation (ETH):"
        :value="holdings ? ethAmount.toFixed(8) : ''"
        :usd-allocation="holdings ? formatCurrency(ethUsd) : '-'"
        :rate="rates.ETH ? rates.ETH.toFixed(8) : '—'"
        rate-label="ETH/USD"
      />
    </section>
  </main>
</template>
