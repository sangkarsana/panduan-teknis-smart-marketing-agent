# LAMPIRAN

## Lampiran A: Template Excel Import Data

### A.1 Template Import Pelanggan

Download template: `template_import_pelanggan.xlsx`

**Format Kolom:**
| Kolom | Nama Field | Tipe Data | Wajib | Keterangan |
|-------|------------|-----------|-------|------------|
| A | Nama | Text | Ya | Nama lengkap pelanggan |
| B | No_HP | Text | Ya | Format: 08xxxxxxxxxx |
| C | Email | Text | Tidak | Format email valid |
| D | Alamat | Text | Tidak | Alamat lengkap |
| E | Kota | Text | Tidak | Nama kota |
| F | Tags | Text | Tidak | Pisahkan dengan koma |

**Contoh Data:**
```
Nama            | No_HP        | Email              | Alamat                    | Kota      | Tags
----------------|--------------|-------------------|---------------------------|-----------|------------------
Ibu Siti Rahman | 08123456789  | siti@email.com    | Jl. Malioboro No. 123    | Yogyakarta| VIP, Reseller
Bpk Ahmad Yani  | 08234567890  |                   | Jl. Asia Afrika No. 45   | Bandung   | Corporate
Ibu Ratna Dewi  | 08345678901  | ratna@gmail.com   | Jl. Pemuda No. 78        | Surabaya  | Online, Loyal
```

**Validation Rules:**
- Nama: Minimal 2 karakter, maksimal 100 karakter
- No_HP: Harus diawali 08, panjang 10-13 digit
- Email: Format email valid (optional)
- Tags: Maksimal 10 tags per pelanggan

### A.2 Template Import Transaksi

Download template: `template_import_transaksi.xlsx`

**Format Kolom:**
| Kolom | Nama Field | Tipe Data | Wajib | Keterangan |
|-------|------------|-----------|-------|------------|
| A | Nama_atau_HP | Text | Ya | Nama/HP pelanggan yang sudah ada |
| B | Tanggal | Date | Ya | Format: DD/MM/YYYY |
| C | Total_Nilai | Number | Ya | Tanpa titik/koma |
| D | Produk | Text | Tidak | Deskripsi produk |
| E | Catatan | Text | Tidak | Catatan tambahan |

**Contoh Data:**
```
Nama_atau_HP    | Tanggal    | Total_Nilai | Produk                          | Catatan
----------------|------------|-------------|--------------------------------|------------------
08123456789     | 01/08/2025 | 1500000     | Batik Parang 2pcs              | Untuk acara kantor
Ibu Siti Rahman | 05/08/2025 | 750000      | Batik Kawung                   | Repeat order
08345678901     | 06/08/2025 | 2000000     | Batik Tulis Premium 3pcs       | Customer baru dari IG
```

## Lampiran B: API Documentation

### B.1 Authentication

**Base URL:** `https://api.smartmarketingagent.id/v1`

**Authentication Header:**
```http
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

### B.2 Customer Endpoints

#### Create Customer
```http
POST /customers

Request Body:
{
  "name": "Ibu Siti Rahman",
  "phone": "08123456789",
  "email": "siti@email.com",
  "address": "Jl. Malioboro No. 123",
  "city": "Yogyakarta",
  "tags": ["VIP", "Reseller"]
}

Response (201 Created):
{
  "success": true,
  "data": {
    "id": "cust_abc123",
    "name": "Ibu Siti Rahman",
    "phone": "08123456789",
    "created_at": "2025-08-06T02:20:46Z"
  }
}
```

#### Get Customers
```http
GET /customers?page=1&limit=20&search=siti&segment=champions

Response (200 OK):
{
  "success": true,
  "data": {
    "customers": [...],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45,
      "total_pages": 3
    }
  }
}
```

### B.3 Transaction Endpoints

#### Create Transaction
```http
POST /transactions

Request Body:
{
  "customer_id": "cust_abc123",
  "transaction_date": "2025-08-06",
  "amount": 1500000,
  "products": [
    {
      "name": "Batik Parang",
      "quantity": 2,
      "price": 750000
    }
  ],
  "notes": "Untuk acara kantor"
}

