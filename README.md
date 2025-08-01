
````
# Web Infrastructure Lab

This repository sets up a simple container-based web infrastructure lab using Docker Compose. It demonstrates basic load balancing using HAProxy across two web servers running a static site.

---

## Demo Video
```
https://drive.google.com/file/d/1q7stdKsk0tgfx3Ua7KmyXhVCTn2uYOuR/view?usp=sharing
```

## 📦 Stack Overview

- **web-01** – Static site container (port 8080, SSH 2211)
- **web-02** – Static site container (port 8081, SSH 2212)
- **lb-01** – HAProxy load balancer (port 8082, SSH 2210)

All services are connected to a custom bridge network `lablan` with static IPs to allow stable backend referencing by the load balancer.

---

## ✅ Requirements

- Docker
- Docker Compose
- ~2 GB of free RAM
- Linux or WSL2 recommended for networking consistency

---

## 🚀 Deployment Instructions

Follow these steps to deploy the application and configure the load balancer:

### 1. Clone the repository

```bash
git clone git@github.com:Udopara/api-summative.git
cd api-summative
````

### 2. Build and start all containers

```bash
docker compose up -d --build
```

This command:

* Builds Docker images if necessary.
* Starts three containers (`web-01`, `web-02`, and `lb-01`) in detached mode.
* Connects all containers to the `lablan` network with static IPs (configured in `docker-compose.yml`).

### 3. Confirm all containers are running

```bash
docker compose ps
```

Expected output:

```
       Name                 Command               State           Ports
-------------------------------------------------------------------------
web-01          ...                         Up       0.0.0.0:8080->80/tcp, 2211->22/tcp
web-02          ...                         Up       0.0.0.0:8081->80/tcp, 2212->22/tcp
lb-01           haproxy -f /usr/local/etc/haproxy/haproxy.cfg  Up       0.0.0.0:8082->80/tcp, 2210->22/tcp
```

### 4. Verify network connectivity

Inside the `lb-01` container, the HAProxy load balancer resolves `web-01` and `web-02` by their container hostnames on the custom Docker network (`lablan`), allowing it to balance HTTP traffic correctly.

---

## 🌐 HAProxy Load Balancer Configuration

Located at `./lb-01/haproxy.cfg`:

```haproxy
global
  daemon
  maxconn 256

defaults
  mode http
  timeout connect 5000ms
  timeout client  50000ms
  timeout server  50000ms

frontend http-in
  bind *:80
  default_backend web_backends

backend web_backends
  balance roundrobin
  server web01 web-01:80 check
  server web02 web-02:80 check
  http-response set-header X-Served-By %[srv_name]
```

**Key points:**

* The load balancer listens on port 80 inside its container.
* Uses **round-robin** strategy to distribute requests evenly between `web-01` and `web-02`.
* Health checks (`check`) ensure only healthy backends receive traffic.
* Custom HTTP header `X-Served-By` identifies which backend served the request — useful for verifying load balancing.

---

## 🧪 Testing the Load Balancer

Test the load balancer on your host machine by sending multiple requests:

```bash
curl -I http://localhost:8082
```

Look for the response header:

```
X-Served-By: web01
```

or

```
X-Served-By: web02
```

By running the curl command multiple times, you should see the header alternate between `web01` and `web02`, confirming that requests are balanced correctly.

You can also open [http://localhost:8082](http://localhost:8082) repeatedly in a browser to visually confirm the load balancer is directing traffic across both web servers.

---

## 🐳 Docker Image Information

* **Image ID:** `sha256:9d06846b3acf6437f92a38640e96283cf47aa35a3a4fb8370ebf7192b75dbe3c`
* **Tagged as:** `puopara/request-app:v1`
* **Base OS:** Ubuntu 24.04
* **Exposed Ports:** 80 (HTTP), 22 (SSH)
* **Static Content Path:** `/var/www/html/index.html`

---

## 🔐 Accessing Containers via SSH

You can SSH into each container for inspection or troubleshooting:

| Container | Command                        | SSH Port | Password  |
| --------- | ------------------------------ | -------- | --------- |
| web-01    | `ssh ubuntu@localhost -p 2211` | 2211     | `pass123` |
| web-02    | `ssh ubuntu@localhost -p 2212` | 2212     | `pass123` |
| lb-01     | `ssh ubuntu@localhost -p 2210` | 2210     | `pass123` |

---

## 🧼 Tearing Down

To stop and remove all containers and networks created:

```bash
docker compose down
```

---

## 📁 Project Structure Overview

```
api-summative/
├── docker-compose.yml
├── lb-01/
│   └── haproxy.cfg
└── README.md
```


##🧩 Frappe REST API Integration
```
This project uses the Frappe REST API to interact with a custom Doctype named Request. This doctype handles user maintenance requests, which can be created, viewed, updated, or deleted via API calls.

API Endpoints
The Frappe REST API base URL is typically:

perl
Copy
Edit
http://<frappe-server>/api/resource/Request
Example to create a new request (POST):

http
Copy
Edit
POST /api/resource/Request
Content-Type: application/json

{
  "data": {
    "category": "Fix streetlight on 5th Ave",
    "description": "The streetlight near the corner is not working.",
    "status": "Open",
    "priority: "Medium"
  }
}
Official Documentation
Frappe REST API: https://frappeframework.com/docs/user/en/api/rest

Custom DocTypes: https://frappeframework.com/docs/user/en/basics/doctype

⚠️ Challenges & Solutions
1. Networking Between Containers and Load Balancer
Challenge: Ensuring HAProxy could reliably route requests to backend containers by hostname.

Solution: Configured a custom Docker network (lablan) with static IP assignments so HAProxy can resolve container hostnames consistently.

2. Handling API Authentication & CORS
Challenge: Securely interacting with the Frappe REST API while respecting CORS policies.

Solution: Implemented proper API token authentication headers in requests and configured CORS settings on the Frappe server to allow requests from frontend origins.

3. Maintaining State Consistency
Challenge: Ensuring user requests via the API reflect correctly across multiple backend containers.

Solution: Centralized API backend (Frappe server) separate from static web containers; all API calls target the same Frappe backend ensuring consistent state.

🙏 Credits & Resources
Frappe Framework & REST API — https://frappeframework.com
Used extensively for backend API services and custom Doctype management.

HAProxy — https://www.haproxy.org
For robust and performant HTTP load balancing.

Docker & Docker Compose — https://www.docker.com
Containerization and orchestration of services.

Open-source libraries and tools referenced throughout the project.
```

---

## 📌 Additional Notes

* All containers run Ubuntu 24.04 and include OpenSSH for remote access.
* Static web content can be customized by modifying the app image or replacing files under `/var/www/html/` inside the containers.
* The `lablan` Docker network enables static IP assignment and reliable hostname resolution critical for HAProxy backend targeting.
* Health checks in HAProxy ensure faulty web servers are automatically removed from the load balancing rotation.

---
