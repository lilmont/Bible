
# Adapter Design Pattern in Angular

The **Adapter Design Pattern** is a structural pattern used to allow incompatible interfaces to work together. In Angular, this is particularly useful when integrating third-party services, APIs, or adapting legacy code to fit into a modern Angular application.

In an Angular app, an adapter can help wrap an external service or convert data formats so they match the app's expected interfaces.

## Key Concepts

- **Target**: The interface or expected format that your Angular component or service wants to work with.
- **Adaptee**: The existing service, API, or class that does not match the target interface.
- **Adapter**: A service or class that translates data or methods from the Adaptee to match the Target.

## Example: Adapting a Weather API

Suppose you have an Angular component that displays weather data. Your component expects data in a specific format, but the weather API returns it differently. The adapter will transform the API’s response to fit the expected format.

### Step 1: Define Expected Weather Data Interface (`WeatherData`)

```typescript
// weather-data.model.ts
export interface WeatherData {
  temperature: number;
  humidity: number;
  windSpeed: number;
  description: string;
}
```

### Step 2: Create the Adaptee (3rd Party Weather Service)

Imagine this service is from a third-party library and cannot be modified.

```typescript
// third-party-weather.service.ts
export interface ThirdPartyWeatherResponse {
  temp: number;
  humid: number;
  wind_spd: number;
  weather_desc: string;
}

export class ThirdPartyWeatherService {
  getWeather(city: string): Promise<ThirdPartyWeatherResponse> {
    // Imagine this makes an HTTP request and returns a promise
    return Promise.resolve({
      temp: 20,
      humid: 65,
      wind_spd: 10,
      weather_desc: 'Sunny'
    });
  }
}
```

The response fields (`temp`, `humid`, `wind_spd`, and `weather_desc`) do not match what our app expects (`temperature`, `humidity`, `windSpeed`, and `description`).

### Step 3: Create the Adapter Service

To make the `ThirdPartyWeatherService` compatible with our `WeatherData` interface, we create an adapter.

```typescript
// weather-adapter.service.ts
import { Injectable } from '@angular/core';
import { WeatherData } from './weather-data.model';
import { ThirdPartyWeatherService } from './third-party-weather.service';

@Injectable({
  providedIn: 'root'
})
export class WeatherAdapterService {
  constructor(private thirdPartyService: ThirdPartyWeatherService) {}

  async getWeather(city: string): Promise<WeatherData> {
    const response = await this.thirdPartyService.getWeather(city);
    return {
      temperature: response.temp,
      humidity: response.humid,
      windSpeed: response.wind_spd,
      description: response.weather_desc
    };
  }
}
```

Here, the adapter (`WeatherAdapterService`) converts the `ThirdPartyWeatherService` data to match the `WeatherData` interface format.

### Step 4: Using the Adapter in a Component

Now, the `WeatherAdapterService` can be used in an Angular component to get the weather data in the desired format.

```typescript
// weather.component.ts
import { Component, OnInit } from '@angular/core';
import { WeatherAdapterService } from './weather-adapter.service';
import { WeatherData } from './weather-data.model';

@Component({
  selector: 'app-weather',
  template: `
    <div *ngIf="weather">
      <h3>Weather Information</h3>
      <p>Temperature: {{ weather.temperature }}°C</p>
      <p>Humidity: {{ weather.humidity }}%</p>
      <p>Wind Speed: {{ weather.windSpeed }} km/h</p>
      <p>Description: {{ weather.description }}</p>
    </div>
  `
})
export class WeatherComponent implements OnInit {
  weather: WeatherData | null = null;

  constructor(private weatherAdapterService: WeatherAdapterService) {}

  ngOnInit(): void {
    this.weatherAdapterService.getWeather('New York').then((data) => {
      this.weather = data;
    });
  }
}
```

### Step 5: Results

When you run the component, it will display the weather information in the format expected by your app:

```
Weather Information
Temperature: 20°C
Humidity: 65%
Wind Speed: 10 km/h
Description: Sunny
```

## Summary

The **Adapter Design Pattern** in Angular is an effective way to transform incompatible data or services so they align with your app's requirements. By creating a custom adapter, we can use third-party services or data sources without modifying the original code.

---
