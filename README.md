# ☀️ Clima Agora

![Vue.js](https://img.shields.io/badge/Vue.js-3.x-4FC08D?logo=vue.js)
![Open-Meteo](https://img.shields.io/badge/API-Open--Meteo-00A8E8)
![Leaflet](https://img.shields.io/badge/Map-Leaflet-199900)
![Chart.js](https://img.shields.io/badge/Chart.js-3.x-FF6384)

Aplicação web moderna de previsão do tempo que combina dados meteorológicos em tempo real com efeitos visuais dinâmicos, mapa interativo e gráficos detalhados. Desenvolvida com **Vue 3 Composition API**, a interface se adapta visualmente às condições climáticas e ao período do dia (dia/noite).

## Funcionalidades

- **Busca por localização** – Usar geolocalização do navegador ou pesquisar manualmente por cidade, estado e país.
- **Clima atual** – Temperatura, máxima/mínima, velocidade do vento e descrição textual com ícone.
- **Previsão horária** – Carrossel com as próximas 24 horas, mostrando horário, ícone (com variação dia/noite) e temperatura.
- **Previsão semanal** – Gráfico interativo (máximas, mínimas, chance de chuva e vento) usando Chart.js. Ao clicar em um dia, o carrossel horário filtra as horas daquele dia.
- **Mapa interativo** – Exibe a localização da cidade com um marcador e o contorno da região (boundary) obtido via Nominatim/OSM.
- **Fundo dinâmico** – Gradientes que mudam de acordo com o clima (céu limpo, nublado, chuva, neve, tempestade) e a hora do dia (dia/noite).
- **Partículas meteorológicas** – Animação de chuva ou neve sobre o fundo quando aplicável.
- **Efeito de raios** – Flash aleatório em dias de tempestade.
- **Design responsivo** – Adaptação para dispositivos móveis (esconde gráfico semanal em telas pequenas).

## Tecnologias utilizadas

- **Vue 3** (Composition API, `<script setup>`)
- **Vite** (como bundler)
- **Chart.js** com plugin `chartjs-plugin-datalabels`
- **Leaflet** (mapas) + tiles CartoDB
- **Open-Meteo API** (dados meteorológicos)
- **Nominatim / OpenStreetMap** (geocodificação e boundaries)
- **HTML5 Canvas** (animação de partículas)
- **CSS3** (Glassmorphism, Gradientes, Transições)

## Instalação e execução

### Pré-requisitos

- Node.js (v16 ou superior)
- npm ou yarn

### Passos

```bash
# Clone o repositório
git clone https://github.com/seu-usuario/clima-agora.git
cd clima-agora

# Instale as dependências
npm install

# Execute em modo de desenvolvimento
npm run dev

```

A aplicação estará disponível em http://localhost:5173 (ou a porta que o Vite indicar).

## Build para produção

```bash
npm run build
```

## 🗺️ APIs externas utilizadas

| Serviço | Finalidade |
| --------| --------------- |
| **Open-meteo** | Dados de previsão (atual, horário e diária) |
| **Nominatim (OSM)** | Geocodificação reversa e busca de cidades (+ obtenção do polígono da cidade) |

Nenhuma chave de API é necessária – ambos os serviços são abertos e gratuitos.

## 🧩 Estrutura dos componentes principais

| Componentes | Responsabilidades |
| ----------- | ----------------- |
| App.vue | Orquestra os componentes, gerencia weatherData, estado dia/noite e gradientes do título. |
| SearchSection.vue | Inputs de busca (geolocalização ou cidade/estado/país) e emissão dos dados climáticos. |
| WeatherCard.vue | Exibe resumo do clima atual (temperatura, vento, ícone, máx/mín). |
| HourlyCarousel.vue | Carrossel com previsão por hora, suporta filtro por data clicada no gráfico. |
| WeeklyChart.vue | Gráfico semanal (linhas + barras) usando Chart.js. Emite evento day-selected. |
| Map.vue | Mapa Leaflet com marcador e boundary (polígono) da cidade. |
| BackgroundEffect.vue | Fundo com gradiente animado (crossfade), partículas (rain/snow) e flashes de raios. |

## 🎨 Personalização do tema visual

- Os gradientes de fundo são definidos no componente BackgroundEffect.vue – há classes CSS para cada condição climática tanto para modo diurno quanto noturno (ex: .bg-rain-night).

- O título da aplicação também muda de gradiente de acordo com o clima, controlado por App.vue.

## 📱 Responsividade

- O gráfico semanal é oculto em telas com largura ≤ 767px (display: none).
- O mapa tem altura reduzida (250px) em mobile.
- O carrossel horário permite rolagem horizontal com botões ou touch.

## 🧪 Possíveis melhorias futuras

- Adicionar suporte a múltiplos idiomas (i18n)
- Salvar cidade favorita no localStorage
- Exibir nascer/pôr do sol no gráfico ou carrossel
- Animações mais ricas (neve com acúmulo, raios com som)
- Testes unitários (Vitest)

## 📄 Licença

Este projeto é distribuído sob a licença MIT. Sinta-se livre para usar, modificar e distribuir.

## 👤 Autor

Desenvolvido por [Israel Henrique](https://github.com/isrique9) – [henriqueisraeul790@gmail.com](mailto:henriqueisraeul790@gmail.com)  
Projeto de portfólio para demonstração de Vue.js e integração com APIs meteorológicas.


**Nota:** Para o funcionamento correto dos marcadores do Leaflet, é necessário configurar os URLs dos ícones no início do componente Map.vue (já está feito no código fornecido). Certifique-se de que os assets Leaflet estão disponíveis na build.

