---
title: "Working with APIs in Java"
date: 2023-01-24T07:24:44+03:00
draft: false
weight: 6
description: This lecture presents an introduction to REST APIs and how to consume an API in the Java programming language.
---
## Introduction to REST APIs
API stands for Application Programming Interface. An API often describes a contract of service between the server running the API itself and the client that consumes the API. This contract defines how the API server and client communicates with each other and what the request and response should look like. This contract is often described in the API documentation.

When we talk about APIs, we often mean REST APIs. REST stands for Representational State Transfer. REST defines a set of functions like `GET`, `POST`, `PUT`, `DELETE`, etc. that clients can use to access and send data using HTTP.


APIs are often protected through proper authentication and usage monitoring mechanisms. The most common ones are Authentication tokens and API keys. If you need to use an API, you're often required to sign up and generate an API key or an auth token. This key should be sent with every request you make to the API for the purposes of authenticating your application, preventing misuse, and authorizing your requests. An API often tracks your usage, so you may be asked to pay if you want to make more requests than what the current API plan allows.


REST APIs have the following components:
- **API Host**: The domain name or IP address (IPv4) and port number of the host that serves the API. For example, `api.github.com` is the API host name for the GitHub API.
- **Base path:** This part of the URL that acts as a URL prefix for all API paths. For example, if an API endpoint has the following URL `https://example.com/api/v1` then, the base path is `/api/v1`.
- **endpoint:** An endpoint is the final end of the communication channel. It is often a fancy term for the URL of the server that has a resource from which the responses will be sent to the client. For example, GitHub has the following endpoint: `/repos/{owner}/{repo}/issues/{issue_number}`, which allows you to `GET` all data for a specific reported issue in a repo that belongs to the given GitHub user.
- **Parameter:**
API parameters are the variable parts of the API end point. They're used to determine what type of actions to perform on a resource. There are three common types of API parameters:
  1. **Header parameters:** These are custom HTTP headers that contain meta-data associated with the request. These are often related to authorization tokens are often placed in the header parameters. For example, some endpoints of the GitHub API requires an authorization token in the HTTP header as `Authorization: Bearer <YOUR-TOKEN>`.
  2. **Query parameters** Query parameters are appended to endpoint url in key value pairs separated by ampersands (e.g., `/state=open&page=1`). For example in the endpoint `/repos/{owner}/{repo}/issues/{issue_number}` has an optional query parameter called `state`, which can be one of `open`, `close`, `all`. 
  3. **Path parameters:** Declared within curly braces in the endpoint. For example in the endpoint `/repos/{owner}/{repo}/issues/{issue_number}`, the path parameters can be replaced by values and sent as `/repos/facebook/react/issues/101`.

--------

# Create a simple REST API Client in Java

We're going to write a Java client application that makes use of a weather API. The application prompts the user to enter a city and then makes an HTTP GET request to the weather REST API to get the current weather for that city, parse the JSON response, and display the current weather information.