Response (201 Created):
{
  "success": true,
  "data": {
    "id": "trx_xyz789",
    "customer_id": "cust_abc123",
    "amount": 1500000,
    "created_at": "2025-08-06T02:20:46Z"
  }
}
```

### B.4 RFM Analysis Endpoints

#### Get RFM Analysis
```http
GET /rfm/analysis

Response (200 OK):
{
  "success": true,
  "data": {
    "analysis_date": "2025-08-06",
    "total_customers": 547,
    "segments": {
      "champions": 47,
      "loyal_customers": 89,
      "potential_loyalists": 124,
      "new_customers": 67,
      "at_risk": 98,
      "lost": 122
    },
    "results": [
      {
        "customer_id": "cust_abc123",
        "customer_name": "Ibu Siti Rahman",
        "segment": "champions",
        "recency_score": 5,
        "frequency_score": 5,
        "monetary_score": 5,
        "last_transaction": "2025-08-01",
        "total_transactions": 24,
        "total_spent": 15000000
      }
      // ... more customers
    ]
  }
}
```

### B.5 Content Generation Endpoints

#### Generate Marketing Content
```http
POST /content/generate

Request Body:
{
  "segment": "champions",
  "channel": "whatsapp",
  "objective": "new_collection",
  "tone": "exclusive",
  "additional_context": "Koleksi Lebaran 2025"
}

Response (200 OK):
{
  "success": true,
  "data": {
    "content": "Selamat pagi Ibu 🌸\n\nSebagai pelanggan istimewa kami...",
    "tokens_used": 150,
    "suggestions": [
      "Add customer name for personalization",
      "Include specific product images",
      "Best sending time: 10:00-11:00 AM"
    ]
  }
}
```

### B.6 Error Responses

```json
// 400 Bad Request
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Data tidak valid",
    "details": {
      "phone": "Format nomor HP tidak sesuai"
    }
  }
}

// 401 Unauthorized
{
  "success": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "API key tidak valid"
  }
}

// 429 Too Many Requests
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Terlalu banyak request",
    "retry_after": 60
  }
}

// 500 Internal Server Error
{
  "success": false,
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "Terjadi kesalahan sistem"
  }
}
```

### B.7 Rate Limits

| Plan | Requests/Hour | Requests/Day | AI Generations/Month |
|------|---------------|--------------|---------------------|
| Free | 100 | 1,000 | 100 |
| Starter | 1,000 | 10,000 | 1,000 |
| Business | 10,000 | 100,000 | 10,000 |
| Enterprise | Unlimited | Unlimited | Custom |

## Lampiran C: Contoh Skrip WhatsApp

### C.1 Welcome Series (3 Bagian)

**Pesan 1 - Immediately After First Purchase:**
```
Halo Kak [Nama] 👋

Terima kasih sudah mempercayai [Nama Toko] untuk kebutuhan batik Kakak! 🙏

Pesanan Kakak sedang kami siapkan dengan penuh cinta. Estimasi kirim: [Tanggal]

Oh ya, kami kirimkan panduan perawatan batik di attachment ya, biar batiknya awet dan tetap cantik ✨

Ada yang bisa kami bantu? Just chat aja!

Salam hangat,
[Nama CS] - [Nama Toko]
```

**Pesan 2 - Day 3 After Delivery:**
```
Hi Kak [Nama], gimana batiknya? Sudah sampai kan? 😊

Semoga suka ya! Kalau ada kendala apapun, jangan sungkan kabarin kami.

BTW, kalau Kakak puas dengan produk & pelayanan kami, boleh banget di-share ke teman2. We really appreciate it! 🥰

Sebagai thank you, nanti dapat special discount deh buat pembelian berikutnya 🎁

Have a great day!
```

**Pesan 3 - Day 7:**
```
Hai Kak [Nama] 🌺

Quick sharing aja nih! Tau ga sih, motif batik yang Kakak beli ([Nama Motif]) itu punya makna [Makna Motif].

Keren kan? Jadi ga cuma cantik, tapi meaningful juga! 

Kalau mau tau lebih banyak tentang batik & styling tips, join group WA kami yuk! Banyak sharing seru + exclusive deals 😉

Join here: [Link Group]

See you there! 💕
```

### C.2 Win-Back Campaign Series

**Soft Approach (3 bulan tidak aktif):**
```
Halo Ibu [Nama], apa kabar? 😊

