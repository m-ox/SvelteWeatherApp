<script>
  let weatherStatus = "clear";
  const WEATHER_API_KEY = import.meta.env.VITE_WEATHER_API_KEY;
  let lon = '';
  let lat = '';
  let weather;

  // promises
  async function fetchLocation() {
    const successCallback = (position) => {
      lon = position.coords.longitude;
      lat = position.coords.latitude;
    };
    const errorCallback = (error) => {
      console.error(error);
    };
    navigator.geolocation.getCurrentPosition(successCallback, errorCallback)
  }
  async function fetchWeather(lat, lon) {
    if (lat && lon) {
      const weatherPromise = await fetch(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${WEATHER_API_KEY}`)
      let weatherRes = await weatherPromise.json();
      weatherStatus = weatherRes.weather[0].main.toLowerCase() || 'cloudy';
      return weatherRes;
    }
  }

  // converters
  function convertToCelsius(k) {
    return Math.floor(weather.main.temp - 273.15)
  }
  function convertToFahrenheit(k) {
    return Math.floor((k - 273.15) * 9/5 + 32)
  }

  // stylish little button shit
  // it's horrible
  // YOU try to code with someone playing a miniature hangdrum in the same room, bud
  let measurementType = 'F';
  let celsiusColor = 'red';
  let fahrenColor = 'green';
  function switchMeasurement() {
    if (measurementType === 'F') {
      measurementType = 'C'
      celsiusColor = 'green';
      fahrenColor = 'red';
    } else if (measurementType === 'C') {
      measurementType = 'F'
      celsiusColor = 'red';
      fahrenColor = 'green';
    }
  }
  function calcTemp(temp) {
    if (measurementType === 'F') {
      return convertToFahrenheit(temp)
    } else if (measurementType === 'C') {
      return convertToCelsius(temp)
    } else {
      return 'eat shit'
    }
  }
</script>

<main class={`${weatherStatus}-bg`}>
  <h1>Hello. The main weather today is <span class="weather-status--{weatherStatus}">{weatherStatus}</span>!</h1>

  <div class="card">
    {#await fetchLocation()}
      <div class="weather-container">Loading...</div>
    {:then}
    <div class="weather-container">
      {#await fetchWeather(lat, lon)}
        Fetching weather from coordinates {lon}, {lat}...
      {:then weather}
        <div class="weather-tile">
          <h1>Location</h1>
          <p>{weather?.name}</p>
        </div>
        <div class="weather-tile">
          <h1>Humidity Rating</h1>
          <p>{weather?.main.humidity}</p>
        </div>
        <div class="weather-tile">
          <div class="weather-tile-header">
            <h1>Temperature</h1>
            <button on:click={switchMeasurement}>
              <span style="color: {celsiusColor}">C</span>/<span style="color: {fahrenColor}">F</span>
            </button>
          </div>
          <p>{calcTemp(weather?.main.temp)} {measurementType}°</p>
        </div>
        <div class="weather-tile">
          <h1>Wind Speed</h1>
          <p>{weather?.wind.speed}</p>
        </div>
      {/await}
    </div>
    {/await}
  </div>

  <p>
    Provided coordinates: {lon} and {lat}
  </p>
  
  <p>
    For more information on weather, please visit the <a href="https://www.weather.gov/" target="_blank" rel="noreferrer">National Weather Service</a> site.
  </p>

  <p class="read-the-docs">
    This app was created with Vite and Svelte.
  </p>
</main>

<style>
  main {
    height: -webkit-fill-available;
    width: 100%;
    margin: 0;
    padding: 1rem 0;
    background-position-y: center;
    display: flex;
    flex-direction: column;
    place-content: center;
  }

  .card {
    display: flex;
    place-content: center;
  }

  .clear-bg {
    background-image: url(./assets/backgrounds/sunny.jpg);
  }

  .clouds-bg {
    background-image: url(./assets/backgrounds/cloudy.jpg);
    mix-blend-mode: multiply;
    color: whitesmoke !important;
  }

  .rain-bg {
    background-image: url(./assets/backgrounds/rainy.jpg);
  }

  .snow-bg {
    background-image: url(./assets/backgrounds/snow.jpg);
  }

  .drizzle-bg {
    background-image: url(./assets/backgrounds/drizzle.jpg);
  }

  .thunderstorm-bg {
    background-image: url(./assets/backgrounds/thunderstorm.jpg);
  }

  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #343effaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #dfdfdf;
  }

  .weather-status--cloudy {
    color: lightblue;
  }

  .weather-status--clear {
    color: orange;
  }
  
  button {
    height: auto;
    width: auto;
    font-size: small;
    margin: 0.3rem;
    padding: 2px;
    border-radius: 1rem;
  }

  h1 {
    font-size: 1rem;
    padding: .5rem 0;
    margin: 0 1rem;
  }

  p {
    margin: 0;
    padding: 0;
  }

  .weather-container {
    background-color: rgba(234, 234, 234, 0.5);
    border-radius: 1rem;
    height: fit-content;
    padding: .5rem;
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: .75rem;
  }

  .weather-tile {
    max-height: 75px;
    border: 1px solid white;
    border-radius: 1rem;
    margin: 0.2rem;
    white-space: no-wrap;
  }

  .weather-tile-header {
    display: flex;
  }
</style>
