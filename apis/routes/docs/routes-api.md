# KongAir Routes API Documentation

## Overview

The **KongAir Routes API** allows developers to access the network of routes operated by **KongAir (airline code: KA)**.  
A *route* represents a complete journey between an origin and a destination airport and can consist of one or more *legs* (segments). Each route includes important details such as average flight duration and associated airport codes.

Developers can use this API to query all available KongAir routes or to fetch detailed information for a specific route by its identifier.


---

## Possible Use Cases

1. **Displaying Available KongAir Routes in a Booking App**  
   Retrieve all routes KongAir operates and display them for users to select origin and destination options when booking flights.

2. **Building an Operations Dashboard**  
   Retrieve details for a specific route (e.g., `LHR-SYD`) to analyze flight durations and optimize scheduling efficiency for each leg.

---

## Getting Started Guide: Displaying Available Routes in a Booking App

In this guide, you’ll learn how to list all registered KongAir routes and query details for a specific route — a common use case for flight search and booking applications.

### Sequence of API Calls

1. **Get All Routes** → `GET /routes`  
   Retrieves all routes currently registered in the KongAir network.
2. **Get Specific Route** → `GET /routes/{id}`  
   Fetch detailed information for a single route using its ID (e.g., `LHR-SFO`).

### Example Implementation (Python)

```python
import requests

BASE_URL = "https://api.kong-air.com"
API_KEY = "YOUR_API_KEY"  # Replace with your valid API key
OIDC_TOKEN = "YOUR_OIDC_ACCESS_TOKEN"  # Replace with your valid OIDC token

HEADERS = {
    "Authorization": f"Bearer {OIDC_TOKEN}",
    "apikey": API_KEY,
    "Accept": "application/json"
}

# 1. Get all routes
def get_routes(origin=None):
    params = {"origin": origin} if origin else {}
    response = requests.get(f"{BASE_URL}/routes", headers=HEADERS, params=params)
    response.raise_for_status()
    return response.json()

# 2. Get a specific route by ID
def get_route_by_id(route_id):
    response = requests.get(f"{BASE_URL}/routes/{route_id}", headers=HEADERS)
    response.raise_for_status()
    return response.json()

if __name__ == "__main__":
    # Get all routes
    routes = get_routes()
    print("Available KongAir Routes:")
    for route in routes:
        print(f"- {route['id']} ({route['origin']} → {route['destination']}) | Avg Duration: {route['avg_duration']} mins")

    # Get details for a specific route
    route_id = routes[0]["id"]
    route_details = get_route_by_id(route_id)
    print(f"\nDetails for Route {route_id}:")
    print(route_details)
````

---

## API Reference

### Authentication

All requests to KongAir APIs must include:

* **OIDC access token** in the `Authorization` header:
  `Authorization: Bearer <token>`
* **API key** in the `apikey` header.

---

## Endpoints

---

### **GET /health**

**Summary:**
Check if the Routes service is operational.

**Description:**
This endpoint is mainly used by monitoring systems (e.g., Kubernetes) to verify the health of the service.

#### Example Requests

**cURL**

```bash
curl -X GET "https://api.kong-air.com/health" \
  -H "Authorization: Bearer <token>" \
  -H "apikey: <your_api_key>"
