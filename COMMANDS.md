# 📋 Command Reference - WA Finance Tracker

## 🐳 Docker Commands

```powershell
# Jalankan semua container
docker compose up -d

# Matikan semua container
docker compose down

# Cek status container
docker compose ps

# Restart container tertentu
docker compose restart ollama
docker compose restart waha
docker compose restart n8n

# Restart + apply perubahan env
docker compose up -d --force-recreate waha

# Lihat log container
docker logs waha --tail 20
docker logs ollama --tail 20
docker logs n8n --tail 20

# Lihat log realtime
docker logs waha -f

# Cek environment variable dalam container
docker exec waha env | grep WHATSAPP
docker exec waha env | grep WAHA
```

---

## 🤖 Ollama Commands

```powershell
# Download model
docker exec -it ollama ollama pull qwen2.5:3b

# Lihat model yang terinstall
docker exec -it ollama ollama list

# Hapus model
docker exec -it ollama ollama rm qwen2.5:3b
```

---

## 📱 WAHA Session Commands

```powershell
# Cek status session
Invoke-RestMethod -Method GET -Uri "http://localhost:3000/api/sessions/default" -Headers @{"X-Api-Key"="finance123"}

# Start session
Invoke-RestMethod -Method POST -Uri "http://localhost:3000/api/sessions/default/start" -Headers @{"X-Api-Key"="finance123"}

# Stop session
Invoke-RestMethod -Method POST -Uri "http://localhost:3000/api/sessions/default/stop" -Headers @{"X-Api-Key"="finance123"}

# Hapus session
Invoke-RestMethod -Method DELETE -Uri "http://localhost:3000/api/sessions/default" -Headers @{"X-Api-Key"="finance123"}

# Buat session baru via API
Invoke-RestMethod -Method POST -Uri "http://localhost:3000/api/sessions" -Headers @{"Content-Type"="application/json"; "X-Api-Key"="finance123"} -Body '{"name":"default"}'

# Cek versi WAHA
Invoke-RestMethod -Method GET -Uri "http://localhost:3000/api/server/version" -Headers @{"X-Api-Key"="finance123"}
```

---

## 🔗 WAHA Webhook Commands

```powershell
# Daftarkan webhook (ganti WEBHOOK_ID)
$body = @{
  webhooks = @(
    @{ url = "http://n8n:5678/webhook/WEBHOOK_ID/waha"; events = @("message") }
  )
} | ConvertTo-Json -Depth 5

Invoke-RestMethod -Method PUT -Uri "http://localhost:3000/api/sessions/default" -Headers @{"Content-Type"="application/json"; "X-Api-Key"="finance123"} -Body $body

# Test kirim pesan manual ke webhook n8n
Invoke-RestMethod -Method POST -Uri "http://localhost:5678/webhook/WEBHOOK_ID/waha" -Headers @{"Content-Type"="application/json"} -Body '{"event":"message","payload":{"body":"test 123","from":"628123456789@c.us","fromMe":false,"type":"chat"}}'

# Test kirim pesan via WAHA (ganti chatId)
Invoke-RestMethod -Method POST -Uri "http://localhost:3000/api/sendText" -Headers @{"Content-Type"="application/json"; "X-Api-Key"="finance123"} -Body '{"chatId":"628xxx@c.us","text":"test balas","session":"default"}'
```

---

## 📊 WAHA Data Commands

```powershell
# Lihat semua chat
Invoke-RestMethod -Method GET -Uri "http://localhost:3000/api/default/chats" -Headers @{"X-Api-Key"="finance123"} | ConvertTo-Json -Depth 2

# Lihat pesan dari chat tertentu
Invoke-RestMethod -Method GET -Uri "http://localhost:3000/api/default/chats/CHATID/messages?limit=10" -Headers @{"X-Api-Key"="finance123"} | ConvertTo-Json -Depth 2
```

---

## 🌐 URL Penting

| Service | URL |
|---|---|
| WAHA Dashboard | http://admin:admin123@localhost:3000/dashboard |
| WAHA API Swagger | http://localhost:3000/api/ |
| WAHA QR Code | http://localhost:3000/api/sessions/default/auth/qr?x-api-key=finance123 |
| n8n Editor | http://localhost:5678 |
| Ollama API | http://localhost:11434 |

---

## 🔑 Credentials Default

| Service | Username | Password / Key |
|---|---|---|
| WAHA Dashboard | admin | admin123 |
| WAHA API Key | — | finance123 |
| n8n | admin | admin123 |
| Redis | — | redis123 |

---

## 💬 Perintah WA ke Bot

| Perintah | Fungsi |
|---|---|
| `laporan hari ini` | Laporan transaksi hari ini |
| `laporan minggu ini` | Laporan transaksi minggu ini |
| `laporan bulan ini` | Laporan transaksi bulan ini |
| `laporan juni 2026` | Laporan bulan spesifik |
| `saldo hari ini` | Cek saldo hari ini |
| `saldo bulan ini` | Cek saldo bulan ini |
| `hapus terakhir` | Hapus transaksi terakhir |
| *(teks bebas)* | Catat transaksi |

---

## 📁 Git Commands

```powershell
# Clone repository
git clone https://github.com/kirfansyah/wa-finance-tracker.git

# Push perubahan
git add .
git commit -m "pesan commit"
git push

# Pull perubahan terbaru
git pull origin main
```
