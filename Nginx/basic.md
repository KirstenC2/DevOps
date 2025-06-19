# Nginx

## What is Nginx?
- web server
- can be used as:
    - reverse proxy
    - load-balancer
    - mail-proxy
    - HTTP cache
    - proxy server for email communications protocols

## NGINX Architecture
- master-slave architecture

![Nginx Architecture](images/nginxArchi.png)

Based on the image, there would be a master process linked with fellow child processes. (aka Worker Processes)

### Master Process
when it is started, it is responsible for:
- read and validate the configuration
    - nginx.conf
- Manage worker processes (master's slave)
    - start, stop, reload
- Handle signals
    - reload, stop, reopen logs
- it will not handle client request directly

### Worker Processes
each worker is *single-threaded*.
- event-driven
- non-blocking I/O
- can handle thousand of concurrent connections
- configuration:
```
worker_processes auto;
```

#### Worker's Event-driven model
each worker runs an **Event Loop**
- uses system-level APIs:
    - epoll (Linux)
    - kqueue (macos/BSD)
    - IOCP (Windows)
- allow high concurrency without spawing threads or processes per connection

### Analogy: NGINX as a restaurant
Imagine NGINX is a restaurant. Here’s the cast:

🧑‍🍳 Master Process = 餐廳經理
🧍‍♂️ Worker Processes = 服務生
👥 Clients = 來用餐的客人 (web browsers)
🍽️ Static Menu = Static files (HTML, JS, CSS)
🧾 Custom Orders to Kitchen = Proxy to backend server (like Node.js/Express)
🧂 Config File = Restaurant Rulebook (nginx.conf)

#### Workflow
1. 客人（browser）走進來餐廳，然後點單（make a request）.   //can be "Give me /home"
2. 餐廳經理（Master process）不會直接接單，而是吩咐服務生（worker process）去處理訂單。
3. 服務生就會檢查（Rulebook）去看說，request點的是不是在菜單上的東西，有的話就會serve directly。但是如果點的東西不在menu上，服務生就會把該request傳去給後廚（backend server）。
    - 這邊說的rulebook就是config file
    ```
    config file說的就是一個.conf， 用來控制Nginx web server要怎樣運作。
    ```
4. 服務生當然就會把客人點的東西送給他們
    - 點的東西（HTML或者API response）
5. 客人結束了用餐就會離開（或者在web會keep connection alive ）

## NGINX's Example
```
# 全域區塊（Global Settings）
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    # 設定多個虛擬主機（server）
    server {
        listen       80;
        server_name  example.com;

        root   /usr/share/nginx/html;
        index  index.html;

        location / {
            try_files $uri $uri/ =404;
        }

        location /api/ {
            proxy_pass http://localhost:3000/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        error_page 404 /404.html;
    }
}

```

### 配置說明
#### Global Area -- main 
```
worker_processes auto;   # 根據 CPU 自動配置
pid /var/run/nginx.pid;
```
在這個main area裡面主要是：
1. set 工作進程數
- auto： nginx會根據CPU的數量自己進行調整
```
worker_processes auto;
```
2. 指定error log要存放的path
```
error_log /path/to/error.log;
```
3. set 好要存放nginx's process ID 的path
```
pid /path/to/nginx.pid;
```
4. 匯入其他的config file
```
include /etc/nginx/conf.d/*.conf;
```

- 這邊就可以進行拆分configuration，從而變的更加readable跟modularizable
- Suggestion for production
    - normally only need to set for worker_processes， error_log, pid
    - use include to separate the large configuration set (尤其是大型專案)

#### Event Area  
```
events {
    worker_connections 1024;   # 每個 worker 可處理的最大連線數
}
```
這個區塊呢，是獨立存在的。
能做的：
1. set好 maximum connection count
```
worker_connections 1024;
```
- means every worker can handle 1024 connections at the same time.
2. I/O model selection
```
use epoll;
```
- 這個是based on systemOS, linux = epoll.
- normally automatically chosen, unless special requirements.
3. multi-worker 接受連線開啟
```
multi_accept on;
```
- 一次接受多個新連線，提高效率
- Suggestion for production
    - worker——connections 建議set高一點（例如2048）
    - 最大連線數 = worker processes * worker connections


#### http Area  
```
http {
    include mime.types;               # 支援多種檔案類型
    default_type application/octet-stream;

    sendfile on;                      # 啟用高效檔案傳輸
    tcp_nopush on;
    keepalive_timeout 65;

    gzip on;                          # 壓縮設定
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    client_max_body_size 10M;         # 上傳檔案最大限制
}

```

#### server Area  (Virtual Machine)
```
server {
    listen 80;
    server_name mysite.com;

    root /var/www/mysite;
    index index.html index.htm;

    access_log /var/log/nginx/mysite_access.log;
    error_log  /var/log/nginx/mysite_error.log;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    error_page 404 /404.html;
}
```
