This is a basic application that shows the weather information that fetches the data from an open API. (RapidAPI)

Firstly we created an application interface.

```tsx
<div class="container" *ngIf="weatherData">
  <div class="upper-data">
    <img
      src="../assets/sunnyweather.png"
      alt="Sunny Weather"
      *ngIf="weatherData.main.temp >= 15"
    />
    <img
      src="../assets/coldweather.png"
      alt="Cold Weather"
      *ngIf="weatherData.main.temp < 15"
    />
    <div class="weather-data">
      <div class="location">{{ weatherData.name }}</div>
      <div class="temperature">
        {{ weatherData.main.temp | number: "1.0-0" }} &deg;C
      </div>
    </div>
  </div>
  <div class="lower-data">
    <div class="more-info-label">More Information</div>
    <div class="more-info-container">
      <div class="info-block">
        <div class="info-block-label">
          <img src="../assets/coldtemperature.png" />
          <span>min</span>
        </div>
        <div class="info-block-value">
          {{ weatherData.main.temp_min | number: "1.0-0" }}°C
        </div>
      </div>

      <div class="info-block">
        <div class="info-block-label">
          <img src="../assets/overheat.png" />
          <span>max</span>
        </div>
        <div class="info-block-value">
          {{ weatherData.main.temp_max | number: "1.0-0" }}°C
        </div>
      </div>

      <div class="info-block">
        <div class="info-block-label">
          <img src="../assets/humidity.png" />
          <span>humidity</span>
        </div>
        <div class="info-block-value">{{ weatherData.main.humidity }}%</div>
      </div>

      <div class="info-block">
        <div class="info-block-label">
          <img src="../assets/wind.png" />
          <span>wind</span>
        </div>
        <div class="info-block-value">{{ weatherData.wind.speed }} km/h</div>
      </div>
    </div>
  </div>
</div>
```

This code is a classic HTML code. We see the pipes unlike what we saw in the previous project. The purpose of pipe in this code is to convert fractional numbers to integers. And then, we add some css codes to style.css. I don’t show css codes in here because it’s basic css.

And then we add http modules to the app.module.ts file. This is required for fetching data to api. When we fetch the data, we have to use HttpClientModule. Then we create a weatherService.

```tsx
import { Injectable } from "@angular/core";
import { HttpClient, HttpHeaders, HttpParams } from "@angular/common/http";
import { environment } from "src/environments/environment";
import { WeatherData } from "../models/weather.module";
import { Observable } from "rxjs";

@Injectable({
  providedIn: "root",
})
export class WeatherService {
  constructor(private http: HttpClient) {}

  getWeatherData(cityName: string): Observable<WeatherData> {
    return this.http.get<WeatherData>(environment.weatherApiBaseUrl, {
      headers: new HttpHeaders()
        .set(
          environment.XRapidAPIHostHeaderName,
          environment.XRapidAPIHostHeaderValue
        )
        .set(
          environment.XRapidAPIKeyHeaderName,
          environment.XRapidAPIKeyHeaderValue
        ),
      params: new HttpParams()
        .set("q", cityName)
        .set("units", "metric")
        .set("mode", "json"),
    });
  }
}
```

These codes may be seen as complicated a little bit. But we jus set the request due to the api url.

We already know what is the HttpClient and HttpHeaders from the previous project. Bu we haven’t seen HttpParams.

**HttpParams()**:

We add the URL parameters using the helper class HttpParams.The HttpParams is passed as one of the arguments to HttpClient.get() method.

Also we add backend endpoint to the environment class. We use in weatherService file as you see on above.

I forgot mention the interface for api data. As we know when we fetch the data we need to sure data’s type to fetch. So we create an interface and we define all variables type in api.

```tsx
export interface WeatherData {
  coord: Coord;
  weather: Weather[];
  base: string;
  main: Main;
  visibility: number;
  wind: Wind;
  clouds: Clouds;
  dt: number;
  sys: Sys;
  timezone: number;
  id: number;
  name: string;
  cod: number;
}

export interface Coord {
  lon: number;
  lat: number;
}

export interface Weather {
  id: number;
  main: string;
  description: string;
  icon: string;
}

export interface Main {
  temp: number;
  feelslike: number;
  temp_min: number;
  temp_max: number;
  pressure: number;
  humidity: number;
}

export interface Wind {
  speed: number;
  deg: number;
}

export interface Clouds {
  all: number;
}

export interface Sys {
  type: number;
  id: number;
  country: string;
  sunrise: number;
  sunset: number;
}
```

We created this file under the model file.

After finish weatherService and Interface we add some codes in app.componnet.ts.

```tsx
import { Component, OnInit } from "@angular/core";
import { WeatherData } from "./models/weather.module";
import { WeatherService } from "./services/weather.service";

@Component({
  selector: "app-root",
  templateUrl: "./app.component.html",
  styleUrls: ["./app.component.css"],
})
export class AppComponent implements OnInit {
  constructor(private weatherService: WeatherService) {}

  cityName: string = "Wellington";
  weatherData?: WeatherData;

  ngOnInit(): void {
    this.getWeatherData(this.cityName);
    this.cityName = "";
  }

  onSubmit() {
    this.getWeatherData(this.cityName);
    this.cityName = "";
  }

  private getWeatherData(cityName: string) {
    this.weatherService.getWeatherData(cityName).subscribe({
      next: (response) => {
        this.weatherData = response;
        console.log(response);
      },
    });
  }
}
```
