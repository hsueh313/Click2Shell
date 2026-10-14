Click2Shell WordPress POC 弱點重現環境（打包日期 2026-10-14）

【警告】
  本環境僅供實驗室重現弱點使用，設計上即不安全：
  - 請只在隔離的內網執行，勿放到公網或公司網路。
  - 帳號密碼為公開的預設值（見下方「帳號資訊」）。
  - 8080（WordPress）與 3306（MySQL）對外開放。
    注意：Docker 開的 port 會繞過 ufw 防火牆，ufw 擋不住。

內容
  docker-compose.yml                   compose 設定（pull_policy: never，不會從網路拉映像）
  images/click2shell-images.tar.gz     映像：click2shell-wordpress:6.4.3（WordPress 6.4.3 / PHP 8.2.17 / Apache）、mysql:5.7
                                       檔案 393MB，不放在 git 內，請從本儲存庫的 Releases 下載
  theme/mobile-repair-zone.2.5.6.zip   主題離線備份（未安裝，需手動安裝）
  SHA256SUMS                           檔案雜湊值

需求：目標機器已安裝 Docker 與 docker compose。

取得映像
  從本儲存庫的 Releases 下載 click2shell-images.tar.gz，放到 images/ 目錄。

建立環境（在本目錄執行）
  sha256sum -c SHA256SUMS                          # 檢查檔案完整性，每行都要顯示 OK
  docker load -i images/click2shell-images.tar.gz  # 載入映像
  docker compose up -d                             # 啟動
  約 5 秒後，瀏覽器開啟 http://<IP>:8080/ 進行 WordPress 安裝（離線安裝時語言選單只有英文）。

安裝主題
  後台「外觀 → 佈景主題 → 安裝佈景主題 → 上傳佈景主題」，上傳 theme/mobile-repair-zone.2.5.6.zip。
  - 直接上傳原始 zip，不要解壓縮、不要改檔名或副檔名。
    WordPress 會檢查檔案內容，只把檔名改成 .zip 的檔案會被拒絕。
  - PHP 上傳上限已在 docker-compose.yml 調為 64M（預設 2M，主題 2.8MB 會上傳失敗）。

刪除環境（含資料）
  docker compose down -v

帳號資訊
  WordPress     http://<IP>:8080/            管理員帳號於安裝畫面自行設定
  MySQL         <IP>:3306                    資料庫 wordpress
                一般使用者 wordpress / wordpress
                root 密碼  rootpassword

說明
  - 全新 WordPress 6.4.3，空資料庫。
  - 已關閉自動更新（AUTOMATIC_UPDATER_DISABLED、WP_AUTO_UPDATE_CORE=false）。
  - 容器：click2shell-db-1、click2shell-wordpress-1；volume：click2shell_db_data、click2shell_wordpress_data。

第三方元件與授權
  本環境未修改以下元件，僅原樣重新散布：
  - WordPress 6.4.3         GPLv2 或更新版本   https://wordpress.org/
  - Mobile Repair Zone 2.5.6 GPLv3             https://wordpress.org/themes/mobile-repair-zone/
  - MySQL 5.7               GPLv2              原始碼：https://github.com/mysql/mysql-server/tree/5.7
  - 映像基底為 Docker 官方映像 wordpress:6.4.3-php8.2-apache、mysql:5.7，內含 Debian/Oracle Linux 套件，各依其授權。
