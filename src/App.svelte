<script>
  import { fade } from "svelte/transition";
  import { tick } from "svelte";

  let city = "";
  let weatherData = null;
  let currentIndex = 0;
  let errorMessage = null;
  let loading = false;
  let intervalId = null;

  async function fetchWeather() {
    if (loading) return;
    loading = true;
    errorMessage = null;

    const apiKey = "18337e34d4e093fb8e81ae9c1f8af5e0";
    const response = await fetch(
      `https://api.openweathermap.org/data/2.5/forecast?q=${city}&units=metric&appid=${apiKey}`
    );

    if (!response.ok) {
      errorMessage = "City does not exist or no data available";
      weatherData = null;
      loading = false;
      return;
    }

    const data = await response.json();

    if (!data.list || data.list.length === 0) {
      errorMessage = "City does not exist or no data available";
      weatherData = null;
      loading = false;
      return;
    }

    const dailyData = [];
    const seenDates = new Set();

    for (let i = data.list.length - 1; i >= 0; i--) {
      const entry = data.list[i];
      const date = new Date(entry.dt * 1000).toLocaleDateString();

      if (!seenDates.has(date)) {
        seenDates.add(date);
        dailyData.unshift(entry);
      }
    }

    await tick();
    weatherData = { ...data, list: dailyData };
    loading = false;
    currentIndex = 0;

    if (intervalId) {
      clearInterval(intervalId);
    }

    startIndexUpdate();
  }

  function startIndexUpdate() {
    intervalId = setInterval(() => {
      if (weatherData && weatherData.list.length > 0) {
        currentIndex = (currentIndex + 1) % weatherData.list.length;
      }
    }, 5000);
  }

  function handleKeyup(event) {
    if (event.key === "Enter") {
      fetchWeather();
    }
  }

  function capitalizeDescription(description) {
    return description
      .split(" ")
      .map((word) => word.charAt(0).toUpperCase() + word.slice(1))
      .join(" ");
  }

  const gifMapping = {
    "01d": "https://media.giphy.com/media/VHf2YBdIm7GsyRzwVZ/giphy.gif",
    "01n": "https://media.giphy.com/media/VHf2YBdIm7GsyRzwVZ/giphy.gif",
    "02d": "https://media.giphy.com/media/3ohs4uJC1G9NNq03fi/giphy.gif",
    "02n": "https://media.giphy.com/media/3ohs4uJC1G9NNq03fi/giphy.gif",
    "03d": "https://media.giphy.com/media/0Styincf6K2tvfjb5Q/giphy.gif",
    "03n": "https://media.giphy.com/media/0Styincf6K2tvfjb5Q/giphy.gif",
    "04d": "https://media.giphy.com/media/GFXNdR1tuMopi/giphy.gif",
    "04n": "https://media.giphy.com/media/GFXNdR1tuMopi/giphy.gif",
    "09d": "https://media.giphy.com/media/26DMWExfbZSiV0Btm/giphy.gif",
    "09n": "https://media.giphy.com/media/26DMWExfbZSiV0Btm/giphy.gif",
    "10d": "https://media.giphy.com/media/26DMWExfbZSiV0Btm/giphy.gif",
    "10n": "https://media.giphy.com/media/26DMWExfbZSiV0Btm/giphy.gif",
    "11d": "https://media.giphy.com/media/13ZEwDgIZtK1y/giphy.gif",
    "11n": "https://media.giphy.com/media/13ZEwDgIZtK1y/giphy.gif",
    "13d": "https://media.giphy.com/media/KFUx0Rtz7p0HTzbJ7x/giphy.gif",
    "13n": "https://media.giphy.com/media/KFUx0Rtz7p0HTzbJ7x/giphy.gif",
    "50d": "https://media.giphy.com/media/McDhCoTyRyLiE/giphy.gif",
    "50n": "https://media.giphy.com/media/McDhCoTyRyLiE/giphy.gif",
  };

  function getWeatherGif(icon) {
    const gifUrl = gifMapping[icon];
    return gifUrl || "";
  }

  $: gifUrl = getWeatherGif(weatherData?.list[currentIndex]?.weather[0]?.icon);
