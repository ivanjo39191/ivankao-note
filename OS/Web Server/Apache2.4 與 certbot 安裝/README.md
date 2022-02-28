# Apache2.4 與 certbot 安裝

Created: February 1, 2022 5:22 PM
Tags: OS

1. Apache安裝
    
    到apache 官網下載 apache2.4.41
    
    ```jsx
    wget https://downloads.apache.org//httpd/httpd-2.4.46.tar.gz
    ```
    
    安裝pcre
    
    ```bash
    # centos7
    yum -y install pcre pcre-devel
    # ubuntu
    sudo apt-cache search pcre
    ```
    
    安裝 apache至你指定的路徑
    
    ```bash
    ./configure --prefix /usr/local/apache2.4  --with-pcre=/usr/pcre/bin/pcre-config
    ```
    
    Linux中安裝Apache 編譯出現問題：
    
    解決辦法：
    
    1、下載所需要的軟件包
    
    ```bash
    wget http://archive.apache.org/dist/apr/apr-1.4.5.tar.gz
    wget http://archive.apache.org/dist/apr/apr-util-1.3.12.tar.gz
    wget http://jaist.dl.sourceforge.net/project/pcre/pcre/8.10/pcre-8.10.zip
    ```
    
    2、為了下面編譯不報錯，最好先裝上編譯環境和工具
    
    **yum** -y install gcc+ gcc-c++
    
    3、解決APR not found報錯
    
    ```jsx
    tar -zxf apr-1.4.5.tar.gz
    cd apr-1.4.5
    ./configure --prefix=/usr/local/apr
    make && make install
    ```
    
    4、解決APR-util not found問題
    
    ```jsx
    cd apr-util-1.3.12
    ./configure --prefix=/usr/local/apr-util -with-apr=/usr/local/apr/bin/apr-1-config
    make && make install
    ```
    
    5、解決pcre問題
    
    ```jsx
    unzip -o pcre-8.10.zip
    cd pcre-8.10
    ./configure --prefix=/usr/local/pcre
    make && make install
    ```
    
    6、最後編譯加上
    
    ```bash
    cd httpd-2.4.46
    ./configure --prefix /usr/local/apache2.4 --**with**-apr=/usr/local/apr/ --**with**-apr-util=/usr/local/apr-util/ --**with**-pcre=/usr/local/pcre
    make && make install
    ```
    
    7、完成。
    
    解決安裝Apache中出現checking for APR... no configure: error: APR not found. Please read the documentation的問題
    