Sudah 3 bulan nih kita ga ketemu. Semoga Ibu sehat selalu ya!

Kebetulan kami baru terima koleksi batik [Jenis] yang cantik banget. Ingat dulu Ibu pernah cari yang warna [Warna] kan? Nah ini ada lho!

Mau lihat fotonya? Atau ada motif lain yang Ibu cari?

Kangen melayani Ibu lagi 🥺

Warm regards,
[Nama] - [Nama Toko]
```

**Medium Approach (4-5 bulan):**
```
Dear Ibu [Nama],

Kami notice Ibu sudah lama tidak berbelanja di [Nama Toko]. 

Apa ada yang kurang memuaskan dari pelayanan kami? Kalau ada, please let us know ya Bu. Feedback Ibu sangat berharga untuk kami improve 🙏

Sebagai apresiasi untuk Ibu sebagai pelanggan lama, kami ada:
✨ Special Discount 20% all items
✨ Free Ongkir se-Indonesia
✨ Bonus Batik Scrunchie

Valid sampai: [Tanggal] (7 hari dari sekarang)

Kode: COMEBACKIBU

Really hope to serve you again! 💝
```

**Urgent Approach (6+ bulan):**
```
Ibu [Nama], this is [Owner Name] from [Nama Toko] 👋

Saya personally mau reach out karena Ibu adalah salah satu pelanggan terbaik kami, tapi sudah 6 bulan tidak ada kabar 😢

Jujur, kami merasa kehilangan. Apa ada yang bisa kami perbaiki?

Saya mau kasih penawaran SUPER SPECIAL untuk Ibu:
🎁 Discount 40% ALL ITEMS (yes, forty percent!)
🎁 FREE Premium Batik Pouch
🎁 Priority Personal Service

No minimum purchase. Valid 72 jam saja.

Ibu bisa langsung reply WA ini atau call saya di [Nomor].

Really hope we can serve you again 🙏

Warmest regards,
[Owner Name]
P.S. Even if Ibu tidak tertarik, feedback Ibu tetap sangat berarti untuk kami
```

### C.3 Seasonal Campaign Templates

**Lebaran Campaign:**
```
Assalamualaikum Ibu [Nama] 🌙

Menyambut Ramadhan yang penuh berkah, [Nama Toko] mempersembahkan:

✨ KOLEKSI LEBARAN 2025 ✨

🎁 Early Bird Discount 25% (sampai [Tanggal])
🎁 FREE Packaging Eksklusif 
🎁 Kirim sampai H-3 Lebaran

Koleksi terbatas dengan motif:
- Batik [Motif 1]: Elegan untuk silaturahmi
- Batik [Motif 2]: Modern untuk generasi muda
- Batik [Motif 3]: Couple/Family series

Lihat koleksi lengkap: [Link]

Atau mau personal shopping assistance? Saya bantu pilihkan yang cocok untuk Ibu 😊

Happy shopping & Ramadhan Mubarak!
```

**Batik Day Campaign (2 Oktober):**
```
Happy Batik Day, Kak [Nama]! 🇮🇩

Tau ga sih? Tanggal 2 Oktober adalah Hari Batik Nasional untuk memperingati UNESCO mengakui batik sebagai Warisan Budaya Indonesia!

Spesial hari ini, kami ada:
🎉 Flash Sale 30% untuk 100 pembeli pertama
🎉 Batik Trivia berhadiah
🎉 Free Batik Pin untuk setiap pembelian

Plus, 10% dari penjualan hari ini kami donasikan untuk pembinaan pengrajin batik lokal 💝

Yuk celebrate Indonesian heritage bareng!

Shop now: [Link]
Quiz: [Link]