## OpenWeather API
The [OpenWeather API](https://openweathermap.org/) is a free weather API that allows you to make 1,000 API calls per day for free. You can make calls to get the current weather, 3-hour forecast, 5-days forecast, air-pollution, geocoding and more. Please refer to the [pricing page](https://openweathermap.org/price) for more information. Before we can use this API, we need to sign up for an account and get an API key.

1. Sign up for an account on [OpenWeather API](https://home.openweathermap.org/users/sign_up). Make sure you choose the free plan.
2. After signing up and confirming your email address, go to the top menu and select **"API keys"**. You will need to copy this key to make calls to the openweather API.
![OpenWeather API key](/images/notes/apis/copy-apikey.png)


#### OpenWeather API documentation

Read the [OpenWeather API documentation for the current weather data](https://openweathermap.org/current). We will use this API to get the current weather by our geo location coordinates (combination of latitude and longitude points).

The API documentation shows that in order to make an HTTP GET request to the following endpoint:

```shell
https://api.openweathermap.org/data/2.5/weather?lat={lat}&lon={lon}&appid={API key}
```
It also states that the API requires three query parameters: geographical coordinates (latitude, longitude) named `lat` and `lon` in addition to the unique API key named `apid`.


An HTTP GET can be sent using the CLI with `curl` as:

```shell
curl 'https://api.openweathermap.org/data/2.5/weather?lat=21.543333&lon=39.172778&appid=your-api-key'
```

You should get a response similar to:

```json
{
    "coord": {
        "lon": 39.1728,
        "lat": 21.5433
    },
    "weather": [
        {
            "id": 800,
            "main": "Clear",
            "description": "clear sky",
            "icon": "01n"
        }
    ],
    "base": "stations",
    "main": {
        "temp": 297.13,
        "feels_like": 297.39,
        "temp_min": 297.13,
        "temp_max": 297.13,
        "pressure": 1012,
        "humidity": 69
    },
    "visibility": 10000,
    "wind": {
        "speed": 2.06,
        "deg": 330
    },
    "clouds": {
        "all": 0
    },
    "dt": 1674679736,
    "sys": {
        "type": 1,
        "id": 7410,
        "country": "SA",
        "sunrise": 1674619375,
        "sunset": 1674659269
    },
    "timezone": 10800,
    "id": 105343,
    "name": "Jeddah",
    "cod": 200
}
```

You can also use your browser or an interactive GUI tool like Postman (next below).


## Postman [Optional]
Before we write the code to consume the API, it's always better to see how the API endpoint can be called and what the response looks like. [Postman](https://www.postman.com) is an API client that makes it easy for developers to test, create, prototype, and document APIs. We will download Postman and test the openweather API.

- [Download Postman](https://www.postman.com/downloads/). You may be asked to create a free account to save your work in the cloud.
- Launch Postman and create a new request. From the Postman home screen, click on **New > HTTP Request**, or by selecting **+** to open a new tab.

![Postman HTTP Request](/images/notes/apis/postman-01.png)

- Select `GET` as the HTTP request method and enter the API endpoint 

![Postman HTTP Request](/images/notes/apis/postman-02.png)

- Add the three query parameters: geographical coordinates (latitude, longitude) named `lat` and `lon` in addition to the unique API key named `apid`.

![Postman HTTP Request](/images/notes/apis/postman-03.png)

- Click Send.

--------
Now we know how to work with this API endpoint, we will write a Java application that sends an HTTP request and parse the JSON response into a Java object.


- Create a maven project and add the follow dependencies to the `pom.xml` file.

```xml
    <dependency>
      <groupId>org.apache.httpcomponents.client5</groupId>
      <artifactId>httpclient5</artifactId>
      <version>5.2.1</version>
    </dependency>
```

We will use this useful library for building URLs with query parameters.


Create a class named `HTTPHelper` with the following content:

{{< highlight java "linenos=table, linenostart=1" >}}

import java.io.IOException;
import java.net.MalformedURLException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

public class HTTPHelper {
    public static HttpResponse<String> sendGet(URI uri){
        try {
            // create a client
            HttpClient httpClient = HttpClient.newHttpClient();
            // create an HTTP GET request
            HttpRequest request = HttpRequest.newBuilder(uri)
                    .GET()
                    .header("accept", "application/json")
                    .build();
            // Get the response
            HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());
            // Test if the response from the server is successful
            if (response.statusCode() !=200) {
                System.err.println(response.statusCode());
                System.err.println(response.body());
                return null;
            }
            return response;
        }
        catch (MalformedURLException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return null;
    }
}


{{< / highlight >}}

Add class named `WeatherInfo.java`

{{< highlight java "linenos=table, linenostart=1" >}}

import com.fasterxml.jackson.annotation.JsonProperty;
import java.util.Map;

public class WeatherInfo {
    public double getTempMin() {
        return tempMin;
    }

    public void setTempMin(double tempMin) {
        this.tempMin = tempMin;
    }

    public double getTempMax() {
        return tempMax;
    }

    public void setTempMax(double tempMax) {
        this.tempMax = tempMax;
    }

    private double tempMin;
    private double tempMax;

    @SuppressWarnings("unchecked")
    @JsonProperty("main")
    private void unpackNested(Map<String,Object> main) {
        this.tempMin = (double)main.get("temp_min");
        this.tempMax = (double)main.get("temp_max");
    }

    @Override
    public String toString() {
        return "WeatherInfo{" +
                " tempMin=" + tempMin +
                ", tempMax=" + tempMax +
                '}';
    }
}


{{< / highlight >}}

Finally, add the following to the main class

{{< highlight java "linenos=table, linenostart=1" >}}

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.hc.core5.net.URIBuilder;
import java.net.URI;
import java.net.URISyntaxException;
import java.net.http.HttpResponse;

public class App {
    private final static String API_URL = "https://api.openweathermap.org/data/2.5/weather";

    public static void main(String[] args) {
        URIBuilder uriBuilder = null;
        try {
            uriBuilder = new URIBuilder(API_URL);
            uriBuilder.addParameter("lat", "21.543333");
            uriBuilder.addParameter("lon", "39.172778");
            uriBuilder.addParameter("appid", "your-api-key");
            uriBuilder.addParameter("units", "metric");
            URI uri = uriBuilder.build();
            HttpResponse<String> response = HTTPHelper.sendGet(uri);
            if (response != null) {
                System.out.println(response.body());
                WeatherInfo wInfo = parseWeatherResponse(response.body(), WeatherInfo.class);
                System.out.println(wInfo);
            }

        } catch (URISyntaxException e) {
            e.printStackTrace();
        }
    }

    public static WeatherInfo parseWeatherResponse(String responseString, Class<?> elementClass){
        ObjectMapper objectMapper = new ObjectMapper();
        try {
            JsonNode weatherInfoNode = objectMapper.readTree(responseString);
            WeatherInfo wInfo = new WeatherInfo();
            double tempMin = weatherInfoNode.get("main").get("temp_min").doubleValue();
            double tempMax = weatherInfoNode.get("main").get("temp_max").doubleValue();

            wInfo.setTempMin(tempMin);
            wInfo.setTempMax(tempMax);

            return wInfo;
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            return null;
        }
    }

}

{{< / highlight >}}

