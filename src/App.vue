
<template>
  <div v-if="isLoading" class="loading">Loading...</div>
  <div v-if="showDetails" class="card">
      <h4>name: {{ detailed.name }}</h4>
    <h4>symbol: {{ detailed.symbol }}</h4>
    <h4>Total supply: {{ detailed.total_supply }}</h4>
    <h4>Price: {{ detailed.price }}</h4>
    <h4>24H Volume: {{ detailed.volume_24h }}</h4>
    <h4>Capitalization: {{ detailed.market_cap}}</h4>
    <h4>Dominance: {{ detailed.dominance }}%</h4>
    </div>
  <div class="crypto-list">
    <div v-for="crypto in cryptos" :key="crypto.id" @click="showCryptoDetails(crypto.id)" class="crypto-item">
        <img :src="imageUrl(crypto.id)" alt="icon" class="crypto-icon">
        <h4>Name: {{ crypto.name }}</h4>
        <h4>Ticker: {{ crypto.symbol }}</h4>
      <h4>Price: {{ crypto.price }}</h4>
    </div>
  </div>
</template>

<script>
import {ref, onMounted, reactive} from 'vue';
import axios from 'axios';
export default {
  setup() {
    const isLoading = ref(false);
    const cryptos = ref([]);
    const showDetails = ref(false);
    const detailed = reactive({});
    const imageUrl = async (id) => `https://s2.coinmarketcap.com/static/img/coins/64x64/${id}.png`;
    const showCryptoDetails = async (id) => {
      isLoading.value = true;
      try {
        const response = await axios.get(`http://localhost:8000/curr/${id}`);
          if (response.status === 200) {
          detailed.name = response.data["name"];
          detailed.symbol = response.data["symbol"];
          detailed.price = response.data["quote"]["USD"]["price"];
          detailed.total_supply = response.data["total_supply"];
          detailed.volume_24h = response.data["quote"]["USD"]["volume_24h"];
          detailed.market_cap = response.data["quote"]["USD"]["market_cap"];
          detailed.dominance = response.data["quote"]["USD"]["market_cap_dominance"];
          showDetails.value = true;
        } else {
          alert("Ошибка загрузки данных");
        }
      } catch (e) {
        console.error("Error loading additional info:", e)
        alert("Error loading additional info");
      } finally {
        isLoading.value = false;
      }
    }
        onMounted(async () => {
      try {
        isLoading.value = true
        const response = await axios.get("http://localhost:8000/list");
        cryptos.value = response.data
        card.value = "big dick";
      } catch (e) {
        console.error("Something went wrong, details: ", e)
        alert("Something went wrong, details: ", e)
      } finally {
        isLoading.value = false
      }
    });
    return { isLoading, cryptos, imageUrl, showCryptoDetails, detailed, showDetails };
  }
};
</script>
<style scoped>
.loading {
  padding: 0;
  margin: 0;
  width: 100%;
  position: fixed;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgba(0, 0 ,0, 0.4);
  color: white;
  font-size: 50px;
}
.crypto-list {
  min-height: 100vh;
  width: 340px;
  background: linear-gradient(to bottom right, #ffffff, #f4f2e0);
  border-right: 1px solid #999;
}
.card {
  display: grid;
  grid-auto-rows: 50px 50px;
  max-width: 400px;
  height: auto;
  position: fixed;
  left: 50%;
  top: 40%;
  border: black 1px solid;
  border-radius: 15px;
  padding: 5px 10px;
}
.crypto-icon {
  height: 40px;
  width: 40px;
  position: absolute;
  top: 8%;
  left: 6%;
  margin-left: auto;
}


@keyframes color-shadow {
  0% {
   box-shadow: 0 3px 5px 0 rgba(6, 36, 67, 0.58);
  }
  33% {
     box-shadow: 0 3px 5px 0 rgba(4, 69, 90, 0.6);
  }
  66% {
    box-shadow: 0 3px 5px 0 rgba(0, 199, 205, 0.66);
  }
  100% {
    box-shadow: 0 3px 5px 0 rgba(5, 96, 108, 0.6);
  }
}
.crypto-item {
  box-shadow: 0 0 1px 0 rgba(5, 17, 51, 0.6);
  animation: color-shadow 3s infinite linear;
  border: black 1px solid;
  margin: 10px 10px 10px 10px;
  max-width: 300px;
  padding: 2px;
  display: flex;
  position: relative;
  flex-direction: column;
  text-align: center;
  justify-content: space-between;
  font-family: cursive;
  border-radius: 10px/15px;
}

.crypto-item:hover {
  animation: none;
  border: black 3px solid;
  cursor: pointer;
}
</style>
