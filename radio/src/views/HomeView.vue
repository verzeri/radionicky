<script setup>
// Funzione che gestisce la pausa del video
function processPause(progress) {
  console.log(progress)
}
</script>

<template>
  <div class="radio-wrapper">
    <v-container>
      
      <v-row>
        <!-- Titolo RadioV -->
        <v-col cols="12">
          <h2 class="titleR">RadioV</h2>
        </v-col>
      </v-row>
      
      <v-row>
        <!-- Campo di ricerca -->
        <v-col cols="12">
          <!-- Campo di ricerca delle radio -->
          <v-text-field v-model="search" label="Cerca" prepend-inner-icon="mdi-magnify" variant="outlined" hide-details
            single-line dense @input="filterRadios"></v-text-field>
        </v-col>
      </v-row>
      <v-row>
        <!-- Scheda per ogni radio -->
        <v-col v-for="(radio, index) in filteredRadios" :key="index" cols="12" sm="6" md="6">
          <v-card class="radio-card">
            <v-row no-gutters>
              <v-col cols="8">
                <!-- Nome della radio -->
                <v-card-title class="text-center">{{ radio.name }}</v-card-title>
                <v-card-text>
                  <p class="text-center">{{ radio.tags }}</p>

                  <div class="text-center align-end"
                    style="position: absolute; bottom: 0; width: 60%; margin-bottom: 30px;">

                    <!-- Bottone di play/pausa -->
                    <v-btn class="mr-2" icon @click="togglePlayPause(radio)" :color="isPlaying(radio) ? 'blue' : ''">
                      <v-icon v-if="isPlaying(radio)">mdi-pause</v-icon>
                      <v-icon v-else>mdi-play</v-icon>
                    </v-btn>
                    
                    <!-- Bottone per aggiungere/rimuovere dai preferiti -->
                    <v-btn class="mr-2" icon @click="toggleFavorite(radio)" :color="isFavorite(radio) ? 'red' : ''">
                      <v-icon v-if="isFavorite(radio)" :color="isPlaying(radio) ? 'white' : ''">mdi-heart</v-icon>
                      <v-icon v-else>mdi-heart-outline</v-icon>
                    </v-btn>

                    <!-- Componente VideoPlayer per la riproduzione dei video -->
                    <VideoPlayer type="default" @pause="processPause" :link="radio.url" :progress="30" :isMuted="false"
                      :isControls="true" class="customClassName" v-if="radio.hls == '1'" />

                  </div>
                </v-card-text>
              </v-col>
              <v-col cols="4">
                <!-- Immagine della radio con link alla homepage -->
                <a :href="radio.homepage" target="_blank">
                  <v-img :src="getFaviconUrl(radio)" aspect-ratio="1/1" style="margin: 10px;"></v-img>
                </a>
              </v-col>
            </v-row>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
// Importazione del componente VideoPlayer
import { VideoPlayer } from 'vue-hls-video-player';

