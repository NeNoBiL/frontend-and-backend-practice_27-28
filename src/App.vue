<template>
  <div id="app">
    <header class="app-header">
      <h1>🎨 Генератор цветовых палитр</h1>
      <p>Vue 3 + Composition API • Практика 27-28</p>
    </header>

    <main class="main-content">
      <ColorPalette />
    </main>

    <footer class="app-footer">
      <p>Разработано с использованием Vue 3 • Контрольная работа №5</p>
    </footer>
  </div>
</template>

<script>
import ColorPalette from './components/ColorPalette.vue'

export default {
  name: 'App',
  components: {
    ColorPalette
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f5f5f5;
  color: #333;
  line-height: 1.6;
}

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 2rem;
  text-align: center;
}

.app-header h1 {
  margin-bottom: 0.5rem;
  font-size: 2.5rem;
}

.app-header p {
  opacity: 0.9;
  font-size: 1.1rem;
}

.main-content {
  flex: 1;
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

.app-footer {
  background-color: #333;
  color: white;
  text-align: center;
  padding: 1rem;
  margin-top: auto;
}
</style>