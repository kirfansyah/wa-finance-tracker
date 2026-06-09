# 📖 Panduan Setup WA Finance Tracker di Windows (Laptop Baru)

## Prerequisites

Pastikan sudah terinstall:

- **Docker Desktop** — https://www.docker.com/products/docker-desktop/
- **Git** — https://git-scm.com/download/win
- **VS Code** (opsional tapi recommended)

---

## STEP 1 — Clone Repository

Buka PowerShell atau Terminal, jalankan:

```powershell
cd D:\Project  # atau folder mana saja yang kamu mau
git clone https://github.com/kirfansyah/wa-finance-tracker.git
cd wa-finance-tracker
```

---

## STEP 2 — Setup File .env

Copy template dan isi sesuai kebutuhan:

```powershell
copy .env.example .env
```

Buka `.env` dengan VS Code dan isi:

```env
# WAHA
WAHA_API_KEY=finance123
WAHA_DASHBOARD_USERNAME=admin
WAHA_DASHBOARD_PASSWORD=admin123
WHATSAPP_HOOK_URL=http://n8n:5678/webhook/GANTI_NANTI/waha
WHATSAPP_HOOK_EVENTS=message
WHATSAPP_START_SESSION=default

# n8n
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=admin123
N8N_ENCRYPTION_KEY=supersecretkey123456
WEBHOOK_URL=http://localhost:5678

# Redis
REDIS_PASSWORD=redis123

# Ollama
OLLAMA_MODEL=qwen2.5:3b
```

> ⚠️ `WHATSAPP_HOOK_URL` akan diupdate nanti setelah dapat webhook URL dari n8n

---

## STEP 3 — Jalankan Docker

```powershell
docker compose up -d
```

Tunggu semua image selesai di-pull (~5-10 menit tergantung koneksi).

Cek semua container running:

```powershell
docker compose ps
```

Pastikan semua status **Up**:

```
NAME      STATUS
n8n       Up
ollama    Up
redis     Up
waha      Up
```

---

## STEP 4 — Pull Model Ollama

```powershell
docker exec -it ollama ollama pull qwen2.5:3b
```

> ⏳ Download ~2GB, tunggu sampai muncul "success"

Verifikasi model sudah ada:

```powershell
docker exec -it ollama ollama list
```

---

## STEP 5 — Setup n8n

### 5.1 Buka n8n

Buka browser → http://localhost:5678

### 5.2 Daftar Akun

Isi form registrasi:

- Email: bebas (contoh: `admin@finance.local`)
- First Name: nama kamu
- Password: minimal 8 karakter, ada angka & huruf kapital

### 5.3 Install Community Node WAHA

- Klik **Settings** (pojok kiri bawah) → **Community nodes**
- Klik **Install**
- Ketik: `n8n-nodes-waha`
- Centang "I understand..." → **Install**
- Tunggu selesai

### 5.4 Import Workflow

- Klik **+** di sidebar → **New workflow**
- Klik **...** (titik tiga) di pojok kanan atas
- Pilih **Import from JSON**
- Upload file `finance-tracker-final-v2.json`
- Klik **Import**

### 5.5 Setup Google Sheets Credential

- Klik node **Simpan ke Sheets**
- Di field **Credential** → klik **+ Add new credential**
- Pilih **Google Sheets OAuth2 API**
- Isi **Client ID** dan **Client Secret** dari Google Cloud Console
  - Buka https://console.cloud.google.com
  - Project: `wa-finance-tracker`
  - Menu: **OAuth** → **Clients** → klik `n8n-finance`
  - Copy **Client ID**
  - Untuk **Client Secret**: klik **+ Add secret** → copy secret baru yang muncul
- Klik **Sign in with Google** → login dengan akun Google yang sama
- Klik **Save**

### 5.6 Catat Webhook URL

- Klik node **WAHA Trigger1**
- Klik tab **"Production URL"**
- Copy URL-nya, contoh:
  ```
  http://localhost:5678/webhook/02a091c5-xxxx-xxxx/waha
  ```
- Ganti bagian host dengan `n8n`:
  ```
  http://n8n:5678/webhook/02a091c5-xxxx-xxxx/waha
  ```
- Simpan URL ini — akan dipakai di Step 7

### 5.7 Publish Workflow

- Klik **Ctrl+S** untuk save
- Klik tombol **Publish** di pojok kanan atas

---

## STEP 6 — Update .env dengan Webhook URL

Buka file `.env`, update baris `WHATSAPP_HOOK_URL`:

```env
WHATSAPP_HOOK_URL=http://n8n:5678/webhook/02a091c5-xxxx-xxxx/waha
```

Ganti `02a091c5-xxxx-xxxx` dengan ID webhook dari Step 5.6.

