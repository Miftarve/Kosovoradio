<template>
  <div class="radio-wrapper">
    <v-container>
      <br>
      <!-- Barra di ricerca -->
      <v-text-field v-model="search" label="Cerca" prepend-inner-icon="mdi-magnify" variant="outlined" hide-details
        single-line @input="filterFavorites"></v-text-field>
        <br><br>
      <v-row>
        <br><br>
        <!-- Iterazione su tutti i preferiti filtrati -->
        <v-col v-for="(favorite, index) in filteredFavorites" :key="index" cols="12" sm="6" md="4">
          <v-card class="radio-card" :class="{ 'active-radio': index === activeRadioIndex }">
            <v-row no-gutters>
              <v-col cols="8">
                <!-- Titolo della radio -->
                <v-card-title>{{ favorite.name }}</v-card-title>
                <!-- Altri dettagli della radio se necessario -->
                <v-card-text>
                  <p>{{ favorite.tags }}</p>
                  <p>{{ favorite.country }}</p>
                  <div class="text-center align-end"
                    style="position: absolute; bottom: 0; width: 60%; margin-bottom: 30px;">
                    
                    <!-- Pulsante Play/Pause -->
                    <v-btn :style="{ margin: '0 10px' }" icon @click="togglePlayPause(favorite, index)"
                      :color="isPlaying(favorite) ? 'blue' : ''">
                      <v-icon v-if="isPlaying(favorite)">mdi-pause</v-icon>
                      <v-icon v-else>mdi-play</v-icon>
                    </v-btn>

                    <!-- Pulsante per rimuovere dai preferiti -->
                    <v-btn :style="{ marginRight: display && display.mdAndUp.value ? '10px' : '0' }" icon
                      @click="removeFromFavorites(index)" color="error">
                      <v-icon>mdi-delete</v-icon>
                    </v-btn>

                  </div>
                </v-card-text>
              </v-col>
              <!-- Immagine della radio -->
              <v-col cols="4">
                <a :href="favorite.homepage" target="_blank">
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
    // Rimuovi dai preferiti
    removeFromFavorites(index) {
      console.log('Rimozione dalla lista dei preferiti:', this.favorites[index].name);
      this.favorites.splice(index, 1);
      localStorage.setItem('favorites', JSON.stringify(this.favorites));
      this.filterFavorites(); // Aggiorna i preferiti filtrati dopo la rimozione
    },
    // Filtra i preferiti in base al testo di ricerca
    filterFavorites() {
      const searchText = this.search.toLowerCase();
      this.filteredFavorites = this.favorites.filter(favorite => {
        return favorite.name.toLowerCase().includes(searchText) ||
          favorite.tags.toLowerCase().includes(searchText) ||
          favorite.country.toLowerCase().includes(searchText);
      });
    },
    // Avvia o metti in pausa la radio
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
    // Ottieni URL favicon della radio
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
    // Interrompi la riproduzione della radio
    stopRadio() {
      if (this.audio instanceof Audio) {
        this.audio.pause();
        this.audio = null; // Resetta l'elemento audio
      }
      this.sheet = false;
      this.selectedRadio = null;
      this.activeRadioIndex = null; // Resetta l'indice della radio attiva
    },

    // Metodo per gestire il cambio di route
    handleRouteChange() {
      this.stopRadio();
    },
  },
  created() {
    const storedFavorites = JSON.parse(localStorage.getItem('favorites'));
    this.favorites = storedFavorites ? storedFavorites : [];
    this.filterFavorites(); // Filtra i preferiti al caricamento della pagina
    this.$router.beforeEach((to, from, next) => {
      this.handleRouteChange();
      next();
    });
  },
  setup() {
    const display = useDisplay();
    return { display };
  }
}
</script>

<style>
body {
  background-color: #000000;
}

.radio-wrapper {
  margin-right: 5%;
  margin-left: 5%;
  background-color: rgb(0, 0, 0);
  border-radius: 20px;
}

.v-text-field {
  margin-bottom: 10px;
  border-radius: 20px;
  border: none;
  background-color: white;
}

.radio-card {
  height: 185px;
}

.radio-wrapper {
  background-color: rgb(0, 0, 0);
}

.active-radio {
  background-color: #F0F0F0;
  /* Imposta il colore di sfondo desiderato */
}
</style>
