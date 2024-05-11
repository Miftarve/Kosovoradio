<template>
  <div class="radio-wrapper">
    <v-container>
      <br>
      <!-- Barra di ricerca -->
      <v-row>
        <!-- Campo di ricerca -->
        <v-col cols="6">
          <v-text-field v-model="search" label="Cerca" prepend-inner-icon="mdi-magnify" variant="outlined" hide-details
            single-line dense @input="filterRadios"></v-text-field>
        </v-col>
        <!-- Combobox per i paesi -->
        <v-col cols="6">
          <v-combobox v-model="selectedTags" :items="allTags" label="Seleziona tag" multiple chips dense outlined hide-details></v-combobox>
        </v-col>
      </v-row>
      <br><br>
      <v-row>
        <!-- Iterazione su tutte le radio filtrate -->
        <v-col v-for="(radio, index) in filteredRadios" :key="index" cols="12" sm="6" md="4">
          <v-card :class="{ 'playing': isPlaying(radio) }" class="radio-card">
            <v-row no-gutters>
              <v-col cols="8">
                <!-- Titolo della radio -->
                <v-card-title>{{ radio.name }}</v-card-title>
                <v-card-text>
                  <!-- Altri dettagli della radio se necessario -->
                  <p>{{ radio.tags }}</p>
                  <p>{{ radio.country }}</p>

                  <!-- Animazione dell'onda sonora -->
                  <div v-if="isPlaying(radio)" class="sound-wave">
                    <div class="bar"></div>
                    <div class="bar"></div>
                    <div class="bar"></div>
                    <div class="bar"></div>
                  </div>

                  <!-- Controlli audio -->
                  <div class="text-center align-end"
                    style="position: absolute; bottom: 0; width: 60%; margin-bottom: 30px;">
                    <!-- Bottone play/pause -->
                    <v-btn class="mr-2" icon @click="togglePlayPause(radio)" :color="isPlaying(radio) ? 'green' : ''">
                      <v-icon v-if="isPlaying(radio)">mdi-pause</v-icon>
                      <v-icon v-else>mdi-play</v-icon>
                    </v-btn>

                    <!-- Bottone preferiti -->
                    <v-btn class="mr-2" icon @click="toggleFavorite(radio)" :color="isFavorite(radio) ? 'red' : ''">
                      <v-icon v-if="isFavorite(radio)" :color="isPlaying(radio) ? 'white' : ''">mdi-heart</v-icon>
                      <v-icon v-else>mdi-heart-outline</v-icon>
                    </v-btn>

                    <!-- Player per i file M3U8 -->
                    <VideoPlayer type="default" @pause="processPause" :link="radio.url" :progress="30" :isMuted="false"
                      :isControls="true" class="customClassName" v-if="radio.hls == '1'" />
                  </div>
                </v-card-text>
              </v-col>
              <!-- Immagine della radio -->
              <v-col cols="4">
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
import { useDisplay } from 'vuetify';

