# Book Finder App

A simple and secure web application that allows users to search for books using the Google Books API, save favorites, and manage them across sessions. Built with HTML, CSS, and JavaScript, deployed on multiple web servers behind a load balancer.

---

##  Features
- Search for books by title, author, or keyword
- View detailed book information (author, publisher, description, cover image)
- Save and remove favorites
- Responsive design for desktop and mobile
- Secure API key management using `.gitignore`

---

##  Running Locally

### Prerequisites
- Git
- A modern browser (Chrome, Firefox, Edge)
- Node.js (optional, if you want to run a local server)

### Steps
1. Clone the repository:
   ```bash
   git clone https://github.com/nIgihozo/Book-Finder.git
   cd Book-Finder
Create a config.js file in the project root:

js
const API_KEY = "your-google-books-api-key";
export default API_KEY;
Open index.html in your browser to start the app.

###  Deployment to Web Servers
On web01 and web02
SSH into each server.

Clone the repo into /var/www/bookfinder:

bash
sudo git clone https://github.com/nIgihozo/Book-Finder.git /var/www/bookfinder
Add your config.js file manually (not tracked in GitHub).

Configure Nginx:

nginx
server {
  listen 80;
  server_name <server_ip>;
  root /var/www/bookfinder;
  index index.html search.html favorite.html;
  location / {
    try_files $uri $uri/ =404;
  }
}
Restart Nginx:

bash
sudo nginx -t
sudo systemctl restart nginx

- On the Load Balancer
Configured HAProxy to distribute traffic between web-01 and web-02 using round‑robin:

haproxy
frontend bookfinder-frontend
    bind *:80
    mode http
    default_backend bookfinder-backend

backend bookfinder-backend
    balance roundrobin
    server web-01 <web01_ip>:80 check
    server web-02 <web02_ip>:80 check
HAProxy listens on port 80 and forwards requests to the backend servers. This ensures high availability and balanced traffic distribution.

###  APIs Used
Google Books API Official docs: https://developers.google.com/books Used to fetch book data by title, author, or keyword.

###  Challenges & Solutions
- API Key Security: Avoided exposing keys by using .gitignore and manual server-side config.

- Deployment Errors: Fixed permission issues in /var/www by using sudo and adjusting ownership.

- On the Load Balancer refuse to run in the web browser
Load Balancer Access: Couldn’t SSH into the school‑managed LB externally, so functionality was verified directly on the server via HTTP and logs.

Commands used to check if load balancer is working
bash
curl -I http://localhost
Output alternates between:

#### Code
ubuntu@6904-lb-01:~$ curl -I http://localhost
HTTP/1.1 200 OK
server: nginx/1.18.0 (Ubuntu)
date: Tue, 25 Nov 2025 22:54:01 GMT
content-type: text/html
content-length: 612
last-modified: Sat, 08 Nov 2025 09:18:25 GMT
etag: "690f0ae1-264"
x-served-by: 6904-web-02
accept-ranges: bytes

ubuntu@6904-lb-01:~$ curl -I http://localhost
HTTP/1.1 200 OK
server: nginx/1.18.0 (Ubuntu)
date: Tue, 25 Nov 2025 22:54:07 GMT
content-type: text/html
content-length: 30
last-modified: Fri, 07 Nov 2025 13:10:30 GMT
etag: "690defc6-1e"
x-served-by: 6904-web-01
accept-ranges: bytes

- Deadline Pressure: Streamlined deployment to servers + HAProxy configs to minimize manual steps.
- Backend servers (web-01, web-02) serve the Book Finder app via Nginx.
- HAProxy handles load balancing at the LB layer.

### Credits
Google Books API developers for providing free access to book data.

HAProxy for load balancing and traffic distribution.

Nginx on backend servers for serving static files.

School infrastructure team for providing servers and load balancer setup.

Open-source community for documentation and troubleshooting resources.

### Demo Video
- Demo Video: https://youtu.be/eFQ4ngOELJQ?si=v-9YlwZDZwbXdNUU
  
### Web server links
- Link 1 : http://3.84.38.154/  (Web-server 01)
- Link 2 : http://3.87.54.183/  (Web-server 02)
  
