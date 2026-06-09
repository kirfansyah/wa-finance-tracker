# 💰 WA Finance Tracker

Aplikasi pencatatan keuangan pribadi berbasis WhatsApp yang menggunakan AI lokal untuk menganalisis dan mengkategorikan transaksi secara otomatis.

---

## ✨ Fitur

- 📱 **Input via WhatsApp** — Cukup kirim pesan teks biasa ke bot
- 🤖 **AI Parsing** — AI lokal (Ollama) memahami input bebas, typo, dan singkatan
- 🏷️ **Auto Kategorisasi** — Transaksi dikategorikan otomatis (Belanja Dapur, Transportasi, dll)
- 💸 **Pengeluaran & Pemasukan** — Support kedua jenis transaksi
- 📊 **Google Sheets** — Data tersimpan rapi di spreadsheet
- ✅ **Konfirmasi Real-time** — Bot membalas ringkasan setiap input

---

## 🛠️ Tech Stack

| Komponen | Teknologi |
|---|---|
| WhatsApp Gateway | [WAHA](https://waha.devlike.pro/) (WhatsApp HTTP API) |
| Workflow Orchestrator | [n8n](https://n8n.io/) |
| AI Local | [Ollama](https://ollama.ai/) + `qwen2.5:3b` |
| Cache | Redis |
| Storage | Google Sheets |
| Infrastructure | Docker & Docker Compose |

---

## 📋 Cara Penggunaan

### Catat Pengeluaran
```
bawang merah 5rb
cabe ijo 3kg 200rb
nasi goreng 12k
bensin 50rb
```

### Catat Pemasukan
```
gaji 5jt
dapat arisan 2juta
bonus 500rb
```

### Format Harga yang Didukung
| Format | Nilai |
|---|---|
| `5rb` / `5k` | Rp 5.000 |
| `200rb` / `200k` | Rp 200.000 |
| `1jt` / `1juta` / `1m` | Rp 1.000.000 |
| `17jt` / `17juta` | Rp 17.000.000 |
| `2.5jt` | Rp 2.500.000 |

### Typo & Singkatan Otomatis Diperbaiki
```
bwng mrh → Bawang Merah
cb ijo   → Cabe Ijo
bnsin    → Bensin
ojol     → Ojek Online
aym grg  → Ayam Goreng
```

---

## 🏷️ Kategori Transaksi

### Pengeluaran
`Belanja Dapur` `Makan & Minum` `Transportasi` `Kesehatan` `Rumah & Utilitas` `Pakaian` `Pulsa & Internet` `Pendidikan` `Hiburan` `Cicilan & Hutang` `Anak & Keluarga` `Sosial` `Investasi & Tabungan` `Lain-lain`

### Pemasukan
`Gaji` `Arisan` `Bonus & THR` `Hutang Masuk` `Hasil Investasi` `Penjualan` `Transfer Masuk` `Lain-lain`

---

## 🚀 Instalasi

### Prerequisites
- Docker & Docker Compose
- Git
- Google Account (untuk Google Sheets)

### 1. Clone Repository
```bash
git clone https://github.com/kirfansyah/wa-finance-tracker.git
cd wa-finance-tracker
```

### 2. Setup Environment
```bash
cp .env.example .env
# Edit .env sesuai kebutuhan
```

### 3. Jalankan Docker
```bash
docker compose up -d
```

### 4. Pull Model Ollama
```bash
docker exec -it ollama ollama pull qwen2.5:3b
```

### 5. Setup Google Sheets
- Buat spreadsheet baru di Google Sheets
- Buat sheet dengan nama `Transaksi` dan kolom: `Tanggal`, `Nama Item`, `Jumlah/Satuan`, `Harga`, `Tipe (Pengeluaran/Pemasukan)`, `Kategori`, `Catatan`, `Timestamp`
- Buat Google Cloud Project dan enable Google Sheets API & Google Drive API
- Buat OAuth 2.0 credentials
- Catat Spreadsheet ID dari URL Google Sheets

### 6. Setup n8n
- Buka `http://localhost:5678`
- Import workflow dari file `finance-tracker-final.json`
- Setup Google Sheets credential
- Publish workflow

### 7. Setup WAHA
- Buka `http://localhost:3000/dashboard`
- Login dengan credentials dari `.env`
- Buat session `default`
- Scan QR Code dengan WhatsApp
- Daftarkan webhook ke n8n

---

## 📁 Struktur Project

```
wa-finance-tracker/
├── docker-compose.yml          # Docker services config
├── .env                        # Environment variables (tidak di-commit)
├── .env.example                # Template environment variables
├── n8n/
│   └── finance-tracker-final.json  # n8n workflow
└── README.md
```

---

## ⚙️ Services & Port

| Service | URL | Keterangan |
|---|---|---|
| WAHA Dashboard | http://localhost:3000/dashboard | WhatsApp Gateway |
| n8n | http://localhost:5678 | Workflow Editor |
| Ollama API | http://localhost:11434 | AI Local |
| Redis | localhost:6379 | Cache |

---

## 🔧 Troubleshooting

### Ollama lambat / timeout
- Normal untuk CPU only, estimasi 1-2 menit per request
- Upgrade ke GPU (NVIDIA CUDA atau AMD ROCm) untuk performa lebih baik
- Gunakan model lebih kecil jika OOM

### WAHA session disconnect
```bash
# Restart session
curl -X POST http://localhost:3000/api/sessions/default/start \
  -H "X-Api-Key: your_api_key"
```

### Webhook tidak terdeteksi
```bash
# Re-register webhook
curl -X PUT http://localhost:3000/api/sessions/default \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: your_api_key" \
  -d '{"webhooks":[{"url":"http://n8n:5678/webhook/your-id/waha","events":["message"]}]}'
```

---

## 📝 Contoh Output Bot

```
✅ Berhasil dicatat:

1. 🛒 Bawang Merah (-)
   Rp 5.000 | Belanja Dapur
2. 🛒 Cabe Ijo (3 kg)
   Rp 200.000 | Belanja Dapur
3. 🛒 Nasi Goreng (1 porsi)
   Rp 12.000 | Makan & Minum
4. 💰 Gaji (-)
   Rp 5.000.000 | Gaji

💸 Total pengeluaran: Rp 217.000
💵 Total pemasukan: Rp 5.000.000
```

---

## 🗺️ Roadmap

- [x] Input transaksi via WhatsApp
- [x] AI parsing dengan Ollama
- [x] Auto kategorisasi
- [x] Simpan ke Google Sheets
- [x] Konfirmasi via WhatsApp
- [ ] Laporan harian/mingguan/bulanan
- [ ] Budget alert
- [ ] Hapus transaksi terakhir
- [ ] Multi-user support
- [ ] Export laporan PDF

---

## 📄 License

MIT License — bebas digunakan dan dimodifikasi.

---

Made with ❤️ by [kirfansyah](https://github.com/kirfansyah)
