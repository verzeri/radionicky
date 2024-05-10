<template>
  <nav>
    <router-link to="/" class="nav-link">Home</router-link>  /               
    <router-link to="about" class="nav-link">Preferiti</router-link>  /     
    <router-link to="world" class="nav-link">Mappa</router-link>  /
  </nav>
  <router-view/>
</template>

<style>
#app {
  font-family: 'Arial';
  text-align: center;
  color: black;
  font-size: 80px;
}

nav {
  padding: 30px;
}

nav .nav-link {
  font-weight: bold;
  font-family: 'Arial';
  color: black;
  font-size: 80px;
  padding: 0 10px;
  text-decoration: none; /* Rimuove la sottolineatura */
}


</style>
