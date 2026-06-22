<!-- Header -->
![Blueprint Docker](https://github.com/BlueprintFramework/docker/assets/103201875/f1c39e6e-afb0-4e24-abd3-508ec883d66b)
<p align="center">你所熟知和喜愛的 <a href="https://github.com/BlueprintFramework/main"><b>Blueprint</b></a> 擴充生態系統，現在支援 🐳 Docker。</p>

<!-- Information -->
<br/><h2 align="center">🐳 Docker 版 Blueprint</h2>

### 支援的架構
| 架構          | 支援狀態 |
|--------------|---------|
| AMD64        | :white_check_mark: 已支援   |
| ARM64        | :white_check_mark: 已支援   |

- Things you should be aware of before reading this guide:
  - 雖然提供的面板和 Wings 映像檔可以在 Arm64 上正常運行，但大多數遊戲伺服器 _無法_ 運行，因此如果你在 Arm64 機器上運行 Wings，需要注意這一點。
  - In all of these examples, ``/srv/pterodactyl`` is used as the base directory. If you set a different value for BASE_DIR in your .env file, use that path instead.
- 如果在 Raspberry Pi 上運行 Wings，請參閱 quintenqvd 在 Pterodactyl Discord 中發布的以下內容：
  > 在 Pi 4 或 5 上運行 Wings
  > Wings 需要 docker cgroups。這些在 Ubuntu 版本中不存在，只存在於 Debian 11 或 12 中
  > 安裝 Debian Lite 64 位元作業系統
  > 安裝 Docker 後開啟 /boot/cmdline.txt 檔案並在已有內容的末尾新增（不要刪除任何內容且不要換行）cgroup_memory=1 cgroup_enable=memory systemd.unified_cgroup_hierarchy=0，儲存並退出，然後重新啟動
  > 注意：在基於 Debian 12 的作業系統中，路徑是 /boot/firmware/cmdline.txt

### docker-compose.yml 和 classic-docker-compose.yml 有什麼區別？
- classic-docker-compose.yml 盡可能保持與原始 Pterodactyl compose 檔案接近
  - 這意味著它仍然有過時的 "version" 屬性，沒有健康檢查，也不使用 .env 檔案進行設定
  - 這個檔案看起來更簡單、更容易理解，主要是因為它不會像推薦的 docker-compose.yml 檔案那樣提供相同等級的控制和資訊
- docker-compose.yml（推薦）可以並且已經隨著時間的推移而改進
  - 如果你使用此版本，請下載並設定 .env 檔案；大部分（如果不是全部）設定都可以透過 .env 檔案完成

### 這是你第一次在 Docker 中運行 Wings 嗎？
- 需要準備的一點是，Wings 透過掛載的 socket 使用主機系統的 Docker Engine；它不使用 Docker in Docker。
- 這意味著如果你想自訂儲存資料的目錄，必須在 mounts 中為主機和容器設定相同的值，然後你必須使你的 config.yml 中的值相符；否則 Wings 容器會看到一個目錄，然後當建立一個不受此 docker-compose.yml 的 mounts 影響的新容器時，它不會看到相同的目錄。這裡有一個範例：
  - docker-compose.yml 中的掛載：``"${BASE_DIR}/:${BASE_DIR}/"``
  - 假設在此範例中，你在 .env 檔案中將 ``BASE_DIR`` 設定為 **/srv/pterodactyl**。如果你想將 Wings 伺服器資料掛載到其他位置，只需新增任何其他掛載，確保掛載的兩側相符。
  - 現在當你建立節點時，你會為 **Daemon Server File Directory** 選擇你所建立掛載內的某個位置，例如 /srv/pterodactyl/wings/servers
  - Wings 首次成功運行後，你的 **config.yml** 檔案中將會出現更多選項。它們看起來像這樣：
  - ```
    root_directory: /var/lib/pterodactyl
    log_directory: /var/log/pterodactyl
    data: /srv/pterodactyl/wings/servers
    archive_directory: /var/lib/pterodactyl/archives
    backup_directory: /var/lib/pterodactyl/backups
    tmp_directory: /tmp/pterodactyl
    ```
  - 如你所見，只有 **data** 會被設定為你設定的位置。你可以透過將 **/var/lib/pterodactyl** 更改為與你的基礎目錄相符來使其他目錄相符，在此範例中再次為 **/srv/pterodactyl**。你也可以選擇性地更改日誌位置，如果你想盡可能將***所有內容***保持在一個目錄中，這是使用容器的好處之一。完成後，它可能看起來像：
  - ```
    root_directory: /srv/pterodactyl
    log_directory: /srv/pterodactyl/wings/logs
    data: /srv/pterodactyl/wings/servers
    archive_directory: /srv/pterodactyl/archives
    backup_directory: /srv/pterodactyl/backups
    tmp_directory: /tmp/pterodactyl
    ```
### 建立你的第一個使用者
- ``cd`` 進入包含 compose 檔案的目錄，例如 ``cd /srv/pterodactyl``
- ```bash
  docker compose exec panel php artisan p:user:make
  ```

### 上傳擴充套件
擴充套件必須放置/拖曳到 `extensions` 資料夾中。

### 與 Blueprint 互動
預設情況下，你只能透過 Docker Engine 命令列與 Blueprint 互動，即：
```bash
docker compose exec panel blueprint (參數)
```

#### 我們建議設定一個別名，這樣你就可以像在非 Docker 版本中一樣與 Blueprint 互動（如果你的 compose 檔案在不同的地方，請相應調整）：
```bash
# 為當前工作階段設定別名
alias blueprint="docker compose -f /srv/pterodactyl/docker-compose.yml exec panel blueprint"
# 附加到你的 .bashrc 檔案末尾以使其持久化
echo 'alias blueprint="docker compose -f /srv/pterodactyl/docker-compose.yml exec panel blueprint"' >> ~/.bashrc
```

### 安裝擴充套件範例
這是一個快速範例，展示了如何在 Docker 版本的 Blueprint 上安裝擴充套件。請注意，每個擴充套件的體驗可能有所不同。
  1. [尋找擴充套件](https://blueprint.zip/browse)，你想安裝並尋找具有 `.blueprint` 副檔名的檔案。
  2. 將 `example.blueprint` 檔案拖曳/上傳到你的 extensions 資料夾，即預設情況下為 `/srv/pterodactyl/extensions`。
  3. 透過 Blueprint 命令列工具安裝擴充套件：
     ```bash
     docker compose exec panel blueprint -i example
     ```
     或者，如果你已經套用了我們上面建議的別名：
     ```bash
     blueprint -i example
     ```

#### 所以，你安裝了你的第一個擴充套件。恭喜！Blueprint 現在正在 `pterodactyl_app` 卷中保留持久資料，因此你需要開始定期備份該卷。

### 首先，我們將安裝 Restic 來處理備份
為什麼選擇 Restic？壓縮、去重複和增量備份。與每次簡單地歸檔目錄相比，可以節省空間。
套件名稱通常是 `restic`，例如：
| 作業系統                          | 指令                                                            |
|----------------------------------|-----------------------------------------------------------------|
| Ubuntu / Debian / Linux Mint     | `sudo apt -y install restic`                                    |
| Fedora                           | `sudo dnf -y install restic`                                    |
| Rocky Linux / AlmaLinux / CentOS | `sudo dnf -y install epel-release && sudo dnf -y install restic`|
| Arch Linux                       | `sudo pacman -S --noconfirm restic`                             |
| openSUSE                         | `sudo zypper install -n restic`                                 |
| Gentoo                           | `sudo emerge --ask=n app-backup/restic`                         |

#### 建立備份目錄和腳本
```bash
mkdir -p /srv/backups/pterodactyl
export RESTIC_PASSWORD="CHANGE_ME"
restic init --repo /srv/backups/pterodactyl
cat <<EOF > /srv/backups/backup.sh
#!/bin/bash
docker compose -f /srv/pterodactyl/docker-compose.yml down panel
cd /var/lib/docker/volumes/pterodactyl_app/_data
RESTIC_PASSWORD="${RESTIC_PASSWORD}" restic backup . -r /srv/backups/pterodactyl
docker compose -f /srv/pterodactyl/docker-compose.yml up -d panel
EOF
chmod +x /srv/backups/backup.sh
```

#### 設定 crontab 來備份你的面板（選擇最不可能被使用的時間）
```bash
(crontab -l 2>/dev/null; echo "59 23 * * * /srv/backups/backup.sh") | crontab -
```

#### 很好。我現在有每日備份了，並且設定為最多保留 30 個備份。我如何從其中一個還原？
你可以使用 ``restic snapshots --repo /srv/backups/pterodactyl`` 列出快照
你在尋找一個看起來像 ``46adb587`` 的 **ID** 值。**Time** 會在每個 ID 旁邊，這樣你就可以看到你的備份來自哪一天。

#### 確定要還原哪個快照後，停止你的 compose 堆疊，還原你的資料，然後再次啟動你的堆疊
```bash
docker compose -f /srv/pterodactyl/docker-compose.yml down
# 清除目錄以便還原乾淨
rm -rf /var/lib/docker/volumes/pterodactyl_app/_data/.[!.]* /var/lib/docker/volumes/pterodactyl_app/_data/*
# 記得將 "46adb587" 替換為你要還原的快照的實際 ID
restic restore 46adb587 -r /srv/backups/pterodactyl -t /var/lib/docker/volumes/pterodactyl_app/_data
docker compose -f /srv/pterodactyl/docker-compose.yml up -d
```
# Setting up a Development Environment
## If logged in as the root user
- ``ln -s /var/lib/docker/volumes/pterodactyl_app/_data /srv/pterodactyl/webroot``
## If logged in as a non-root user
- If you have already started the panel, stop it and clear the volume before starting: ``docker compose down -v panel``
- Make sure your filesystem supports ACL and that you have it installed. ``setfacl -v`` should give you an output.
- Set permissions (assumes you're logged in as the nonroot user you want to use): ``sudo setfacl -R -m u:$USER:rwx,d:u:$USER:rwx /srv/pterodactyl/``
- Create the webroot folder: ``mkdir -p /srv/pterodactyl/webroot``
- Replace the section at the bottom of your compose file,
```
volumes:
  app:
```
with this new section:
```
volumes:
  app:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: ${BASE_DIR}/webroot
```
- Bring the panel up: ``docker compose up -d``

# 在 Docker 中更新 Blueprint
- 記住，在更新之前一定要[建立備份](<https://github.com/BlueprintFramework/docker?tab=readme-ov-file#first-well-install-restic-to-handle-backups>)
## 選項 1：僅更新 Blueprint
- 如果你已經設定了我們之前建議的別名
  ```bash
  blueprint -upgrade
  ```
- 如果你沒有
  ```bash
  docker compose -f /srv/pterodactyl/docker-compose.yml exec panel blueprint -upgrade
  ```

## 選項 2：同時更新 Blueprint 和 Pterodactyl 面板
- 本指南假設個別擴充套件/主題作者已選擇將任何持久資料（例如設定）儲存在資料庫中。如果他們沒有這樣做...擴充套件資料沒有特定的儲存位置，因此資料可能在任何地方。你需要詢問他們是否有任何持久資料儲存在任何地方，在更新之前需要備份。
- 進入你的 docker-compose.yml 檔案所在的目錄
- ```bash
    docker compose down -v
  ```
- -v 告訴它刪除任何命名卷，即我們使用的 app 卷。它不會刪除 bind-mounts 中的資料。這樣新映像檔的 app 卷就可以取代舊的。
- 更改面板映像檔中的標籤（即從 **v1.11.5** 升級到 **v1.11.7**，你會將 ``ghcr.io/blueprintframework/blueprint:v1.11.5`` 更改為 ``ghcr.io/blueprintframework/blueprint:v1.11.7``）。
- ```bash
    docker compose pull
  ```
- ```bash
    docker compose up -d
  ```
- 最後，再次安裝你的擴充套件。你可以使用 ``blueprint -i *.blueprint`` 重新安裝 extensions 資料夾中的所有擴充套件。
- 如果在此步驟後任何擴充套件的設定消失了，請從備份還原並詢問這些擴充套件的作者持久資料儲存在哪裡，這樣你就可以在每次更新後備份並還原它。

## Option 3: Update both Blueprint and Pterodactyl Panel WHEN YOU HAVE SET UP THE NONROOT DEVELOPMENT ENVIRONMENT in the [previous section](https://github.com/BlueprintFramework/docker#setting-up-a-development-environment) of this README.
- This guide operates under the assumption that individual extension/theme authors have chosen to store any persistent data such as settings in the database. If they have not done this... there isn't any specific place extension data is meant to be stored, so the data could be anywhere. You'll need to ask them if there is any persistent data stored anywhere that you have to back up before updating.
- Go to the directory of your docker-compose.yml file
- ```bash
    docker compose down
  ```
- ```bash
    mv webroot webroot.bak_$(date +%m-%d-%Y)
  ```
- ```bash
    mkdir webroot
  ```
- ```bash
    docker compose down -v
  ```
- The -v tells it to delete any named volumes, i.e. the app volume we use. It will not delete data from bind-mounts. This way the new image's app volume can take place.
- Change the tag in your panel's image (i.e. to upgrade from **v1.11.5** to **v1.11.7**, you would change ``ghcr.io/blueprintframework/blueprint:v1.11.5`` to ``ghcr.io/blueprintframework/blueprint:v1.11.7``.
- ```bash
    docker compose pull
  ```
- ```bash
    docker compose up -d
  ```
- If you want anything from your previous dev enviornment, ``cp -r webroot.bak_$(date +%m-%d-%Y)/.blueprint/dev webroot/.blueprint/dev``
- Lastly, install your extensions again. You can reinstall all of the extensions in your extensions folder with ``blueprint -i *.blueprint``.
- If any of your extensions' settings are gone after this step, restore from your backup and ask the author of those extensions where persistent data is stored so you can back it up and restore it after each update.

<!-- copyright footer -->
<br/><br/>
<p align="center">
  © 2024-2026 Emma (prpl.wtf) and Loki
  <br/><br/><img src="https://github.com/user-attachments/assets/15aa92e8-cef3-420e-ae8e-d0cd83263925"/>
</p>
