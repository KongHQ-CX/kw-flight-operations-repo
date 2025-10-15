# KongAir Flights API Documentation

## Overview

The **KongAir Flights API** provides real-time access to the scheduled flights operated by **KongAir (airline code: KA)**.  
It enables developers to retrieve information about flights, routes, and operational details such as flight numbers, routes, scheduled departure and arrival times, and aircraft details.

This API is designed to help developers integrate KongAir flight data into their applications for use cases such as flight information displays, travel booking systems, and operational dashboards.

---

## Possible Use Cases

1. **Building a Flight Status Dashboard**  
   Retrieve the list of all scheduled KongAir flights for a given day and display their times, aircraft, and in-flight services.

2. **Integrating Flight Details into a Booking System**  
   Use the API to fetch detailed information for a specific flight, including in-flight entertainment and meal options, to show customers during booking.

---

## Getting Started Guide: Building a Flight Status Dashboard

In this guide, we’ll build a simple workflow to list all scheduled flights for a specific day and retrieve additional details for one flight.

### Sequence of API Calls

1. **List Flights** → `GET /flights`
2. **Get Flight Details** → `GET /flights/{flightNumber}/details`

### Example Using Python

```python
import requests

BASE_URL = "https://api.kong-air.com"
API_KEY = "YOUR_API_KEY"  # Replace with your actual KongAir API key
OIDC_TOKEN = "YOUR_OIDC_ACCESS_TOKEN"  # Replace with your valid OIDC token

HEADERS = {
    "Authorization": f"Bearer {OIDC_TOKEN}",
    "apikey": API_KEY,
    "Accept": "application/json"
}

# 1. Get all flights for today
def get_flights(date=None):
    params = {"date": date} if date else {}
    response = requests.get(f"{BASE_URL}/flights", headers=HEADERS, params=params)
    response.raise_for_status()
    return response.json()

# 2. Get flight details for a specific flight
def get_flight_details(flight_number):
    response = requests.get(f"{BASE_URL}/flights/{flight_number}/details", headers=HEADERS)
    response.raise_for_status()
    return response.json()

if __name__ == "__main__":
    flights = get_flights()
    print("Today's Scheduled Flights:")
    for flight in flights:
        print(f"- {flight['number']} ({flight['route_id']})")

    # Get details for the first flight
    first_flight_number = flights[0]['number']
    details = get_flight_details(first_flight_number)
    print(f"\nDetails for Flight {first_flight_number}:")
    print(details)
````

---

## API Reference

### Authentication

All requests must include:

* **OIDC access token** in the `Authorization` header:
  `Authorization: Bearer <token>`
* **API key** in the `apikey` header.

---

## Endpoints

---

### **GET /health**

**Summary:**
Returns the health status of the KongAir Flights service.

**Description:**
Useful for Kubernetes or monitoring systems to verify if the service is operational.

#### Example Requests

**cURL**

```bash
curl -X GET "https://api.kong-air.com/health" -H "Authorization: Bearer <token>" -H "apikey: <your_api_key>"
```

**Python**

```python
requests.get("https://api.kong-air.com/health", headers=HEADERS)
```

**Go**

```go
resp, _ := http.Get("https://api.kong-air.com/health")
```

#### Responses

| Code                          | Description                    | Example                     |
| ----------------------------- | ------------------------------ | --------------------------- |
| **200 OK**                    | Service is healthy             | `{ "status": "OK" }`        |
| **401 Unauthorized**          | Missing or invalid credentials | *(No body)*                 |
| **500 Internal Server Error** | Service is unhealthy           | `{ "status": "unhealthy" }` |

---

### **GET /flights**

**Summary:**
Retrieve the list of scheduled KongAir flights for a given day.

**Description:**
Each flight represents a scheduled operation of a **leg** within a **route**.
For example, the route **JFK → LAX → NRT** could have:

* **KA100**: Leg 1 (JFK → LAX)
* **KA200**: Leg 2 (LAX → NRT)

#### Query Parameters

| Name   | Type                  | Required | Description                                         |
| ------ | --------------------- | -------- | --------------------------------------------------- |
| `date` | string (format: date) | No       | Filter by a specific date. Defaults to current day. |

#### Example Requests

**cURL**

```bash
curl -X GET "https://api.kong-air.com/flights?date=2024-03-20" \
  -H "Authorization: Bearer <token>" \
  -H "apikey: <your_api_key>"
