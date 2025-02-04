# Setup Node.js, Apache, and an Nginx Reverse-Proxy with Docker

## Overview
This project demonstrates setting up a modern web application using **Node.js**, **Apache**, and **Nginx** with **Docker**. The setup includes:
- **Nginx** as a reverse proxy to handle incoming traffic and serve static assets.
- **Node.js** to build pages with content fetched from the PHP API.
- **Apache** running a **PHP API** to serve JSON data.

## Architecture
1. **Nginx** forwards incoming traffic and serves static files.
2. **Node.js** fetches content from the PHP API and renders pages.
3. **Apache with PHP** provides a JSON-based API.

## Prerequisites
- Install **Docker** on your local machine.
- Install **Docker Compose**.

## Installation and Setup
Clone the repository and run the project:
```sh
$ git clone https://github.com/yourusername/your-repo.git && cd your-repo
$ docker-compose up
```
Now, visit [http://localhost:8000](http://localhost:8000) to see the result.

## Project Structure
```
.
├── nginx
│   ├── default.conf
├── nodejs
│   ├── index.js
├── php
│   ├── api
│   │   ├── index.php
│   ├── content
│   │   ├── image.jpg
├── static
│   ├── scripts.js
├── docker-compose.yml
```

## Docker Configuration
Create a `docker-compose.yml` file at the root:
```yaml
version: "3.1"

services:
  nginx:
    image: nginx:alpine
    ports:
      - "8000:80"
    volumes:
      - ./php/content:/srv/www/content
      - ./static:/srv/www/static
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - php
      - nodejs

  nodejs:
    image: node:alpine
    environment:
      NODE_ENV: production
    working_dir: /home/app
    restart: always
    volumes:
      - ./nodejs:/home/app
    depends_on:
      - php
    command: ["node", "index"]

  php:
    image: php:apache
    volumes:
      - ./php:/var/www/html
```

## Nginx Configuration
Inside the `nginx/` directory, create `default.conf`:
```nginx
server {
    listen 80;
    root /srv/www;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    location / {
        try_files $uri @nodejs;
    }

    location /api {
        rewrite ^([^.?]*[^/])$ $1/ break;
        proxy_pass http://php:80;
    }

    location @nodejs {
        proxy_pass http://nodejs:8080;
    }
}
```

## PHP API (Apache)
Inside `php/api/`, create `index.php`:
```php
<?php
$db = [
    ["name" => "apples", "value" => 5, "img" => "/content/apple.jpg"],
    ["name" => "oranges", "value" => 3, "img" => "/content/orange.jpg"],
    ["name" => "pears", "value" => 12, "img" => "/content/pear.jpg"]
];
header("Content-type: application/json");
header("HTTP/1.1 200 Success");
echo json_encode($db);
```

## Node.js Server
Inside `nodejs/`, create `index.js`:
```js
const http = require('http');
const apiUrl = 'http://php:80/api/';

const apiFetch = () => {
    return new Promise((resolve, reject) => {
        http.get(apiUrl, res => {
            let data = '';
            res.on('data', chunk => { data += chunk; });
            res.on('end', () => {
                try {
                    resolve(JSON.parse(data));
                } catch (e) {
                    reject(e.message);
                }
            });
        }).on('error', e => {
            reject(e.message);
        });
    });
};

const server = http.createServer((req, res) => {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    apiFetch().then(json => {
        res.write(`<ul>${json.map(item => `<li><img src="${item.img}">${item.name}: ${item.value}</li>`).join('')}</ul>`);
        res.end();
    }).catch(error => {
        res.write(`<p>${error}</p>`);
        res.end();
    });
});

server.listen(8080);
console.log('Server running at http://localhost:8080/');
```

## Client-Side Rendering
Inside `static/`, create `scripts.js`:
```js
const url = 'http://localhost:8000/api';
fetch(url)
    .then(data => data.json())
    .then(json => {
        document.getElementById('client-list').innerHTML = json.map(item =>
            `<li><img src="${item.img}">${item.name}: ${item.value}</li>`
        ).join('');
    })
    .catch(error => console.log(error));
```

## Running the Application
Start the containers:
```sh
$ docker-compose up
```
Check the output at [http://localhost:8000](http://localhost:8000).

## Next Steps
- Extend the API with more functionality.
- Integrate a front-end framework (Vue.js, React, etc.).
- Deploy the application to a cloud provider.

## License
This project is licensed under the MIT License. Feel free to use and modify!

