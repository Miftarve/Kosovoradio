<template>
  <div class="radio-wrapper">
    <v-container>
      <h1 style="color: white;">Preferiti🌟</h1>
      <br>
      <!-- Search bar -->
      <v-text-field v-model="search" label="Cerca" prepend-inner-icon="mdi-magnify" variant="outlined" hide-details
        single-line @input="filterFavorites"></v-text-field>

      <v-row>
        <v-col v-for="(favorite, index) in filteredFavorites" :key="index" cols="12" sm="6" md="4">
          <v-card class="radio-card" :class="{ 'active-radio': index === activeRadioIndex }">
            <v-row no-gutters>
              <v-col cols="8">
                <v-card-title>{{ favorite.name }}</v-card-title>
                <!-- Add other radio details if necessary -->
                <v-card-text>
                  <p>{{ favorite.tags }}</p>
                  <p>{{ favorite.country }}</p>
                  <div class="text-center align-end"
                    style="position: absolute; bottom: 0; width: 60%; margin-bottom: 30px;">
                    <!-- Button for favorites -->
                    <v-btn :style="{ marginRight: display && display.mdAndUp.value ? '10px' : '0' }" icon
                      @click="removeFromFavorites(index)" color="error">
                      <v-icon>mdi-delete</v-icon>
                    </v-btn>

                    <v-btn :style="{ margin: '0 10px' }" icon @click="togglePlayPause(favorite)"
                      :color="isPlaying(favorite) ? 'blue' : ''">
                      <v-icon v-if="isPlaying(favorite)">mdi-pause</v-icon>
                      <v-icon v-else>mdi-play</v-icon>
                    </v-btn>

                    <v-btn :style="{ marginLeft: display && display.mdAndUp.value ? '10px' : '0' }" icon
                      @click="stopRadio(favorite)" :color="isPlaying(favorite) ? 'blue' : ''">
                      <v-icon>mdi-stop</v-icon>
                    </v-btn>

                  </div>
                </v-card-text>
              </v-col>
              <v-col cols="4">
                <a :href="favorite.homepage" target="_blank">
                  <v-img :src="favorite.favicon ? getFaviconUrl(favorite) : '/image.png'" aspect-ratio="1/1" style="margin: 10px;"></v-img>
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
    activeRadioIndex: null // Aggiunta la proprietà activeRadioIndex
  }
},

  methods: {
    removeFromFavorites(index) {
      console.log('Rimozione dalla lista dei preferiti:', this.favorites[index].name);
      this.favorites.splice(index, 1);
      localStorage.setItem('favorites', JSON.stringify(this.favorites));
      this.filterFavorites(); // Update filtered favorites after removal
    },
    filterFavorites() {
      const searchText = this.search.toLowerCase();
      this.filteredFavorites = this.favorites.filter(favorite => {
        return favorite.name.toLowerCase().includes(searchText) ||
          favorite.tags.toLowerCase().includes(searchText) ||
          favorite.country.toLowerCase().includes(searchText);
      });
    },
    togglePlayPause(favorite, index) {
  if (this.isPlaying(favorite)) {
    this.stopRadio();
  } else {
    this.playRadio(favorite);
    this.activeRadioIndex = index; // Imposta l'indice della radio attiva
  }
},

    isPlaying(favorite) {
      return this.selectedRadio === favorite && this.audio && !this.audio.paused;
    },
    getFaviconUrl(favorite) {
      return favorite.favicon || '/default_favicon.png';
    },
    playRadio(favorite) {
      if (this.audio) {
        this.stopRadio();
      }
      this.audio = new Audio(favorite.url_resolved);
      this.audio.play();
      this.sheet = true;
      this.selectedRadio = favorite;
    },
    stopRadio() {
      if (this.audio) {
        this.audio.pause();
        this.audio = null;
        this.sheet = false;
        this.selectedRadio = null;
      }
    },
  },
  created() {
    const storedFavorites = JSON.parse(localStorage.getItem('favorites'));
    this.favorites = storedFavorites ? storedFavorites : [];
    this.filterFavorites(); // Filter favorites on page load
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

body{
  /*background-color: #1B3659;*/
  background-color: #000000;
}
.radio-card {
  height: 185px;
  /* Desired height for cards */
}

.radio-wrapper{
  background-color: rgb(0, 0, 0);
}

.active-radio {
  background-color: #F0F0F0; /* Imposta il colore di sfondo desiderato */
}

</style>
