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
On the Load Balancer
Configure Nginx to proxy traffic to web01 and web02:

nginx
upstream bookfinder_backend {
  server <web01_ip>;
  server <web02_ip>;
}

server {
  listen 80;
  server_name <load_balancer_ip>;
  location / {
    proxy_pass http://bookfinder_backend;
  }
}
###  APIs Used
Google Books API Official docs: https://developers.google.com/books Used to fetch book data by title, author, or keyword.

###  Challenges & Solutions
API Key Security: Avoided exposing keys by using .gitignore and manual server-side config.

Deployment Errors: Fixed permission issues in /var/www by using sudo and adjusting ownership.

Load Balancer Access: Couldn’t SSH into the school-managed LB, so verified functionality via HTTP and logs.

Deadline Pressure: Streamlined deployment with GitHub + Nginx configs to minimize manual steps.

### Credits
Google Books API developers for providing free access to book data.

Nginx for load balancing and serving static files.

School infrastructure team for providing servers and load balancer setup.

Open-source community for documentation and troubleshooting resources.

### Demo Videos
