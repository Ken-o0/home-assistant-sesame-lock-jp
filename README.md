# 📦 Maintained Fork Notice / メンテナンス継続フォークのお知らせ

> **English**: The original repository ([thematrixdev/home-assistant-sesame-lock-jp](https://github.com/thematrixdev/home-assistant-sesame-lock-jp)) was archived on 2024-12-24 and is now read-only.
> This fork keeps the integration working with modern Home Assistant releases.
>
> **日本語**: 本家リポジトリ ([thematrixdev/home-assistant-sesame-lock-jp](https://github.com/thematrixdev/home-assistant-sesame-lock-jp)) は 2024年12月24日にアーカイブされ、読み取り専用になりました。
> このフォークでは、最新の Home Assistant で動作するよう修正を加えています。

## ✅ What's Fixed / 修正内容

**English**

1. **Added `version` key to `manifest.json`** — Required by Home Assistant since 2021.6. Without it, the integration is blocked from loading with the error `"The custom integration 'sesame_jp' does not have a version key in the manifest file and was blocked from loading."`
2. **Added `unique_id` to the lock entity** — Enables UI-based area assignment, renaming, and customization. Fixes the warning `"This entity does not have a unique ID, therefore its settings cannot be managed from the UI."`

**日本語**

1. **`manifest.json` に `version` キーを追加** — HA 2021.6 以降は必須。無いと `"The custom integration 'sesame_jp' does not have a version key in the manifest file and was blocked from loading."` エラーで読み込みがブロックされます。
2. **ロックエンティティに `unique_id` を追加** — UIからエリア割当・名前変更・カスタマイズが可能になります。「このエンティティにはユニークなIDがないため、UIから設定を管理構成できません」という警告が解消されます。

## 🔌 Tested Devices / 動作確認済みデバイス

- **SESAME 5** ✅ (Home Assistant 2026.4 / Docker)
- SESAME 3 / 4 — original integration already supported these; no regression expected

> The CANDY HOUSE Web API v3 endpoint (`app.candyhouse.co/api/sesame2/{uuid}`) is unified across SESAME 3, 4, and 5, so the same integration works for all generations.
>
> CANDY HOUSE Web API v3 (`app.candyhouse.co/api/sesame2/{uuid}`) は SESAME 3 / 4 / 5 で共通のため、世代を問わず動作します。

## 📥 Installation via HACS (Custom Repository) / HACSによるインストール

**English**

1. In Home Assistant, open **HACS** → **Integrations** → the `⋮` menu (top right) → **Custom repositories**
2. Add this repository URL:


# Sesame-Lock JP-Version Home-Assistant Custom-Component

If you have purchased your Sesame-Lock outside Japan, please use this:

https://www.home-assistant.io/integrations/sesame/

## Prerequisite
- Sesame Lock (for sure)
- Sesame WIFI module
- A pin (to reset the Lock)
- Android Studio
- An Android phone (emulator may work?)

## Setup Sesame Lock
- Install and power-on both Sesame-Lock and Sesame-WIFI-module
- Install the app and setup both the Lock and the WIFI-module
https://play.google.com/store/apps/details?id=co.candyhouse.sesame2
- In the app, sign-up a Sesame account

## Accquire Sesame API-Key
- https://partners.candyhouse.co/login

## Accquire Sesame Secret (private-key)
Please follow either one of the below method
- A. QR-Code-method
- B. Dev-App-method

### A. QR-Code-method
- Open Sesame-App on your phone
- Select the Lock, and choose properties
- "Share" the lock and get the QR-code
- Use "manager" or "owner" key

#### A1. Use any QR-Code Reader
- Scan and get the value of `sk`
- `base64` decode `sk`
- from 1st to 16th byte is the private-key

#### A2. Use an online Sesame-QR-Code Reader
- The QR-code-reader is not made by me. I bear no responsibility!
- https://sesame-qr-reader.vercel.app/
- Upload the QR-code image and get the data

### B. Dev-App-method
- Uninstall the Sesame-App from your phone
- Get Sesame Android app SDK (v2.0.7 at the time of writing this readme)
```
cd ~/
git clone https://github.com/CANDY-HOUSE/SesameSDK_Android_with_DemoApp.git
```
- Open the project with Android Studio
- Locate `app/java/co/candyhouse.app/App.kt`
- Place your `API-KEY` in `onCreate()`
```
class CandyHouseApp : Application() {
    override fun onCreate() {
        // original code here
        
        // MAKE SURE THIS LINE EXIST
        CHConfiguration.API_KEY = "X"
    }
}
```
- Locate any method with `device` and print out the Secret
. For example, `app/java/co/candyhouse.app/tabs/devices/model/CHDeviceViewModel.kt`
```
override fun onBleDeviceStatusChanged(device: CHSesameLocker, status: CHSesame2Status, shadowStatus: CHSesame2ShadowStatus?) {
    // original code here
    
    if (device.deviceStatus == CHSesame2Status.ReceivedBle) {
        device.connect {
            Log.d("sesame2SecretKey", device.getKey().secretKey)
        }
    }
}
```
- Connect your Android phone to Android Studio. Run the app
- Use the pin (the physical pin) to reset the Lock and the WIFI-Module
- Add the Lock and WIFI-Module in the app
- In Android Studio `Logcat`, search `sesame2SecretKey`. This is the Sesame-Secret
- Resetting the lock WILL RESET the secret!

## Accquire Sesame Lock UUID
- In the Android app, click on the Lock and locate it in the property page

## Install Home-Assistant Component
- Download `custom_components/sesame_jp` here
- SSH or SFTP into your Home-Assistant server
- Locate `configuration.yaml`
- Locate `custom_components` in the same directory. Create it if not exist 
- Put the downloaded `sesame_jp` into `custom_components`
- Restart Home-Assistant

## Configure Sesame-Lock in Home-Assistant
- Add these in `configuration.yaml`
```
lock:
  - platform: sesame_jp
    name: Sesame
    device_id: 'SESAME_UUID'
    api_key: 'SESAME_API_KEY'
    client_secret: 'SESAME_SECRET'
```
- Restart Home-Assistant

## Usage
- `Lock: Lock`
- `Lock: Unlock`

## Development environment
- Ubuntu 21.10
- Android 12 phone

## Testing environment
- Raspberry Pi 4B
- Raspbian Buster
- Home Assistant Container 2022.3.1 (i.e. Docker version)
- Sesame Lock 4
- Sesame WIFI-Module (For Sesame Lock 3 and 4)

## Resource
- Sesame Lock 4 https://jp.candyhouse.co/products/sesame4
- Sesame WIFI Module https://jp.candyhouse.co/products/new-wifi
- Sesame Web API https://doc.candyhouse.co/ja/SesameAPI
- Sesame Android SDK https://github.com/CANDY-HOUSE/SesameSDK_Android_with_DemoApp
- Obtain private-key from QR-Code https://qiita.com/kaonaga9/items/fb44d8e0b0aa93aab484
- Obtain private-key from QR-Code https://zenn.dev/key3/articles/6c1c2841d7a8a2

## Thanks
- Home Assistant community on Discord https://www.home-assistant.io/help/
- Google and Stackoverflow


3. カテゴリ：**統合**
4. **追加**をクリック後、HACS内で「Sesame Smart Lock JP」を検索してインストール
5. Home Assistant を再起動
6. `configuration.yaml` に設定を追記（下記の本家READMEを参照）

## 🙏 Credits / クレジット

- Original integration by [@thematrixdev](https://github.com/thematrixdev) — Thank you for the great work!
- CANDY HOUSE Web API documentation: https://doc.candyhouse.co/

---

# Original README Below / 以下は本家READMEの内容