export default {
  name: 'HomeView',
  components: {},
  // Dati del componente
  data() {
    return {
      radios: [],
      filteredRadios: [],
      search: '',
      selectedCountry: null, // Paese selezionato
      countries: [], // Elenco dei paesi
      audio: null,
      sheet: false,
      selectedRadio: null,
      favorites: [],
    }
  },
  // Metodi del componente
  methods: {
    // Recupera l'elenco delle radio
    getRadios(country = 'Italy') {
      fetch(`https://nl1.api.radio-browser.info/json/stations/search?limit=100&hidebroken=true&order=clickcount&reverse=true&country=${country}`)
        .then(response => response.json())
        .then(data => {
          this.radios = data;
          this.filteredRadios = data;
          // Estrae l'elenco dei paesi univoci
          this.countries = Array.from(new Set(data.map(radio => radio.country)));
        });
    },
    // Ottiene l'URL della favicon della radio
    getFaviconUrl(radio) {
      return radio.favicon || '/image.png';
    },
    // Avvia la riproduzione della radio
    playRadio(radio) {
      if (this.audio instanceof Audio) {
        this.audio.pause();
        this.audio = null;
      }

      this.sheet = true;
      this.selectedRadio = radio;

      if (radio.hls === 1) {
        console.log('Using vue-hls-video-player for M3U8 format');
        console.log(radio.url);
        this.audio = null;
      } else {
        console.log('Using <audio> element for MP3 format');
        this.audio = new Audio(radio.url_resolved);
        this.audio.play();
      }
    },
    // Arresta la riproduzione della radio
    stopRadio() {
      if (this.audio instanceof Audio) {
        this.audio.pause();
        this.audio = null;
      }
      this.sheet = false;
    },
    // Filtra le radio in base alla ricerca e al paese selezionato
    filterRadios() {
      let filtered = this.radios;
      if (this.search) {
        filtered = filtered.filter(radio => radio.name.toLowerCase().includes(this.search.toLowerCase()));
      }
      if (this.selectedCountry) {
        if (this.selectedCountry === 'Italy') {
          // Visualizza solo le radio italiane se l'Italia è selezionata
          filtered = filtered.filter(radio => radio.country === 'Italy');
        } else {
          filtered = filtered.filter(radio => radio.country === this.selectedCountry);
        }
      }
      this.filteredRadios = filtered;
    },
    // Gestisce la riproduzione/pausa della radio
    togglePlayPause(radio) {
      if (this.isPlaying(radio)) {
        this.stopRadio();
      } else {
        this.stopRadio();
        this.playRadio(radio);
      }
    },
    // Verifica se una radio è in riproduzione
    isPlaying(radio) {
      return this.selectedRadio === radio && this.audio && !this.audio.paused;
    },
    // Aggiunge/rimuove una radio dai preferiti
    toggleFavorite(radio) {
      const index = this.favorites.findIndex(fav => fav.url === radio.url);
      if (index !== -1) {
        this.favorites.splice(index, 1);
      } else {
        this.favorites.push(radio);
      }
      localStorage.setItem('favorites', JSON.stringify(this.favorites));
    },
    // Verifica se una radio è nei preferiti
    isFavorite(radio) {
      return this.favorites.some(fav => fav.url === radio.url);
    },
  },
  // Metodo eseguito al momento della creazione del componente
  created() {
    this.getRadios(); // Avvia con le radio italiane
    const favorites = localStorage.getItem('favorites');
    this.favorites = favorites ? JSON.parse(favorites) : [];
  }
}
</script>

<style>
/* Stili CSS per il componente */
    body {
      background-color: #B71C1C;
      font-family: Arial, sans-serif;
      color: white;
    }
    .text-center{
      text-align:center;
    }

    .radio-wrapper {
      margin: 20px auto;
      max-width: 800px;
      padding: 20px;
      background-color: #000000;
      border-radius: 20px;
      box-shadow: 0 0 10px rgba(255, 255, 255, 0.3);
    }

    .v-text-field {
      margin-bottom: 10px;
      padding: 10px;
      border-radius: 10px;
      border: none;
      background-color: rgba(255, 255, 255, 0.9);
      color: #000000;
      width: 100%;
      box-sizing: border-box;
    }

    .v-text-field:hover {
      background-color: rgba(255, 255, 255, 1);
    }

    .radio-card {
      height: 185px; /* Aumentata l'altezza delle card */
      width: 350px; /* Impostata la larghezza al 100% */
      display: flex;
      text-align: center;
      border-radius: 10px;
      background-color: rgba(255, 255, 255, 0.1);
      cursor: pointer;
      padding: 20px;
    }

    .radio-card:hover {
      background-color: rgba(255, 255, 255, 0.3);
    }

    .radio-card img {
      max-width: 100%;
      max-height: 100%;
    }

    .radio-card-title {
      font-size: 18px;
      text-align: center;
    }
    .title {
      text-align: center;
      font-size: 24px;
      color: black; /* Cambiato il colore del titolo a nero */
      margin-bottom: 20px;
    }
    .titleR {
    font-weight: bold;
    color: white;
    font-size: 70px; /* Font più grosso */
    text-align: center;
  }
  </style>
