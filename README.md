# Book Finder App
A simple and secure web application that allows users to search for books using the Google Books API, save favorites, and manage them across sessions. The application is built with HTML, CSS, and JavaScript, and deployed on multiple web servers behind an HAProxy Load Balancer to ensure high availability and balanced traffic distribution.

## Features
Search Functionality: Users can search for books by title, author, or keyword using the Google Books API.

Detailed Information: View book details including author, publisher, description, and cover image.

Favorite Management: Ability to save and remove books from a favorites list across sessions.

Responsive Design: Optimized layout for seamless use on desktop and mobile devices.

Secure API Key Management: Google Books API key is managed securely using server-side configuration and .gitignore.

## Running Locally
Prerequisites
Git

A modern web browser (Chrome, Firefox, Edge)

Node.js (Optional, for running a simple local HTTP server)

Steps
Clone the repository:

Bash

git clone https://github.com/nIgihozo/Book-Finder.git
cd Book-Finder
Create API Configuration File: Create a file named config.js in the project root and insert your Google Books API key (this file should be ignored by Git for security):

JavaScript

const API_KEY = "your-google-books-api-key"; 
export default API_KEY;
Run the Application: Open the index.html file directly in your browser, or use a simple HTTP server (e.g., Node's serve package or Python's SimpleHTTPServer).

## Deployment to Web Servers (web-01 & web-02)
The application is served by Nginx on two backend servers (web-01 and web-02).

1. Backend Server Setup (Perform on both web-01 and web-02)
Clone the Repository: SSH into each server and clone the application into the custom Nginx root directory:

Bash

sudo git clone https://github.com/nIgihozo/Book-Finder.git /var/www/bookfinder
Set Permissions: Ensure Nginx can read the files:

Bash

sudo chown -R www-data:www-data /var/www/bookfinder
Add config.js: Manually create and add the config.js file with your API key to /var/www/bookfinder/.

2. Nginx Configuration
Create the configuration file /etc/nginx/sites-available/bookfinder:

Nginx

server {
    listen 80;
    server_name <server_ip>; # Use the public IP of the web server
    root /var/www/bookfinder;
    index index.html search.html favorite.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
3. Activate and Finalize Nginx
Activate the Site: Create the symbolic link:

Bash

sudo ln -s /etc/nginx/sites-available/bookfinder /etc/nginx/sites-enabled/
Disable Default Site (Crucial Fix): Remove the conflicting default configuration to prevent serving the "Welcome to Nginx" page:

Bash

sudo unlink /etc/nginx/sites-enabled/default
Test and Restart:

Bash

sudo nginx -t
sudo systemctl restart nginx
## Load Balancer Setup (HAProxy)
HAProxy is configured to listen on port 80 and distribute incoming requests across the two backend servers using the roundrobin algorithm.

HAProxy Configuration (/etc/haproxy/haproxy.cfg)
Code snippet

frontend bookfinder-frontend
    bind *:80
    mode http
    default_backend bookfinder-backend

backend bookfinder-backend
    balance roundrobin
    server web-01 <web01_ip>:80 check
    server web-02 <web02_ip>:80 check

Verify and Restart HAProxy
Bash

sudo haproxy -c -f /etc/haproxy/haproxy.cfg
sudo systemctl restart haproxy
##  APIs Used
Google Books API

Official documentation: https://developers.google.com/books

Usage: Used to fetch book data by title, author, or keyword, displaying the results to the user.

Challenges & Solutions
HAProxy Startup Failure

Encountered: The HAProxy service failed to start due to overly verbose and conflicting settings inherited from the default global section (e.g., unnecessary stats socket, chroot, and default SSL directives).

Solution: Simplified the global section of the HAProxy configuration to its minimum required directives (log, daemon, maxconn). This allowed the service to initialize and run correctly.

Inconsistent Backend Response

Encountered: The Load Balancer served inconsistent HTML—sometimes the Book Finder app, and other times the "Welcome to Nginx" default page—because of the roundrobin balancing algorithm hitting different server states.

Solution: The issue was resolved by two clean-up steps on both backend servers:

Removal: Deleted the conflicting default Nginx files (/var/www/html/index.nginx-debian.html).

Disabling: Unlinked the default Nginx configuration (sudo unlink /etc/nginx/sites-enabled/default). This forced Nginx to respect only the dedicated bookfinder configuration on Port 80.

External Browser Access

Encountered: After fixing all server-side issues (backend Nginx and HAProxy), the local browser continued to show "Site can't be reached", even though curl from the client machine was successful.

Solution: The issue was aggressive browser caching of the initial connection failure. Verified functionality by using an Incognito Window and advising a hard refresh to force the browser to establish a fresh connection.

Load Balancer Verification Command
The following command, run from the client machine, successfully confirmed the final operational state of the load balancer:

Bash

curl http://18.206.40.183 
### Output now consistently returns the full HTML for the Book Finder application.
## Credits
Google Books API developers for providing free and reliable access to book data.

HAProxy for load balancing and high availability traffic distribution.

Nginx on backend servers for efficient static file serving.

Open-source community for documentation and troubleshooting resources.

## Demo Video
- Demo Video: https://youtu.be/eFQ4ngOELJQ?si=v-9YlwZDzwbXdNUU

## Web Server Links (For direct access verification)

- Link 1 (Web-server 01): http://3.84.38.154/
- Link 2 (Web-server 02): http://3.87.54.183/
- Link 3 (Lb-server 01):  http://18.206.40.183/ 