</script>

<main style="background-image: url('{gifUrl}');">
  <h1 class="app-title">WEATHER INFORMATION NEWS</h1>
  <div class="search-bar">
    <input
      type="text"
      bind:value={city}
      placeholder="Enter City Name"
      on:keyup={handleKeyup}
    />
    <button on:click={fetchWeather}>Search</button>
  </div>

  {#if errorMessage}
    <p class="error-message">{errorMessage}</p>
  {/if}

  {#if weatherData && !loading}
    <div class="weather-card" transition:fade>
      <h2>{weatherData.city.name}</h2>
      <p class="date">
        {new Date(
          weatherData.list[currentIndex].dt * 1000
        ).toLocaleDateString()}
      </p>
      <p class="temp">
        {Math.round(weatherData.list[currentIndex].main.temp)} °
      </p>
      <p class="description">
        {capitalizeDescription(
          weatherData.list[currentIndex].weather[0].description
        )}
      </p>
      <p class="min-max-temp">
        Min: {Math.round(weatherData.list[currentIndex].main.temp_min)} °C | Max:
        {Math.round(weatherData.list[currentIndex].main.temp_max)} °C
      </p>
      <p class="feels-like">
        Feels Like: {Math.round(weatherData.list[currentIndex].main.feels_like)}
        °C
      </p>
      <p class="pressure">
        Pressure: {weatherData.list[currentIndex].main.pressure} hPa
      </p>
      <p class="wind-speed">
        Wind Speed: {weatherData.list[currentIndex].wind.speed} m/s
      </p>
    </div>
  {/if}

  {#if loading}
    <p class="loading-message">Loading...</p>
  {/if}
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    height: 100vh;
    font-family: "Univia Pro", "Regular", Arial, Helvetica, sans-serif;
    color: #04bdc6;
    padding: 20px;
    background-size: auto 100%;
    background-position: center;
    background-repeat: no-repeat;
  }

  .app-title {
    font-size: 9vw;
    font-weight: bold;
    color: #04bdc6;
    margin-top: 50px;
    margin-bottom: 20px;
    text-align: center;
    max-width: 100%;
    padding: 0 10px;
  }

  .search-bar {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 3vh;
    width: 80%;
  }

  .search-bar input {
    padding: 10px;
    font-size: 1.5em;
    border: 2px solid #04bdc6;
    border-radius: 10px;
    width: 70%;
    background-color: #f9f9f9;
  }

  .search-bar button {
    padding: 10px;
    font-size: 1.5em;
    background-color: #04bdc6;
    color: white;
    border-radius: 10px;
    cursor: pointer;
    border: 2px solid #04bdc6;
    width: 25%;
  }

  .search-bar button:hover {
    background-color: #0c1b2b;
    color: #04bdc6;
    border: 2px solid #04bdc6;
  }

  .weather-card {
    background: #0c1b2b;
    border: 2px solid #04bdc6;
    border-radius: 15px;
    padding: 20px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
    width: 75%;
    height: auto; /* Höhe an den Inhalt anpassen */
    text-align: center;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  .weather-card h2 {
    font-size: 6em;
    color: #04bdc6;
    margin-bottom: -50px; /* Weniger Abstand zum Datum */
    margin-top: -5px;
  }

  .weather-card .date {
    font-size: 3em;
    margin-bottom: -200px; /* Weniger Abstand zur Temperatur */
  }

  .weather-card .temp {
    font-size: 12em;
    margin-bottom: -60px; /* Weniger Abstand zur Beschreibung */
  }

  .weather-card .description {
    font-size: 3em;
    margin-bottom: 80px; /* Erhöhe den Abstand nach unten */
    margin-top: 40px;
  }

  .weather-card p {
    font-weight: bold;
    color: #fff;
    font-size: 3em;
    margin-bottom: -20px;
  }

  .weather-card .wind-speed {
    margin-bottom: 100px;
  }

  .error-message {
    color: red;
    font-size: 3em;
    font-weight: bold;
    text-align: center;
    margin-top: 20px;
  }
</style>
