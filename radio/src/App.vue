<template>
  <nav>
    <h1 style="color: white; font-size: 3.5em;">𝓡𝓐𝓓𝓘𝓞 𝓥𝓔𝓛𝓘𝓠𝓘</h1>
    <div class="nav-buttons">
      <div class="nav-item">
        <router-link to="/">𝗛𝗼𝗺𝗲</router-link>
      </div>
      <div class="nav-item">
        <router-link to="/about">𝗣𝗿𝗲𝗳𝗲𝗿𝗶𝘁𝗶</router-link>
      </div>
      <div class="nav-item">
        <router-link to="/World">𝗠𝗮𝗽𝗽𝗮</router-link>
      </div>
    </div>
  </nav>
  <router-view/>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: black;
  background-color: rgb(0, 0, 0); 
}

nav {
  padding: 30px;
  background-color: rgb(0, 0, 0); 
}

h1 {
  margin-bottom: 20px;
}

.nav-buttons {
  display: flex;
  justify-content: center;
}

.nav-item {
  margin-right: 20px;
}

.nav-item a {
  display: inline-block;
  padding: 10px 20px;
  border-radius: 10px;
  background-color: #ffffff;
  color: black;
  text-decoration: none;
  font-weight: bold;
}

.nav-item a:hover {
  background-color: #9d9d9d;
}

.nav-item a.router-link-exact-active {
  background-color: #9d9d9d;
}
</style>