Lalu restart WAHA:

```powershell
docker compose up -d --force-recreate waha
```

---

## STEP 7 — Setup WAHA & Scan QR

### 7.1 Buka Dashboard WAHA

Buka browser → http://admin:admin123@localhost:3000/dashboard

> 💡 **Tips:** Gunakan format `http://username:password@localhost:3000/dashboard` agar login otomatis tanpa popup!

Login:

- Username: `admin`
- Password: `admin123`

### 7.2 Connect ke Server

- Di bagian **Workers**, klik ikon **✏️ (edit)** di baris WAHA
- Isi:
  - API URL: `http://localhost:3000`
  - API Key: `finance123`
- Klik **Save**
- Klik ikon **🔗 (connect)**

### 7.3 Start Session & Scan QR

- Di bagian **Sessions**, klik **Start New**
- Isi Session name: `default`
- Klik **Start**
- Scan QR Code yang muncul dengan WhatsApp di HP

Tunggu status berubah menjadi **WORKING** ✅

### 7.4 Daftarkan Webhook

Jalankan di PowerShell (ganti URL dengan webhook kamu):

```powershell
Invoke-RestMethod -Method POST -Uri "http://localhost:3000/api/sessions/default/start" -Headers @{"X-Api-Key"="finance123"}
```

Tunggu ~20 detik lalu cek:

```powershell
docker logs waha --tail 10
```

Pastikan ada baris:

```
Configuring webhooks for http://n8n:5678/webhook/xxxx/waha
Webhooks were configured
```

---

## STEP 8 — Test!

Kirim pesan dari HP ke nomor bot:

```
nasi goreng 15rb
kopi 8k
gaji 5jt
bensin 50rb
```

Tunggu ~1-2 menit (Ollama memproses di CPU).

Cek:

- ✅ n8n Executions → ada execution Succeeded
- ✅ Google Sheet → data masuk rapi
- ✅ WhatsApp → bot membalas konfirmasi

---

## 🔧 Troubleshooting

### Docker tidak jalan

```powershell
# Pastikan Docker Desktop sudah running
# Cek status container
docker compose ps

# Lihat log error
docker compose logs
```

### Ollama timeout / connection aborted

```powershell
# Restart Ollama
docker compose restart ollama

# Cek apakah model ada
docker exec -it ollama ollama list
```

### WAHA session disconnect

```powershell
# Start ulang session
Invoke-RestMethod -Method POST -Uri "http://localhost:3000/api/sessions/default/start" -Headers @{"X-Api-Key"="finance123"}
```

### Webhook tidak terdeteksi

```powershell
# Cek log WAHA
docker logs waha --tail 20

# Re-register webhook (ganti URL)
Invoke-RestMethod -Method PUT -Uri "http://localhost:3000/api/sessions/default" -Headers @{"Content-Type"="application/json"; "X-Api-Key"="finance123"} -Body '{"webhooks":[{"url":"http://n8n:5678/webhook/WEBHOOK_ID/waha","events":["message"]}]}'
```

### Google Sheets error Forbidden

- Pastikan Google Sheet sudah di-share ke email akun Google yang login di n8n
- Pastikan Google Sheets API dan Google Drive API sudah di-enable di Google Cloud Console project yang benar

### n8n tidak bisa buka

```powershell
# Cek apakah port 5678 sudah dipakai
netstat -ano | findstr :5678

# Restart n8n
docker compose restart n8n
```

---

## 📋 Checklist Setup

- [ ] Docker Desktop running
- [ ] `git clone` berhasil
- [ ] `.env` sudah diisi
- [ ] `docker compose up -d` → semua container Up
- [ ] Ollama model `qwen2.5:3b` sudah di-pull
- [ ] n8n akun sudah dibuat
- [ ] Community node `n8n-nodes-waha` sudah diinstall
- [ ] Workflow sudah di-import
- [ ] Google Sheets credential sudah di-setup
- [ ] Workflow sudah di-Publish
- [ ] Webhook URL sudah di-update di `.env`
- [ ] WAHA container sudah di-restart
- [ ] Session WAHA sudah WORKING (scan QR)
- [ ] Test kirim pesan WA berhasil

---

## 📌 URL & Credentials Penting

| Service        | URL                                            | Login              |
| -------------- | ---------------------------------------------- | ------------------ |
| n8n            | http://localhost:5678                          | akun yang didaftar |
| WAHA Dashboard | http://admin:admin123@localhost:3000/dashboard | admin / admin123   |
| Ollama API     | http://localhost:11434                         | —                  |
| Redis          | localhost:6379                                 | password: redis123 |

**Google Sheets Spreadsheet ID:**

```
1OndrHyft1KHYNTr3eNtERhzd4KoPfa94BDftobvZ5t8
```
