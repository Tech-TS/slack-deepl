程式碼原作者：https://github.com/seratch/deepl-for-slack

修改以供繁體中文使用者使用

## DeepL 整合 Slack

DeepL 可以應用程式與 Slack 進行整合，使用者可在 Slack 上透過點擊 emoji 圖示，將對應的對話內容翻譯為不同語言，例如點擊日本國旗將會翻譯為日文。

## 功能

### 運行 DeepL 翻譯 API 的快捷方式

Slack 使用者可以在模態視窗中運行 DeepL 翻譯 API

<img src="https://user-images.githubusercontent.com/19658/84773721-cb505f80-b017-11ea-8c41-aed57012ab8b.gif" height="500">

### 在對話串中發布翻譯的文本

與[reacjilator](https://github.com/slackapi/reacjilator)大致相同。

<img src="https://user-images.githubusercontent.com/19658/84773773-dc996c00-b017-11ea-9022-017492a7c9df.gif" height="500">

## 前置條件

運行本應用程式需具備以下帳號：

- DeepL Pro（開發者專用）帳號
- Slack 工作空間與使用者帳號
- Heroku 帳號

若上述帳號皆具備，設定只需五分鐘。

若無 Heroku 帳號，可在一台伺服器上運行 Docker 環境並使用一網域指向該伺服器，並設置 Nginx 導向至本機上的 Docker container 內，需要條件：

- Linux 作業系統的伺服器
- 可以自己設定的網域

## 設定

### 建立您的 DeepL Pro（開發者專用）帳號

- 在 https://www.deepl.com/pro/ 選擇 for Developers 計劃（請注意不要選擇其他選項）
- 前往您的 [DeepL Pro Account](https://www.deepl.com/pro-account.html) 頁面
- 保存 **DeepL API 認證金鑰** 的值

有關更多詳情，請參考以下資源：

- https://www.deepl.com/en/pro/
- https://www.deepl.com/docs-api/

### 建立您的 Slack 應用程式

使用 [應用程式清單檔案](https://github.com/seratch/deepl-for-slack/blob/master/app-manifest.yml) 來配置新的應用程式！

<img width="400" src="https://user-images.githubusercontent.com/19658/121115984-cef47c00-c850-11eb-9d7e-dbd80407ac9a.png">
<img width="400" src="https://user-images.githubusercontent.com/19658/121115976-cc922200-c850-11eb-8e23-1054c48b54d0.png">
<img width="400" src="https://user-images.githubusercontent.com/19658/121115986-cf8d1280-c850-11eb-8f7f-9d59112df42b.png">
<img width="400" src="https://user-images.githubusercontent.com/19658/121115989-d025a900-c850-11eb-9cb7-35fc979a81f8.png">

- 點擊左側面板 **Settings > Install App**

  - 點擊 **Install App to Workspace** 按鈕
  - 於 OAuth 確認頁面點擊 **Allow** 按鈕
  - 保存 **Bot User OAuth Access Token** 值（xoxb-\*\*\*）

- 點擊左側面板 **Settings > Basic Information**
  - 選擇 **App Credentials** 區域
  - 對 **Signing Secret** 點擊 **Show** 按鈕
  - 保存 **Signing Secret** 值

### 佈署選擇一：Heroku（推薦）

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/seratch/deepl-for-slack/tree/master)

- 確認 [你的付費設定](https://dashboard.heroku.com/account/billing)
- 設定`app-manifest.yml`：
  - `event_subscriptions`與`interactivity`各自下面的`request_url`值為**https://Heroku 佈署後的網域/slack/events**
- 設定環境變數檔案`.env`以佈署：
  - `SLACK_SIGNING_SECRET`: 從 Slack app 設定頁面取得 **Settings > Basic Information > App Credentials > Signing Secret**
  - `SLACK_BOT_TOKEN`: 從 Slack app 設定頁面取得 **Settings > Install App > Bot User OAuth Access Token**
  - `DEEPL_AUTH_KEY`: 從 DeepL Pro 帳號設定頁面取得 **Authentication Key for DeepL API**
  - `DEEPL_FREE_API_PLAN`: 若為 DeepL API 免費計畫，設置為 1（Pro Plan 預設為 0）
- 可變更 Dyno Type ，啟用該應用程式

### 佈署選擇二：自建主機環境

- 選擇一台 Linux server
- 使用`git clone https://github.com/Tech-TS/slack-deepl.git`指令將程式碼下載至本伺服器
- 修改`.env`環境變數（參照佈署選擇一內容）與設定`app-manifest.yml`：
  - `event_subscriptions`與`interactivity`各自下面的`request_url`值為**https://<網域名稱>/slack/events**
- 設置自己的網域中的子網域 A 記錄指向本伺服器
- 使用 Dockerfile 生成 Docker container：`docker build -t deepl-flag-latest:latest .`
- 運行 container：`docker run -d --name deepl-container-latest -p 3000:3000 deepl-flag-latest`
- 設置 Nginx 將外部請求導入本機的 3000 port：`vim /etc/nginx/sites-enabled/deepl`，內容：

```
server {
    listen 80;
    server_name <網域名稱>;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

- 使用 certbot 指令`certbot certonly -d 網域名稱`，申請 SSL 憑證
- 改寫/etc/nginx/sites-enabled/deepl

```
server {
    listen 443 ssl;
    server_name <網域名稱>;

    ssl_certificate <certbot生成的憑證路徑>;
    ssl_certificate_key <certbot生成的憑證路徑>;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

- 重新啟動 Nginx：`systemctl restart nginx`

### Slack 方面設定

- 點擊面板左側 **Features > Event Subscriptions**

  - 設置請求網址 `https://<Heroku或私有網域>/slack/events`
  - 點擊 **Save Changes** 按鈕以保存設定

- 點擊面板左側 **Features > Interactivity & Shortcuts**
  - 設置請求網址 `https://<Heroku或私有網域>/slack/events`
  - 點擊 **Save Changes** 按鈕以保存設定

### 確認應用程式可用

- 在 Slack 頻道中輸入 `/invite @deepl_translation` 加入該該應用程式
- 發一篇貼文：`In functional programming, a monad is a design pattern that allows structuring programs generically while automating away boilerplate code needed by the program logic. Monads achieve this by providing their own data type (a particular type for each type of monad), which represents a specific form of computation, along with one procedure to wrap values of any basic type within the monad (yielding a monadic value) and another to compose functions that output monadic values (called monadic functions).`
- 在 Slack 頻道的訊息中點擊 `:flag-jp:` emoji
- 確認對話下應用程式將新增其譯文

### 授權條款

MIT 授權條款

### 相關專案

如果您正在尋找更多功能，請查看以下這些很棒的項目：

- https://github.com/monstar-lab-oss/deepl-for-slack-elixir
