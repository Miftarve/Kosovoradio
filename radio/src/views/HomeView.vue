<template>
  <div class="radio-wrapper">
    <v-container>
      <br>
      <!-- Search bar -->
      <v-row>
        <!-- Campo di ricerca -->
        <v-col cols="6">
          <v-text-field v-model="search" label="Cerca" prepend-inner-icon="mdi-magnify" variant="outlined" hide-details
            single-line dense @input="filterRadios"></v-text-field>
        </v-col>
        <!-- Combobox per i paesi -->
        <v-col cols="6">
          <v-combobox v-model="selectedCountry" :items="europeanCountries" label="Paese" dense outlined hide-details
            @change="filterRadiosByCountry" menu-props="{ openOnClick: true }"></v-combobox>
        </v-col>
      </v-row>
      <br><br>
      <v-row>
        <v-col v-for="(radio, index) in filteredRadios" :key="index" cols="12" sm="6" md="4">
          <v-card class="radio-card">
            <v-row no-gutters>
              <v-col cols="8">
                <v-card-title>{{ radio.name }}</v-card-title>
                <v-card-text>
                  <!-- Add other radio details if necessary -->
                  <p>{{ radio.tags }}</p>
                  <p>{{ radio.country }}</p>

                  <div class="text-center align-end"
                    style="position: absolute; bottom: 0; width: 60%; margin-bottom: 30px;">

                    <!--bottone play/pause-->
                    <v-btn class="mr-2" icon @click="togglePlayPause(radio)" :color="isPlaying(radio) ? 'green' : ''">
                      <v-icon v-if="isPlaying(radio)">mdi-pause</v-icon>
                      <v-icon v-else>mdi-play</v-icon>
                    </v-btn>

                    <!--Bottone preferiti-->
                    <v-btn class="mr-2" icon @click="toggleFavorite(radio)" :color="isFavorite(radio) ? 'red' : ''">
                      <v-icon v-if="isFavorite(radio)" :color="isPlaying(radio) ? 'white' : ''">mdi-heart</v-icon>
                      <v-icon v-else>mdi-heart-outline</v-icon>
                    </v-btn>

                    <!--PLAYER PER USARE I FILE M3U8-->
                    <VideoPlayer type="default" @pause="processPause" :link="radio.url" :progress="30" :isMuted="false"
                      :isControls="true" class="customClassName" v-if="radio.hls == '1'" />

                  </div>
                </v-card-text>
              </v-col>
              <v-col cols="4">
                <a :href="radio.homepage" target="_blank">
                  <v-img :src="getFaviconUrl(radio)" aspect-ratio="1/1" style="margin: 10px;"></v-img>
                </a>
              </v-col>
            </v-row>
          </v-card>
        </v-col>
      </v-row>

      <!-- Aggiungi il componente VideoPlayer per i file M3U8 -->


    </v-container>
  </div>
</template>

<script>
import { useDisplay } from 'vuetify';
//import VideoPlayer from 'vue-hls-video-player';

export default {
  name: 'HomeView',
  components: {
    //VideoPlayer
  },
  data() {
    return {
      radios: [],
      filteredRadios: [],
      search: '',
      audio: null,
      sheet: false,
      selectedRadio: null,
      favorites: [], // Aggiunto per gestire i preferiti
      videoOptions: {
        controls: true,
        autoplay: false,
        muted: false,
        europeanCountries: ['Italia', 'Francia', 'Germania', 'Spagna', 'Regno Unito', 'Portogallo', 'Olanda', 'Svezia', 'Danimarca', 'Norvegia', 'Finlandia', 'Belgio', 'Austria', 'Svizzera', 'Grecia', 'Polonia', 'Repubblica Ceca', 'Ungheria', 'Romania', 'Bulgaria', 'Croazia', 'Slovenia', 'Slovacchia', 'Estonia', 'Lettonia', 'Lituania', 'Irlanda', 'Islanda', 'Luxembourg', 'Malta', 'Cipro', 'Liechtenstein', 'Monaco', 'San Marino', 'Vaticano']
      },
    }
  },
  methods: {
    getRadios() {
      fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=100&countrycode=IT&hidebroken=true&order=clickcount&reverse=true')
        .then(response => response.json())
        .then(data => {
          this.radios = data;
          this.filteredRadios = data;
        });
    },
    getFaviconUrl(radio) {
      return radio.favicon || '/si.png';
    },
    playOrPauseRadio(radio) {
      if (this.isPlaying(radio)) {
        this.stopRadio();
      } else {
        this.playRadio(radio);
      }
    },
    filterRadiosByCountry() {
      if (this.selectedCountry) {
        this.filteredRadios = this.radios.filter(radio => radio.country.toLowerCase() === this.selectedCountry.toLowerCase());
      } else {
        this.filteredRadios = this.radios;
      }
    },
    playRadio(radio) {
      this.stopRadio(); // Interrompi la radio attualmente in riproduzione
      this.sheet = true;
      this.selectedRadio = radio;

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
    stopRadio() {
      if (this.audio instanceof Audio) {
        this.audio.pause();
        this.audio = null; // Resetta l'elemento audio
      }
      this.sheet = false;
    },
    filterRadios() {
      if (!this.search) {
        this.filteredRadios = this.radios;
        return;
      }
      this.filteredRadios = this.radios.filter(radio => radio.name.toLowerCase().includes(this.search.toLowerCase()));
    },
    togglePlayPause(radio) {
      if (this.isPlaying(radio)) {
        this.stopRadio();
      } else {
        this.playRadio(radio);
      }
    },
    toggleFavorite(radio) {
      const index = this.favorites.findIndex(fav => fav.url === radio.url);
      if (index !== -1) {
        this.favorites.splice(index, 1);
      } else {
        this.favorites.push(radio);
      }
      localStorage.setItem('favorites', JSON.stringify(this.favorites));
    },
    isFavorite(radio) {
      return this.favorites.some(fav => fav.url === radio.url);
    },
    isPlaying(radio) {
      return this.selectedRadio === radio && this.audio && !this.audio.paused;
    },
  },
  created() {
    this.getRadios();
    const favorites = localStorage.getItem('favorites');
    this.favorites = favorites ? JSON.parse(favorites) : [];
    window.addEventListener('beforeunload', this.stopRadio); // Stoppa la radio prima di passare ad un'altra pagina
  },
  beforeUnmount() {
    window.removeEventListener('beforeunload', this.stopRadio); // Rimuove l'event listener prima di smontare il componente
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
  /* Altezza originale */
  background-color: rgb(0, 0, 0);
  position: relative;
  /* Imposta la posizione relativa per consentire il posizionamento dell'immagine */
}

.radio-card v-img {
  position: absolute;
  /* Imposta la posizione assoluta per consentire il posizionamento dell'immagine */
  top: 0;
  /* Posiziona l'immagine all'inizio della card */
  left: 0;
  /* Posiziona l'immagine all'inizio della card */
  width: 100%;
  /* Occupa tutta la larghezza della card */
  height: 100%;
  /* Occupa tutta l'altezza della card */
  object-fit: cover;
  /* Scala l'immagine per coprire completamente il contenitore mantenendo l'aspect ratio */
}
</style>
