# 📋 Panduan Setup Google Spreadsheet untuk Buku Tamu

## 🎯 Tujuan
Mengintegrasikan form ucapan manual di halaman undangan dengan Google Spreadsheet agar semua ucapan tamu tersimpan dan dapat ditampilkan otomatis.

---

## 📝 LANGKAH-LANGKAH SETUP

### Step 1: Buat Google Spreadsheet Baru
1. Buka https://sheets.google.com
2. Klik **"+ Blank"** untuk membuat spreadsheet baru
3. Beri nama: **"Ucapan Pernikahan Shinta & Dhimas"**
4. Buat header di row pertama dengan kolom:
   - **A1:** `Tanggal`
   - **B1:** `Nama`
   - **C1:** `Ucapan`

Contoh:
```
|      Tanggal      |    Nama    |        Ucapan       |
|-------------------|------------|-------------------|
| 2026-08-14 10:30  | Budi       | Selamat bahagia... |
| 2026-08-15 14:20  | Siti       | Semoga langgeng... |
```

---

### Step 2: Buat Google Apps Script
1. Di spreadsheet, klik **Tools** → **Script Editor**
2. Hapus semua kode yang ada
3. **Paste kode berikut:**

```javascript
function doPost(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSheet();
    const data = JSON.parse(e.postData.contents);
    
    // Tambahkan data ke spreadsheet
    sheet.appendRow([
      new Date(),
      data.name,
      data.message
    ]);
    
    return ContentService.createTextOutput(JSON.stringify({status: "success"}))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({status: "error", message: error.toString()}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSheet();
    const data = sheet.getRange(2, 1, sheet.getLastRow() - 1, 3).getValues();
    
    // Format data untuk front-end
    const wishes = data.map(row => ({
      date: row[0],
      name: row[1],
      message: row[2]
    })).reverse(); // Tampilkan yang terbaru di atas
    
    return ContentService.createTextOutput(JSON.stringify(wishes))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify([]))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

4. Klik **Save** (Ctrl+S)
5. Beri nama project: **"Ucapan Pernikahan Handler"**

---

### Step 3: Deploy sebagai Web App
1. Klik tombol **▶ Deploy** (pojok kanan atas)
2. Pilih **"New Deployment"** (atau klik ikon ⚙️ lalu "+ New")
3. Di dropdown "Select type", pilih **Apps Script**
4. Atur:
   - **Description:** "Endpoint untuk ucapan pernikahan"
   - **Execute as:** Pilih akun Google Anda
   - **Who has access:** **"Anyone"** (PENTING!)

5. Klik **Deploy**
6. **Salin URL Web App** yang ditampilkan. Contoh:
   ```
   https://script.google.com/macros/s/AKfycbXxxxxxxxxxxxx/userweb
   ```

---

### Step 4: Update URL di HTML
1. Buka file `index.html`
2. Cari baris:
   ```javascript
   const GOOGLE_APPS_SCRIPT_URL = "https://script.google.com/macros/d/YOUR_SCRIPT_ID/userweb?v=1";
   ```
3. Ganti `YOUR_SCRIPT_ID` dengan URL yang sudah di-copy pada Step 3
4. **Hasilnya harus terlihat seperti ini:**
   ```javascript
   const GOOGLE_APPS_SCRIPT_URL = "https://script.google.com/macros/s/AKfycbXxxxxxxxxxxxx/userweb";
   ```

---

## ✅ TESTING

1. Buka halaman undangan di browser
2. Klik **"Buka Undangan"**
3. Scroll ke bagian **"Buku Tamu"**
4. Isi:
   - Nama: `Budi`
   - Ucapan: `Selamat bahagia!`
5. Klik **"Kirim Ucapan"**
6. ✅ Jika berhasil, akan muncul notifikasi "Terima kasih!"
7. Cek di Google Spreadsheet - data harus tersimpan
8. Refresh halaman - ucapan harus muncul di list

---

## 🐛 TROUBLESHOOTING

### Masalah: "Gagal mengirim ucapan"
**Solusi:**
- ✓ Pastikan URL Web App sudah benar (tidak ada typo)
- ✓ Pastikan deployment setting "Who has access" = "Anyone"
- ✓ Buka URL Web App langsung di browser untuk test
- ✓ Buka Developer Tools (F12) → Console untuk lihat error detail

### Masalah: "Gagal memuat ucapan"
**Solusi:**
- ✓ Cek apakah spreadsheet memiliki data di bawah header
- ✓ Pastikan Google Apps Script sudah di-deploy ulang setelah perubahan
- ✓ Buka console dan lihat pesan error

### Masalah: CORS Error
**Solusi:**
- ✓ Ini normal! Gunakan mode "Anyone" di deployment
- ✓ Jika masih error, coba mode "execute as" dengan akun pemilik

---

## 🔄 UPDATE / EDIT URL Web App

Jika perlu mengganti URL nanti:
1. Buka Google Apps Script
2. Klik **Deploy** → pilih deployment terbaru
3. Klik ⚙️ (settings) → **Edit**
4. Ubah code jika diperlukan
5. Klik **Deploy** → akan dapat URL baru

---

## 💡 TIPS TAMBAHAN

### Custom Columns
Jika ingin menambah kolom (contoh: "Status Kehadiran"), edit kode di Apps Script:
```javascript
// Di bagian doPost(), tambah:
data.attendance // Dari form HTML

// Di bagian sheet.appendRow():
sheet.appendRow([
  new Date(),
  data.name,
  data.message,
  data.attendance  // Kolom baru
]);
```

### Private Spreadsheet
Jika ingin spreadsheet privat tapi tetap bisa diakses:
- **Jangan gunakan "Anyone"**
- Gunakan authentication dengan token/API key
- Atau gunakan Google Forms alternative

### Backup Data
Secara berkala download data dari spreadsheet:
1. File → Download → CSV
2. Simpan dengan nama `backup_ucapan_[tanggal].csv`

---

**Selamat! Fitur Buku Tamu Anda sudah siap digunakan! 🎉**
