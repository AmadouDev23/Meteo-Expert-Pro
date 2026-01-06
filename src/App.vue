<script setup>
import { ref } from 'vue'

// 1. VARIABLES D'ÉTAT (REACTIVES)
const city = ref('')           // Stocke ce que l'utilisateur tape
const weather = ref(null)      // Stocke les données reçues de l'API
const error = ref(null)        // Stocke un message d'erreur si besoin

// TA CLÉ API (Colle ta clé ici entre les guillemets)
const API_KEY = '45721f5ae350251a6bd860f661e47559'

// 2. FONCTION POUR RÉCUPÉRER LA MÉTÉO
const getWeather = async () => {
  // Réinitialiser les erreurs et résultats précédents
  error.value = null
  weather.value = null

  if (city.value === '') return // Ne rien faire si le champ est vide

  try {
    // Appel à l'API (fetch)
    const response = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${city.value}&units=metric&lang=fr&appid=${API_KEY}`
    )

    if (!response.ok) {
      throw new Error('Ville introuvable')
    }

    // Conversion de la réponse en JSON
    const data = await response.json()
    weather.value = data // On stocke les données pour les afficher

  } catch (err) {
    error.value = "Impossible de trouver cette ville. Vérifiez l'orthographe."
  }
}
</script>

<template>
  <div class="app-container">

    <h1>Météo Express 🌦️</h1>

    <div class="search-box">
      <input
          v-model="city"
          @keyup.enter="getWeather"
          type="text"
          placeholder="Entrez une ville (ex: Paris)..."
      />
      <button @click="getWeather">Rechercher</button>
    </div>

    <div v-if="error" class="error-msg">
      {{ error }}
    </div>

    <div v-if="weather" class="weather-card">
      <h2>{{ weather.name }}, {{ weather.sys.country }}</h2>

      <div class="temp">
        {{ Math.round(weather.main.temp) }}°C
      </div>

      <div class="description">
        <img
            :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@2x.png`"
            alt="icon"
        >
        <p>{{ weather.weather[0].description }}</p>
      </div>

      <div class="details">
        <p>Humidité: <strong>{{ weather.main.humidity }}%</strong></p>
        <p>Vent: <strong>{{ weather.wind.speed }} km/h</strong></p>
      </div>
    </div>

  </div>
</template>

<style scoped>
/* 3. STYLE CSS SIMPLE ET PROPRE */
.app-container {
  max-width: 400px;
  margin: 50px auto;
  text-align: center;
  font-family: 'Segoe UI', sans-serif;
  color: #333;
}

h1 {
  margin-bottom: 20px;
  color: #2c3e50;
}

.search-box {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin-bottom: 30px;
}

input {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  width: 70%;
  font-size: 16px;
}

button {
  padding: 10px 20px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  transition: background 0.3s;
}

button:hover {
  background-color: #3aa876;
}

.weather-card {
  background: linear-gradient(to bottom right, #e0f7fa, #ffffff);
  padding: 30px;
  border-radius: 15px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.1);
  animation: fadeIn 0.5s ease-in;
}

.temp {
  font-size: 4rem;
  font-weight: bold;
  color: #2c3e50;
  margin: 10px 0;
}

.description {
  display: flex;
  align-items: center;
  justify-content: center;
  text-transform: capitalize;
  font-size: 1.2rem;
  margin-bottom: 20px;
}

.details {
  display: flex;
  justify-content: space-around;
  border-top: 1px solid #ddd;
  padding-top: 15px;
}

.error-msg {
  color: #e74c3c;
  background: #fadbd8;
  padding: 10px;
  border-radius: 5px;
  margin-bottom: 20px;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>