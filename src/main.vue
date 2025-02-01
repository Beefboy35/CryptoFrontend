<template>
  <div class="form">
    <input type="text" placeholder="Enter TICKER" class="main-input" v-model="inputValue">
    <button @click=getByTicker class="submit">Submit</button>
    <div class="arrow-pointer"><div v-for="i in numberOfArrows" :key="i" class="arrow-part">&#x25BC</div></div>
  </div>
  <div v-if="isLoading" class="loading"><span class="loading-content">Loading<span class="loading-dots">.</span>
    <span class="loading-dots">.</span>
    <span class="loading-dots">.</span>
    </span>
  </div>
  <div v-if="showDetails" class="card">
    <span @click="closeDetails" id="close-card">✖️</span>
    <h4>name: {{ detailed.name }}</h4>
    <h4>symbol: {{ detailed.symbol }}</h4>
    <h4>Total supply: {{ detailed.total_supply }}</h4>
    <h4>Price: {{ detailed.price.toFixed(4) }}$</h4>
    <h4>24H Volume: {{ detailed.volume_24h.toFixed(4) }}$</h4>
    <h4>Capitalization: {{ detailed.market_cap.toFixed(4) }}$</h4>
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
import './main.css';
import './input.css';
import {ref, onMounted, reactive} from 'vue';
import axios from 'axios';
export default {
  name: 'CryptoMain',
  setup() {
    const apiUrl = import.meta.env.APP_API_URL;
    const numberOfArrows = ref(9);
    const inputValue = ref('');
    const cryptos = ref([]);
    const getByTicker = async () => {
      try {
        isLoading.value = true;
        const response = await axios.get(`${apiUrl}/list_by_ticker/${inputValue.value}`);
        if (response.status === 200) {
          cryptos.value =  response.data;
        }
      } catch (error) {
        alert(error.message);
        console.error(error);
      } finally {
        isLoading.value = false;
      }
    }
    const imageUrl =  (id) => `https://s2.coinmarketcap.com/static/img/coins/64x64/${id}.png`;
    const isLoading = ref(false);
    const showDetails = ref(false);
    const detailed = reactive({});
    const closeDetails = () => {
      showDetails.value = false;
    }
    const showCryptoDetails = async (id) => {
      isLoading.value = true;
      try {
        const response = await axios.get(`${apiUrl}/curr/${id}`);
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
          alert("Error uploading data");
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
        const response = await axios.get(`${apiUrl}/list`);
        cryptos.value = response.data
      } catch (e) {
        console.error("Something went wrong, details: ", e)
        alert("Something went wrong, details: ", e)
      } finally {
        isLoading.value = false
      }
    });
    return { numberOfArrows, getByTicker, inputValue, imageUrl, isLoading, cryptos,  showCryptoDetails, detailed, showDetails, closeDetails };
  }
};
</script>
<style scoped>

@media (min-width: 1200px) {
  .crypto-list{
    width: 300px;
    height: 200px;
  }
}


@media (min-width: 768px) and (max-width: 1199px) {
  .crypto-list {
    width: 200px;
    height: 150px;
  }
  .crypto-icon {
    height: 25px;
    width: 25px;
    float: left;

    margin: 5px 10px;

  }
  .card {
    margin-right: 10px;
  }
  .form {
    left: -10%;
    max-width: 220px;
  }
}


@media (max-width: 767px) {
  .crypto-list {
    width: 100px;
    height: auto;
  }
  .crypto-icon {
    height: 20px;
    width: 20px;
    position: absolute;
    left: 25%;
  }
  .form {
    left: -45%;
    max-width: 175px;
  }

  .card {
    margin-right: 20px;
  }
}


</style>
