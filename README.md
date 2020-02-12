## Docker 

### Docker 概述

#### 常見名詞
1. Docker <br>
```
將應用程式自動化部署為可攜式且可自足的容器，在雲端或內部部署上執行。
``` 
2. Dockerfile <br>
```
是用來描述 Image 的文件
```
3. Image <br>
```
俗稱鏡像或映像檔，是由 Dockerfile build 起來之後產生的
``` 
4. Container <br>
```
Container 是根據 Image 所建立的 Instance (實例)。 
``` 
``` 
Container 可以被啟動、開始、停止、刪除。每個容器都是相互隔離、保證安全的平台。
``` 


### Docker 安裝

下載點 : [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)


### Docker Compose

我們在開發一個專案時，有可能需要 Server、Database、Cache，等多個 Service 才能構成一個可以上線運行的專案，這些 Service 往往會需要多個 Container 來運行。此時若是使用 Docker Compose 進行整合就不需要輸入大量指令去建立這些 Container

### Docker 範例說明

Github : [https://github.com/sprintcube/docker-compose-lamp/tree/7.4.x](https://github.com/sprintcube/docker-compose-lamp/tree/7.4.x)

打開專案的 `docker-compose.yml` 檔案 <br>
`#` 字號後方為說明

```
version: "3" # Docker Compose 目前版本，越新功能越多 (可參考：https://docs.docker.com/compose/compose-file/compose-versioning/#version-3)
services: # 準備建置的服務
  webserver: # 命名
    build: 
      context: ./bin/webserver # 這個服務將由此目錄下的 Dockerfile 建置
    container_name: '7.4.x-webserver'
    restart: 'always' # Container 因意外而中斷時會自動重啟
    ports: # 本地的 PORT : Container 中的 PORT 互相對應
      - "${HOST_MACHINE_UNSECURE_HOST_PORT}:80" # $ 字號開頭變數可在 sample.env 中看到，但實際上需改為 .env 才會生效
      - "${HOST_MACHINE_SECURE_HOST_PORT}:443"
    links: # 讓服務可連線
      - mysql 
    volumes: # 本地硬碟路徑對應到 Container 中的路徑
      - ${DOCUMENT_ROOT-./www}:/var/www/html
      - ${PHP_INI-./config/php/php.ini}:/usr/local/etc/php/php.ini
      - ${VHOSTS_DIR-./config/vhosts}:/etc/apache2/sites-enabled
      - ${LOG_DIR-./logs/apache2}:/var/log/apache2
  mysql: 
    build:
      context: "./bin/${DATABASE}"
    container_name: 'mysql'
    restart: 'always'
    ports:
      - "${HOST_MACHINE_MYSQL_PORT}:3306"
    volumes: 
      - ${MYSQL_DATA_DIR-./data/mysql}:/var/lib/mysql
      - ${MYSQL_LOG_DIR-./logs/mysql}:/var/log/mysql
    environment: # 原本 docker run 可以輸入的環境變數 (可參考：https://hub.docker.com/_/mysql)
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
  phpmyadmin:
    image: phpmyadmin/phpmyadmin # 用 Docker Hub 現成的 image (https://hub.docker.com/r/phpmyadmin/phpmyadmin)
    container_name: 'sc-phpmyadmin'
    links:
      - mysql
    environment:
      PMA_HOST: mysql
      PMA_PORT: 3306
      PMA_USER: ${MYSQL_USER}
      PMA_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    ports:
      - '8080:80'
    volumes: 
      - /sessions # Docker 自己幫你存
  redis:
    container_name: 'sc-redis'
    image: redis:latest
    ports:
      - "${HOST_MACHINE_REDIS_PORT}:6379"

```

備註：<br>自己寫 Dockerfile 是一件吃力不討好的事情，你寫的一定比別人爛，只有拿別人的來改，才會變成適合你自己的。<br>踩在巨人的肩膀上就是爽。

### 實際操作影片

1. 刪除本來有的 Image and Container
2. 開始 Build and Run Instance
3. 範例結果展示

### 實作加入自己的專案

踩在巨人的肩膀上，把別人的東西變成自己的東西

1. 把 `www` 資料夾變成自己的專案 `src` ( 這邊我放入整個 Laravel 專案
2. 調整 Apache 的根目錄 `config/vhosts/default.conf` 為 

	```
	<VirtualHost *:80>
	    ServerAdmin webmaster@localhost
	    DocumentRoot "/var/www/html/public"
	    ServerName localhost
		<Directory "/var/www/html/public/">
			AllowOverride all
		</Directory>
	</VirtualHost>
	```
3. `.env` 只調整 `DOCUMENT_ROOT=./src`
4. `src` 內的 Laravel 專案需 `chmod -R 777 bootstrap/ storage/`
5. 修改 `src` 內的 `.env` 裡面的 DB 連線資訊如下

	```	
	DB_CONNECTION=mysql
	DB_HOST=mysql
	DB_PORT=3306
	DB_DATABASE=docker
	DB_USERNAME=root
	DB_PASSWORD=tiger
	```
	重點： DB_HOST 不可填寫 127.0.0.1，因為 docker-compose.yml 已經有 Link，所以直接填寫 Container 的名稱，填寫 127.0.0.1 只會自己連自己，但是自己本身又沒有 MYSQL 就會出錯
	
6. 以上步驟做完後，就可以開始來 Build and Run Instance，請看影片
	1. 改版後的 Build and Run Instance
	2. 改版後的結果展示