2. 下載 mod_wsgi
    
    [https://pypi.org/project/mod-wsgi/#modal-close](https://pypi.org/project/mod-wsgi/#modal-close)
    
    ```bash
    # wget 取得 mod-wsgi 4.7.1 (python3.8適用)
    wget https://files.pythonhosted.org/packages/74/98/812e68f5a1d51e9fe760c26fa2aef32147262a5985c4317329b6580e1ea9/mod_wsgi-4.7.1.tar.gz
    tar -zxvf mod_wsgi-4.7.1.tar.gz
    cd  mod_wsgi-4.7.1
    ./configure --with-apxs=/usr/local/apache2.4/bin/apxs --with-python=/usr/local/bin/python3.8
    make && make install
    ```
    
3. 安裝 certbot
    
    ```bash
    yum install -y epel-release
    wget https://dl.eff.org/certbot-auto
    chmod a+x certbot-auto
    
    # ubuntu
    sudo snap install --classic certbot
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    ```
    
    指令說明:
    
    ```bash
    # --apache 使用 apache 模式
    # --apache-server-root=/usr/local/apache2.4 指定apache路徑
    # --apache-ctl=/usr/local/apache2.4/bin/apachectl 指定 ctl
    # --apache-bin=/usr/local/apache2.4/bin 指定 bin
    
    ./certbot-auto --apache --apache-server-root=/usr/local/apache2.4 --domain=www.example.com certonly
    #./certbot-auto --apache --apache-server-root=/usr/local/apache2.4 --domain=www.example.com certonly
    
    sudo certbot --apache --apache-server-root=/usr/local/apache2.4 --apache-bin=/usr/local/apache2.4/bin --apache-ctl=/usr/local/apache2.4/bin/apachectl --apache-bin=/usr/local/apache2.4/bin --domain=www.example.com  certonly
    # 更新憑證
    sudo certbot renew
    # 調整 apache 設定
    
    LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
    LoadModule ssl_module modules/mod_ssl.so
    LoadModule rewrite_module modules/mod_rewrite.so
    
    ```
    
    錯誤處理:
    
    1. Unable to find a virtual host listening on port 80 which is currently needed for Certbot to prove to the CA that you control your domain. Please add a virtual host for port 80.
        
        須建立 virtualhost :80
        
        ```bash
        cd /usr/local/apache2.4/conf/extra
        touch telegrambot.conf
        vim telegrambot.conf
        ##### 
        <IfModule !wsgi_module>
            LoadModule wsgi_module /usr/local/apache2.4/modules/mod_wsgi.so
        </IfModule>
        
        WSGISocketPrefix /var/run/wsgi
        
        <VirtualHost *:80>
        	<Directory "/opt/path/src">
            <Files wsgi.py>
                Require all granted
            </Files>
        </Directory>
        
        WSGIDaemonProcess wsginame user=username group=usergroup socket-user=daemon
        WSGIProcessGroup wsginame
        
        WSGIScriptAlias / /opt/path/src/config/wsgi.py
        
        </VirtualHost>
        
        #####
        ```
        
    2. Cleaning up challenges An unexpected error occurred:
    File: /etc/httpd/conf.d/le_http_01_challenge_pre.conf - Could not be found to be deleted
     - Certbot probably shut down unexpectedly
    File: /etc/httpd/conf.d/le_http_01_challenge_post.conf - Could not be found to be deleted
     - Certbot probably shut down unexpectedly
    An unexpected error occurred:
    FileNotFoundError: [Errno 2] No such file or directory: '/etc/httpd/conf.d/le_http_01_challenge_pre.conf'
        
        直接強改:
        
        ```bash
        cd /opt/eff.org/certbot/venv/lib/python3.6/site-packages/certbot_apache/_internal/
        vim override_centos.py
        
        將路徑都替換為 /usr/local/apache2.4/ 和 /usr/local/apache2.4/conf/
        
        ```
        
    3. Cleaning up challenges Error while running /usr/local/apache2.4/bin/apachectl configtest.
        
        AH00526: Syntax error on line 1 of /usr/local/apache2.4/conf/le_http_01_challenge_pre.conf:
        Invalid command 'RewriteEngine', perhaps misspelled or defined by a module not included in the server configuration
        
        將 httpd.conf 的幾個 mod 開啟
        
        ```bash
        LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
        LoadModule ssl_module modules/mod_ssl.so
        LoadModule rewrite_module modules/mod_rewrite.so
        ```
        
    
    生成的 憑證檔案位置在
    
    ```bash
    /etc/letsencrypt/live/www.example.com
    ```
    
4.  使用 ssl 憑證
    
    ```bash
    vim /usr/local/apache2.4/conf/httpd.conf
    
    開啟 httpd-ssl.conf
    ###
    Include conf/extra/httpd-ssl.conf
    ###
    
    cd /extra
    vim httpd-ssl.conf
    ###
    DocumentRoot "/usr/local/apache2.4/htdocs"
    SSLCertificateFile "/etc/letsencrypt/live/www.example.com/cert.pem"
    SSLCertificateKeyFile "/etc/letsencrypt/live/www.example.com/privkey.pem"
    SSLCertificateChainFile "/etc/letsencrypt/live/www.example.com/chain.pem"
    ```