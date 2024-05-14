<template>
  <div class="radio-wrapper">
    <v-container>
     <v-row>
        <!-- Titolo Preferiti -->
        <v-col cols="12">
          <h2 class="titleR">✮Preferiti✮</h2>
        </v-col>
      </v-row>
      <!-- Barra di ricerca -->
      <v-text-field v-model="search" label="Cerca" prepend-inner-icon="mdi-magnify" variant="outlined" hide-details
        single-line @input="filterFavorites"></v-text-field>
      <v-row>
        <!-- Scheda per ogni radio preferita -->
        <v-col v-for="(favorite, index) in filteredFavorites" :key="index" cols="12" sm="6" md="6">
          <v-card class="radio-card" :class="{ 'active-radio': index === activeRadioIndex }">
            <v-row no-gutters>
              <v-col cols="8">
                <v-card-title>{{ favorite.name }}</v-card-title>
                <!-- Aggiungi altri dettagli della radio se necessario -->
                <v-card-text>
                  <p>{{ favorite.tags }}</p>
                  <div class="text-center align-end"
                    style="position: absolute; bottom: 0; width: 60%; margin-bottom: 30px;">
                    
                    <!-- Bottone per avviare/interrompere la riproduzione -->
                    <v-btn :style="{ margin: '0 10px' }" icon @click="togglePlayPause(favorite, index)"
                      :color="isPlaying(favorite) ? 'blue' : ''">
                      <v-icon v-if="isPlaying(favorite)">mdi-pause</v-icon>
                      <v-icon v-else>mdi-play</v-icon>
                    </v-btn>

                    <!-- Bottone per rimuovere dai preferiti -->
                    <v-btn :style="{ marginRight: display && display.mdAndUp.value ? '10px' : '0' }" icon
                      @click="removeFromFavorites(index)" color="error">
                      <v-icon>mdi-delete</v-icon>
                    </v-btn>

                  </div>
                </v-card-text>
              </v-col>
              <v-col cols="4">
                <a :href="favorite.homepage" target="_blank">
                  <!-- Immagine della radio con link alla homepage -->
                  <v-img :src="favorite.favicon ? getFaviconUrl(favorite) : '/si.png'" aspect-ratio="1/1"
                    style="margin: 10px;"></v-img>
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
import { useDisplay } from 'vuetify';

export default {
  name: 'FavoritesPage',
  data() {
    return {
      favorites: [],
      filteredFavorites: [],
      search: '',
      selectedRadio: null,
      audio: null,
      sheet: false,
      activeRadioIndex: null // Indice della radio attiva
    }
  },
  methods: {
    // Rimuove una radio dalla lista dei preferiti
    removeFromFavorites(index) {
      console.log('Rimozione dalla lista dei preferiti:', this.favorites[index].name);
      this.favorites.splice(index, 1);
      localStorage.setItem('favorites', JSON.stringify(this.favorites));
      this.filterFavorites(); // Aggiorna i preferiti filtrati dopo la rimozione
    },
    // Filtra i preferiti in base alla ricerca
    filterFavorites() {
      const searchText = this.search.toLowerCase();
      this.filteredFavorites = this.favorites.filter(favorite => {
        return (favorite.name && favorite.name.toLowerCase().includes(searchText)) ||
       (favorite.tags && favorite.tags.toLowerCase().includes(searchText));
      });
    },
    // Avvia/interrompe la riproduzione di una radio
    togglePlayPause(favorite, index) {
      if (this.isPlaying(favorite)) {
        this.stopRadio(favorite);
      } else {
        this.playRadio(favorite);
        this.activeRadioIndex = index; // Imposta l'indice della radio attiva
      }
    },
    // Verifica se una radio è in riproduzione
    isPlaying(favorite) {
      return this.selectedRadio === favorite && this.audio && !this.audio.paused;
    },
    // Ottiene l'URL della favicon della radio
    getFaviconUrl(favorite) {
      return favorite.favicon || '/default_favicon.png';
    },
    // Avvia la riproduzione della radio
    playRadio(favorite) {
      if (this.audio) {
        this.stopRadio();
      }
      this.audio = new Audio(favorite.url_resolved);
      this.audio.play();
      this.sheet = true;
      this.selectedRadio = favorite;
    },
    // Interrompe la riproduzione della radio
    stopRadio() {
      if (this.audio) {
        this.audio.pause();
        this.audio = null;
        this.sheet = false;
        this.selectedRadio = null;
      }
    },
  },
  // Inizializzazione dei dati e filtraggio dei preferiti al caricamento della pagina
  created() {
    const storedFavorites = JSON.parse(localStorage.getItem('favorites'));
    this.favorites = storedFavorites ? storedFavorites : [];
    this.filterFavorites();
  },
  // Configurazione per l'uso delle dimensioni dello schermo
  setup() {
    const display = useDisplay();
    return { display };
  }
}
</script>

<style>
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
      background-color: white;
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