```

**Python**

```python
requests.get(f"{BASE_URL}/health", headers=HEADERS)
```

**Go**

```go
req, _ := http.NewRequest("GET", "https://api.kong-air.com/health", nil)
req.Header.Add("Authorization", "Bearer <token>")
req.Header.Add("apikey", "<your_api_key>")
client := &http.Client{}
resp, _ := client.Do(req)
```

#### Responses

| Code                          | Description          | Example                     |
| ----------------------------- | -------------------- | --------------------------- |
| **200 OK**                    | Service is healthy   | `{ "status": "OK" }`        |
| **500 Internal Server Error** | Service is unhealthy | `{ "status": "unhealthy" }` |

---

### **GET /routes**

**Summary:**
Retrieve all registered KongAir routes.

**Description:**
Returns a list of routes that **KongAir (KA)** operates.
Each route defines the journey between an **origin** and **destination** airport, which may include intermediate stops (multiple *legs*).

For example, the route `JFK → LAX → NRT` represents two legs:

* **Leg 1:** JFK → LAX
* **Leg 2:** LAX → NRT

Each leg may be part of multiple routes, and routes have an average duration measured in minutes.

#### Query Parameters

| Name     | Type             | Required | Description                                        |
| -------- | ---------------- | -------- | -------------------------------------------------- |
| `origin` | array of strings | No       | Filter routes by one or more origin airport codes. |

#### Example Requests

**cURL**

```bash
curl -X GET "https://api.kong-air.com/routes?origin=LHR" \
  -H "Authorization: Bearer <token>" \
  -H "apikey: <your_api_key>"
```

**Python**

```python
requests.get(f"{BASE_URL}/routes", headers=HEADERS, params={"origin": "LHR"})
```

**Go**

```go
req, _ := http.NewRequest("GET", "https://api.kong-air.com/routes?origin=LHR", nil)
req.Header.Add("Authorization", "Bearer <token>")
req.Header.Add("apikey", "<your_api_key>")
client := &http.Client{}
resp, _ := client.Do(req)
```

#### Responses

| Code       | Description                        | Example                                                                           |
| ---------- | ---------------------------------- | --------------------------------------------------------------------------------- |
| **200 OK** | Successful retrieval of all routes | `[{"id": "LHR-SFO", "origin": "LHR", "destination": "SFO", "avg_duration": 660}]` |

---

### **GET /routes/{id}**

**Summary:**
Retrieve details for a specific KongAir route by its ID.

**Description:**
Returns route information identified by the route ID (e.g., `LHR-SYD`).
A route connects an origin airport to a destination airport and may include intermediate stops. Each route contains:

* `id`: A unique identifier (e.g., `LHR-SFO`)
* `origin`: The origin airport IATA code
* `destination`: The destination airport IATA code
* `avg_duration`: Average total duration (minutes)

#### Path Parameters

| Name | Type   | Required | Description                             |
| ---- | ------ | -------- | --------------------------------------- |
| `id` | string | Yes      | The route identifier (e.g., `LHR-SYD`). |

#### Example Requests

**cURL**

```bash
curl -X GET "https://api.kong-air.com/routes/LHR-SYD" \
  -H "Authorization: Bearer <token>" \
  -H "apikey: <your_api_key>"
```

**Python**

```python
requests.get(f"{BASE_URL}/routes/LHR-SYD", headers=HEADERS)
```

**Go**

```go
req, _ := http.NewRequest("GET", "https://api.kong-air.com/routes/LHR-SYD", nil)
req.Header.Add("Authorization", "Bearer <token>")
req.Header.Add("apikey", "<your_api_key>")
client := &http.Client{}
resp, _ := client.Do(req)
```

#### Responses

| Code                | Description             | Example                                                                          |
| ------------------- | ----------------------- | -------------------------------------------------------------------------------- |
| **200 OK**          | Route found             | `{"id": "LHR-SYD", "origin": "LHR", "destination": "SYD", "avg_duration": 1320}` |
| **400 Bad Request** | Invalid route ID format | `{ "message": "Invalid format for parameter" }`                                  |
| **404 Not Found**   | Route not found         | *(No body returned)*                                                             |

---

## Business Context Summary

* **Routes**: The full journey between two airports, possibly including stops or connections.
  Example: `New York (JFK) → Los Angeles (LAX) → Tokyo (NRT)`

  * *Leg 1*: JFK → LAX
  * *Leg 2*: LAX → NRT

* **Legs**: Single flight segments between two airports.
  The same leg can be part of multiple routes.

* **Flights**: Scheduled operations of legs, identified by unique flight numbers (e.g., `KA100`).

* **Flight Instances**: Actual flight occurrences on specific dates.

* **Airports**: Identified by IATA codes (e.g., `LHR`, `SFO`) and include location, country, and timezone metadata.
