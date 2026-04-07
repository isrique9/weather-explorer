<template>
  <div class="search-box">
    <h2>📍 Localização</h2>
    <button class="location-btn">Usar minha localização</button>
    <div class="manual-search">
      <input type="text" placeholder="Digite o nome da cidade..." />
      <button>Buscar</button>
    </div>
  </div>
</template>

<style scoped>
.search-box {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.location-btn {
  background: #2c3e50;
  color: white;
  border: none;
  padding: 0.75rem;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  transition: background 0.2s;
}
.location-btn:hover {
  background: #1a2a3a;
}
.manual-search {
  display: flex;
  gap: 0.5rem;
}
.manual-search input {
  flex: 1;
  padding: 0.75rem;
  border-radius: 8px;
  border: 1px solid #ccc;
  font-size: 1rem;
}
.manual-search button {
  background: #3498db;
  color: white;
  border: none;
  padding: 0 1.2rem;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.2s;
}
.manual-search button:hover {
  background: #077cca;
}
</style>