#BatikIndonesia #ProudlyIndonesian
```

## Lampiran D: Glossary

### D.1 Istilah Teknis

| Istilah | Definisi |
|---------|----------|
| **API (Application Programming Interface)** | Antarmuka yang memungkinkan aplikasi berkomunikasi dengan aplikasi lain |
| **AI (Artificial Intelligence)** | Kecerdasan buatan yang dapat melakukan tugas-tugas yang biasanya memerlukan kecerdasan manusia |
| **CRM (Customer Relationship Management)** | Sistem pengelolaan hubungan pelanggan |
| **Dashboard** | Tampilan utama yang menampilkan ringkasan data penting |
| **Database** | Tempat penyimpanan data terstruktur |
| **GPT (Generative Pre-trained Transformer)** | Model AI untuk menghasilkan teks |
| **ROI (Return on Investment)** | Perbandingan antara keuntungan dengan investasi |
| **SaaS (Software as a Service)** | Perangkat lunak yang diakses melalui internet |
| **UI/UX** | User Interface (tampilan) dan User Experience (pengalaman pengguna) |
| **Webhook** | Cara aplikasi memberikan informasi real-time ke aplikasi lain |

### D.2 Istilah RFM

| Istilah | Definisi | Contoh |
|---------|----------|---------|
| **Recency** | Seberapa baru pelanggan terakhir bertransaksi | 7 hari yang lalu |
| **Frequency** | Seberapa sering pelanggan bertransaksi | 5x dalam setahun |
| **Monetary** | Total nilai transaksi pelanggan | Rp 10.000.000 |
| **Champions** | Pelanggan terbaik (R↑ F↑ M↑) | Beli tiap bulan, nilai tinggi |
| **At Risk** | Pelanggan berisiko hilang (R↓ F↑ M↑) | Dulu rajin, sekarang jarang |
| **Lost** | Pelanggan yang sudah tidak aktif (R↓ F↓ M↓) | > 6 bulan tidak beli |
| **Segment** | Kelompok pelanggan berdasarkan perilaku | Champions, Loyal, dll |
| **Score** | Nilai 1-5 untuk setiap dimensi RFM | R=5, F=4, M=5 |

### D.3 Istilah Marketing

| Istilah | Definisi | Contoh |
|---------|----------|---------|
| **AOV (Average Order Value)** | Rata-rata nilai per transaksi | Rp 750.000 |
| **CAC (Customer Acquisition Cost)** | Biaya untuk mendapatkan pelanggan baru | Rp 50.000/customer |
| **Churn Rate** | Persentase pelanggan yang berhenti beli | 5% per bulan |
| **Conversion Rate** | Persentase yang melakukan pembelian | 10% dari yang dikontrak |
| **CTR (Click Through Rate)** | Persentase yang klik link | 25% buka WA |
| **Engagement Rate** | Tingkat interaksi dengan konten | Like, comment, share |
| **LTV (Lifetime Value)** | Total nilai pelanggan seumur hidup | Rp 50.000.000 |
| **Retention Rate** | Persentase pelanggan yang tetap aktif | 85% tetap beli |

## Lampiran E: Troubleshooting Guide

### E.1 Masalah Login

**Problem:** Tidak bisa login
```
Solusi:
1. Cek email dan password sudah benar
2. Pastikan Caps Lock tidak aktif
3. Coba reset password:
   - Klik "Lupa Password"
   - Masukkan email terdaftar
   - Cek inbox (dan folder spam)
   - Klik link reset password
4. Jika masih gagal, kontak support
```

**Problem:** Session timeout terus
```
Solusi:
1. Clear browser cache dan cookies
2. Gunakan browser updated (Chrome/Firefox/Safari)
3. Disable browser extensions yang block scripts
4. Check internet connection stability
```

### E.2 Masalah Data

**Problem:** Data pelanggan hilang
```
Solusi:
1. Cek filter - mungkin tersaring
2. Coba search dengan nama/HP
3. Refresh halaman (F5)
4. Cek di tab "Archived" jika ada
5. Kontak support dengan screenshot
```

**Problem:** Import Excel gagal
```
Checklist:
□ File format .xlsx atau .csv
□ Ukuran file < 5MB
□ Kolom sesuai template
□ Tidak ada special character (© ® ™)
□ Nomor HP format 08xx
□ Tidak ada formula Excel aktif

Solusi:
1. Download template baru
2. Copy-paste data only (no formatting)
3. Save as new file
4. Import ulang
```

### E.3 Masalah AI Content

**Problem:** AI hasil tidak sesuai
```
Tips perbaikan:
1. Lebih spesifik dalam request
   ❌ "Buat konten promo"
   ✅ "Buat konten promo batik parang untuk wanita 30-40 tahun, tone friendly, focus pada kualitas"