```

**Python**

```python
requests.get(f"{BASE_URL}/flights", headers=HEADERS, params={"date": "2024-03-20"})
```

**Go**

```go
req, _ := http.NewRequest("GET", "https://api.kong-air.com/flights?date=2024-03-20", nil)
req.Header.Add("Authorization", "Bearer <token>")
req.Header.Add("apikey", "<your_api_key>")
client := &http.Client{}
resp, _ := client.Do(req)
```

#### Responses

| Code                 | Description               | Example                                                                                                                                    |
| -------------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **200 OK**           | List of scheduled flights | `[{"number": "KD924", "route_id": "LHR-SFO", "scheduled_departure": "2024-03-20T09:12:28Z", "scheduled_arrival": "2024-03-20T19:12:28Z"}]` |
| **401 Unauthorized** | Invalid credentials       | *(No body)*                                                                                                                                |

---

### **GET /flights/{flightNumber}**

**Summary:**
Retrieve a specific flight by its number.

**Description:**
Returns detailed information for a specific **flight** — a scheduled operation of a **leg**.

#### Path Parameters

| Name           | Type   | Required | Description                                 |
| -------------- | ------ | -------- | ------------------------------------------- |
| `flightNumber` | string | Yes      | The KongAir flight number (e.g., `KA0284`). |

#### Example Requests

**cURL**

```bash
curl -X GET "https://api.kong-air.com/flights/KA0284" \
  -H "Authorization: Bearer <token>" \
  -H "apikey: <your_api_key>"
```

**Python**

```python
requests.get(f"{BASE_URL}/flights/KA0284", headers=HEADERS)
```

**Go**

```go
req, _ := http.NewRequest("GET", "https://api.kong-air.com/flights/KA0284", nil)
req.Header.Add("Authorization", "Bearer <token>")
req.Header.Add("apikey", "<your_api_key>")
client := &http.Client{}
resp, _ := client.Do(req)
```

#### Responses

| Code                 | Description         | Example                                                                                                                                  |
| -------------------- | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **200 OK**           | Flight found        | `{"number": "KD924", "route_id": "LHR-SFO", "scheduled_departure": "2024-03-20T09:12:28Z", "scheduled_arrival": "2024-03-20T19:12:28Z"}` |
| **401 Unauthorized** | Invalid credentials | *(No body)*                                                                                                                              |
| **404 Not Found**    | Flight not found    | `{ "message": "Flight not found" }`                                                                                                      |

---

### **GET /flights/{flightNumber}/details**

**Summary:**
Fetch additional details about a flight.

**Description:**
Provides more information about the **flight**, including aircraft type, meal options, and whether in-flight entertainment is available.

#### Path Parameters

| Name           | Type   | Required | Description                     |
| -------------- | ------ | -------- | ------------------------------- |
| `flightNumber` | string | Yes      | Flight number (e.g., `KA0293`). |

#### Example Requests

**cURL**

```bash
curl -X GET "https://api.kong-air.com/flights/KA0293/details" \
  -H "Authorization: Bearer <token>" \
  -H "apikey: <your_api_key>"
```

**Python**

```python
requests.get(f"{BASE_URL}/flights/KA0293/details", headers=HEADERS)
```

**Go**

```go
req, _ := http.NewRequest("GET", "https://api.kong-air.com/flights/KA0293/details", nil)
req.Header.Add("Authorization", "Bearer <token>")
req.Header.Add("apikey", "<your_api_key>")
client := &http.Client{}
resp, _ := client.Do(req)
```

#### Responses

| Code                 | Description                  | Example                                                                                                                                    |
| -------------------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **200 OK**           | Flight details retrieved     | `{"flight_number": "KA0293", "in_flight_entertainment": true, "meal_options": ["Vegetarian", "Chicken"], "aircraft_type": "Boeing 787-9"}` |
| **400 Bad Request**  | Invalid flight number format | `{ "message": "Invalid format for parameter" }`                                                                                            |
| **401 Unauthorized** | Invalid credentials          | *(No body)*                                                                                                                                |
| **404 Not Found**    | Flight not found             | `{ "message": "Flight not found" }`                                                                                                        |

---

## Business Context Recap

* **Routes** represent the entire journey between two airports and may contain one or more **legs**.
* **Legs** are individual segments between two airports.
* **Flights** are scheduled operations of legs and are identified by unique flight numbers (e.g., `KA100`).
* **Flight instances** represent actual occurrences of flights on specific days with unique departure and arrival times.
* **Airports** are identified by IATA codes (e.g., `LHR`, `SFO`) and include metadata such as location and timezone.

---
