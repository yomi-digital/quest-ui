<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">YOMI QUEST</router-link><!-- |
      <router-link to="/about">About</router-link> |
      <router-link to="/leaderboard">Leaderboard</router-link>-->
    </div>
    <router-view style="margin-top:40px" />
    <div class="collection external terminal">
      Made with ❤️ at <a href="https://yomi.digital">YOMI</a>
    </div>
  </div>
</template>

<style>
#app {
  font-family: var(--font, monospace);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
#nav {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 999;
  background: #000;
  width: 100%;
  padding: 10px;
  text-align: left;
  text-transform: uppercase;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
  font-size: 15px;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
