# Weather App (Vue + Vite)

A fully functional weather web application built with Vue 3 and Vite.

## Features

- Search weather by city name
- Current weather details:
  - Temperature
  - Feels like
  - Weather condition + icon
  - Humidity
  - Wind speed
  - Pressure
- Error handling for invalid cities and network/API errors
- "Use Current Location" with browser geolocation
- Remembers last searched city with localStorage
- Responsive UI for mobile and desktop

## Tech Stack

- Vue 3
- Vite
- WeatherAPI (current weather endpoint)

## Setup and Run

1. Install dependencies:

```sh
npm install
```

2. Create your environment file:

```sh
copy .env.example .env
```

3. Open `.env` and add your real API key:

```env
VITE_WEATHERAPI_KEY=your_real_api_key
```

4. Start the app:

```sh
npm run dev
```

5. Open the local URL shown in terminal (usually `http://localhost:5173`).

## Build for Production

```sh
npm run build
```

Preview production build:

```sh
npm run preview
```

## Notes

- Do not commit your real `.env` file.
- Get a free API key from [WeatherAPI](https://www.weatherapi.com/).
