<template>
  <!-- Intestazione della navigazione -->
  <nav>
    <!-- Titolo del sito -->
    <h1 style="color: white; font-size: 3.5em;">𝓡𝓐𝓓𝓘𝓞 𝓥𝓔𝓛𝓘𝓠𝓘</h1>
    <!-- Pulsanti di navigazione -->
    <div class="nav-buttons">
      <!-- Pulsante per la pagina Home -->
      <div class="nav-item">
        <router-link to="/">𝗛𝗼𝗺𝗲</router-link>
      </div>
      <!-- Pulsante per la pagina About -->
      <div class="nav-item">
        <router-link to="/about">𝗣𝗿𝗲𝗳𝗲𝗿𝗶𝘁𝗶</router-link>
      </div>
      <!-- Pulsante per la pagina World -->
      <div class="nav-item">
        <router-link to="/World">𝗠𝗮𝗽𝗽𝗮</router-link>
      </div>
    </div>
  </nav>
  <!-- Componente per visualizzare i contenuti delle varie pagine -->
  <router-view/>
</template>

<style>
/* Stili per l'applicazione */
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: black;
  background-color: rgb(0, 0, 0); /* Colore di sfondo */
}

/* Stili per la barra di navigazione */
nav {
  padding: 30px;
  background-color: rgb(0, 0, 0); /* Colore di sfondo */
}

/* Stili per il titolo del sito */
h1 {
  margin-bottom: 20px;
}

/* Stili per i pulsanti di navigazione */
.nav-buttons {
  display: flex;
  justify-content: center;
}

/* Stili per singolo pulsante di navigazione */
.nav-item {
  margin-right: 20px;
}

/* Stili per il testo dei pulsanti di navigazione */
.nav-item a {
  display: inline-block;
  padding: 10px 20px;
  border-radius: 10px;
  background-color: #ffffff; /* Colore di sfondo */
  color: black;
  text-decoration: none;
  font-weight: bold;
}

/* Stili per il cambio di colore al passaggio del mouse sui pulsanti di navigazione */
.nav-item a:hover {
  background-color: #9d9d9d;
}

/* Stili per il pulsante attivo */
.nav-item a.router-link-exact-active {
  background-color: #9d9d9d;
}
</style>
