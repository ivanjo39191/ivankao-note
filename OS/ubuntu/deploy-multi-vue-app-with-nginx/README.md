# Deploy multiple Vue app with Nginx

Created: Mar 21, 2021 1:05 PM  
Property: ivan kao  
Tags: OS, Vue  

1. How To Install Nginx on Ubuntu 20.04

    安裝 nginx

    ```bash
    sudo apt update
    sudo apt install nginx
    sudo ufw allow 'Nginx HTTP'

    # 基本指令
    systemctl status nginx # 檢查狀態
    systemctl stop nginx # 停止
    systemctl start nginx # 運行
    systemctl restart nginx # 重啟

    ```

    預設 nginx 目錄位置

    ```bash
    /etc/nginx/
    ```

    基本配置

    ```bash
    # 主設定檔案
    /etc/nginx/nginx.conf

    # 配置檔案目錄
    /etc/nginx/sites-enabled

    # 預設設定檔
    /etc/nginx/sites-enabled/default
    ```

2. Build Vue

    Vue app1專案配置

    ```bash
    # vue.config.js
    module.exports = {
       publicPath: "/app1" 
    }

    # router/index.js
    export default new Router({
        mode: 'history',
        base: '/app1',
        routes: [...]
    })
    ```

    Vue app2 專案配置

    ```bash
    # vue.config.js
    module.exports = {
       publicPath: "/app2" 
    }

    # router/index.js
    export default new Router({
        mode: 'history',
        base: '/app2',
        routes: [...]
    })
    ```

    軟連結 dist 目錄位置

    ```bash
    # 放置在 /opt 底下
    ## app1
    ln -s /path/app1/dist ./app1
    ## app2 
    ln -s /path/app2/dist ./app2
    ```

3. nginx 配置

    預設配置:

    ```bash
    server {
    	listen 80 default_server;
    	listen [::]:80 default_server;

    	root /var/www/html;

    	# Add index.php to the list if you are using PHP
    	index index.html index.htm index.nginx-debian.html;

    	server_name _;

    	location / {
    		# First attempt to serve request as file, then
    		# as directory, then fall back to displaying a 404.
    		try_files $uri $uri/ =404;
    	}

    }
    ```

    Vue 配置

    ```bash
    server {
    	listen 80;
    	listen [::]:80;

    	server_name www.example.com;

    	root /opt/app1;
    	index index.html;

    	location / {
    		try_files $uri $uri/ =404;
    	}
    	
    	location /admin {
    			alias /opt/app1;
    			try_files $uri $uri/ /index.html =404;
    		}
    	location /index {
    			alias /opt/app2;
    			try_files $uri $uri/ /index.html =404;
    		}
    }
    ```

4. 運行

    ```bash
    systemctl start nginx # 運行
    ```