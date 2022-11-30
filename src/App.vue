<template>
  <div id="nav" class="noPrint">
    <router-link to="/">Verzehrzettel</router-link> |
    <router-link to="/about">Anleitung</router-link>
  </div>
  <router-view />
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  padding: 2rem;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}

@media print {
  .noPrint {
    display: none !important;
  }
  #app {
    padding: 0 2rem !important;
  }
}
</style>