2. Edit manual hasil AI
3. Save sebagai template untuk consistency
4. Report ke support untuk improvement
```

**Problem:** AI generation timeout
```
Solusi:
1. Coba generate ulang
2. Kurangi panjang request
3. Pilih waktu non-peak (bukan jam 10-12, 19-21)
4. Gunakan template yang sudah ada
```

### E.4 Masalah Performance

**Problem:** Aplikasi lambat
```
Optimasi:
1. Clear browser cache
2. Tutup tab browser lain
3. Restart browser/device
4. Check internet speed (min 1 Mbps)
5. Gunakan browser modern
6. Disable heavy browser extensions
```

**Problem:** Data tidak ter-update
```
Solusi:
1. Refresh halaman (F5)
2. Hard refresh (Ctrl+Shift+R)
3. Logout dan login kembali
4. Clear browser data
5. Coba browser lain
```

## Lampiran F: Resources & Links

### F.1 Tutorial Videos

| Topic | Link | Durasi |
|-------|------|--------|
| Getting Started | youtube.com/watch?v=xxx1 | 15 min |
| Import Data Pelanggan | youtube.com/watch?v=xxx2 | 10 min |
| Memahami RFM Analysis | youtube.com/watch?v=xxx3 | 20 min |
| Generate AI Content | youtube.com/watch?v=xxx4 | 12 min |
| Campaign Best Practices | youtube.com/watch?v=xxx5 | 25 min |

### F.2 Download Resources

- **User Manual PDF**: [Download](https://smartmarketingagent.id/downloads/user-manual.pdf)
- **Quick Start Guide**: [Download](https://smartmarketingagent.id/downloads/quick-start.pdf)
- **Template Pack**: [Download](https://smartmarketingagent.id/downloads/templates.zip)
- **API Documentation**: [Download](https://smartmarketingagent.id/downloads/api-docs.pdf)
- **Case Studies**: [Download](https://smartmarketingagent.id/downloads/case-studies.pdf)

### F.3 Community Links

- **Facebook Group**: facebook.com/groups/smartmarketingagent
- **Telegram Community**: t.me/sma_indonesia
- **WhatsApp Updates**: wa.me/628123456789
- **Instagram**: @smartmarketingagent
- **LinkedIn**: linkedin.com/company/smart-marketing-agent

### F.4 Support Channels

**Email Support**
- General: support@smartmarketingagent.id
- Technical: tech@smartmarketingagent.id
- Billing: billing@smartmarketingagent.id

**WhatsApp Support**
- Number: +62 812-3456-7890
- Hours: Senin-Jumat 09:00-17:00 WIB
- Response time: < 2 jam

**Live Chat**
- Available di dashboard
- Hours: Senin-Jumat 09:00-21:00 WIB
- Weekend: 10:00-16:00 WIB

### F.5 Legal & Compliance

- **Terms of Service**: [Link](https://smartmarketingagent.id/terms)
- **Privacy Policy**: [Link](https://smartmarketingagent.id/privacy)
- **Data Protection**: [Link](https://smartmarketingagent.id/data-protection)
- **Cookie Policy**: [Link](https://smartmarketingagent.id/cookies)
- **SLA Agreement**: [Link](https://smartmarketingagent.id/sla)

## Lampiran G: Changelog

### Version 2.0.0 (August 2025)
- ✨ Launch Smart Marketing Agent untuk UMKM Batik
- ✨ RFM Analysis Engine
- ✨ AI Content Generation dengan GPT-4
- ✨ WhatsApp & Email Integration
- ✨ Customer Import dari Excel
- ✨ Basic Dashboard Analytics

### Upcoming Features (Q4 2025)
- 🚀 Multi-language support (English, Javanese)
- 🚀 Instagram Auto-posting
- 🚀 Advanced Analytics Dashboard
- 🚀 Inventory Management Beta
- 🚀 Mobile App (Android)

### Roadmap 2026
- 🔮 Marketplace Integration (Tokopedia, Shopee)
- 🔮 Voice Commerce
- 🔮 AR Product Preview
- 🔮 Blockchain Authentication
- 🔮 Expansion to F&B Industry

---

*Dokumen ini akan terus diperbarui. Last updated: August 6, 2025*