export default {
  name: 'HomeView',
  data() {
    return {
      radios: [],
      filteredRadios: [],
      search: '',
      audio: null,
      sheet: false,
      selectedRadio: null,
      favorites: [],
      isRadioPlaying: false,
      videoOptions: {
        controls: true,
        autoplay: false,
        muted: false,
        europeanCountries: ['Italia', 'Francia', 'Germania', 'Spagna', 'Regno Unito', 'Portogallo', 'Olanda', 'Svezia', 'Danimarca', 'Norvegia', 'Finlandia', 'Belgio', 'Austria', 'Svizzera', 'Grecia', 'Polonia', 'Repubblica Ceca', 'Ungheria', 'Romania', 'Bulgaria', 'Croazia', 'Slovenia', 'Slovacchia', 'Estonia', 'Lettonia', 'Lituania', 'Irlanda', 'Islanda', 'Luxembourg', 'Malta', 'Cipro', 'Liechtenstein', 'Monaco', 'San Marino', 'Vaticano'],
        selectedCountry: ''
      },
      selectedTags: [],
      allTags: [],
    }
  },
  computed: {
    // Radio filtrate per tag selezionati
    filteredRadiosByTags() {
      if (this.selectedTags.length === 0) {
        return this.filteredRadios;
      }
      return this.filteredRadios.filter(radio => {
        const radioTags = radio.tags ? radio.tags.split(/[,\s]+/).map(tag => tag.trim()) : [];
        return this.selectedTags.every(tag => radioTags.includes(tag));
      });
    },
  },
  watch: {
  selectedTags: {
    handler: 'filterRadiosByTags',
    immediate: true // Questo attiverà immediatamente l'handler al montaggio del componente
  }
},

methods: {
  // Filtra le radio per tag selezionati
  filterRadiosByTags() {
    console.log('filterRadiosByTags chiamato');
    if (this.selectedTags.length === 0) {
      this.filteredRadios = this.radios;
      return;
    }
    this.filteredRadios = this.radios.filter(radio => {
      const radioTags = radio.tags ? radio.tags.split(/[,\s]+/).map(tag => tag.trim()) : [];
      return this.selectedTags.every(tag => radioTags.includes(tag));
    });
  },
    // Ottieni elenco di radio
    getRadios() {
      fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=100&countrycode=IT&hidebroken=true&order=clickcount&reverse=true')
        .then(response => response.json())
        .then(data => {
          this.radios = data;
          this.filteredRadios = data;
          this.createTags();
        });
    },
    // Crea elenco di tag da tutte le radio
    createTags() {
      this.allTags = this.radios.reduce((tags, radio) => {
        if (radio.tags) {
          tags.push(...radio.tags.split(/[,\s]+/).map(tag => tag.trim()));
        }
        return tags;
      }, []);
      this.allTags = [...new Set(this.allTags)].sort();
    },
    // Filtra le radio per paese selezionato
    filterRadiosByCountry() {
      if (this.selectedCountry) {
        this.filteredRadios = this.radios.filter(radio => radio.country.toLowerCase() === this.selectedCountry.toLowerCase());
      } else {
        this.filteredRadios = this.radios;
      }
    },
    // Ottieni URL favicon della radio
    getFaviconUrl(radio) {
      return radio.favicon || '/radio1.jpg';
    },
    // Avvia o metti in pausa la radio
    playOrPauseRadio(radio) {
      if (this.isPlaying(radio)) {
        this.stopRadio();
      } else {
        this.playRadio(radio);
      }
    },
    // Avvia la riproduzione della radio
    playRadio(radio) {
      this.stopRadio(); // Interrompi la radio attualmente in riproduzione
      this.sheet = true;
      this.selectedRadio = radio;
      this.isRadioPlaying = true; // Imposta lo stato di riproduzione su true

      // Se il formato è M3U8, utilizza il VideoPlayer
      if (radio.hls === 1) {
        console.log('Using vue-hls-video-player for M3U8 format');
        console.log(radio.url);
        this.audio = null; // Resetta l'elemento audio
      } else {
        console.log('Using <audio> element for MP3 format');
        this.audio = new Audio(radio.url_resolved); // Crea un nuovo elemento audio
        this.audio.play();
      }
    },
    // Interrompi la riproduzione della radio
    stopRadio() {
      if (this.audio instanceof Audio) {
        this.audio.pause();
        this.audio = null; // Resetta l'elemento audio
      }
      this.sheet = false;
      this.isRadioPlaying = false;
    },
    // Prima del smontaggio del componente, interrompi la radio
    beforeUnmount() {
      this.stopRadio();
    },
    // Filtra le radio per il testo di ricerca
    filterRadios() {
      if (!this.search) {
        this.filteredRadios = this.radios;
        return;
      }
      this.filteredRadios = this.radios.filter(radio => radio.name.toLowerCase().includes(this.search.toLowerCase()));
    },
    // Attiva o disattiva la riproduzione della radio
    togglePlayPause(radio) {
      if (this.isPlaying(radio)) {
        this.stopRadio();
      } else {
        this.playRadio(radio);
      }
    },
    // Aggiungi o rimuovi dalle radio preferite
    toggleFavorite(radio) {
      const index = this.favorites.findIndex(fav => fav.url === radio.url);
      if (index !== -1) {
        this.favorites.splice(index, 1);
      } else {
        this.favorites.push(radio);
      }
      localStorage.setItem('favorites', JSON.stringify(this.favorites));
    },
    // Verifica se una radio è tra le preferite
    isFavorite(radio) {
      return this.favorites.some(fav => fav.url === radio.url);
    },
    // Verifica se una radio è in riproduzione
    isPlaying(radio) {
      return this.selectedRadio === radio && this.audio && !this.audio.paused;
    },
    handleRouteChange() {
      this.stopRadio();
    },
  },
  created() {
    this.getRadios();
    const favorites = localStorage.getItem('favorites');
    this.favorites = favorites ? JSON.parse(favorites) : [];
    window.addEventListener('beforeunload', this.stopRadio);
    this.$router.beforeEach((to, from, next) => {
      this.handleRouteChange();
      next();
    });
  },
  // Prima del smontaggio del componente, rimuovi l'ascoltatore di evento
  beforeUnmount() {
    window.removeEventListener('beforeunload', this.stopRadio);
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
  background-color: rgb(0, 0, 0);
  position: relative;
}

.radio-card.playing {
  border: 2px solid red;
}

.radio-card v-img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.sound-wave {
  display: flex;
  align-items: center;
  height: 20px;
  margin-left: 10px;
  margin-top: 2px;
}

.bar {
  width: 4px;
  height: 100%;
  margin: 0 2px;
  background-color: red;
  animation: pulse 0.8s infinite ease-in-out alternate;
}

.bar:nth-child(1) {
  animation-delay: 0s;
}

.bar:nth-child(2) {
  animation-delay: 0.1s;
}

.bar:nth-child(3) {
  animation-delay: 0.2s;
}

.bar:nth-child(4) {
  animation-delay: 0.3s;
}

@keyframes pulse {
  0% {
    transform: scaleY(1);
  }

  100% {
    transform: scaleY(1.5);
  }
}
</style>
