# SIPENA — Frontend GitHub Pages + Backend Google Apps Script

Paket ini mengubah SIPENA menjadi arsitektur berikut:

```text
GitHub Pages (index.html)
        │
        │ POST melalui iframe bridge + postMessage
        ▼
Google Apps Script Web App (Code.gs)
        │
        ├── Google Spreadsheet sebagai database
        └── Fonnte API untuk notifikasi WhatsApp
```

Frontend tidak memakai PHP dan tidak memakai MySQL. Semua data pengaduan, akun, kategori, pengaturan, sesi, dan log disimpan pada Google Spreadsheet.

## Isi paket

```text
index.html                 Frontend utama untuk GitHub Pages
404.html                   Salinan frontend untuk fallback GitHub Pages
config.js                  Tempat memasukkan URL deployment Google Apps Script
.nojekyll                  Mencegah pemrosesan Jekyll

gas/Code.gs               Backend Google Apps Script
gas/appsscript.json        Manifest Apps Script
gas/README-BACKEND.md      Petunjuk backend

STRUKTUR-SPREADSHEET.md     Struktur database Spreadsheet
README.md                   Petunjuk instalasi ini
```

## A. Menyiapkan backend Google Apps Script

1. Buka `https://script.google.com` dan buat **New project**.
2. Ganti isi `Code.gs` dengan file `gas/Code.gs` dari paket ini.
3. Buka **Project Settings** dan aktifkan **Show "appsscript.json" manifest file in editor**.
4. Ganti isi `appsscript.json` dengan file `gas/appsscript.json`.
5. Pada bagian atas `Code.gs`, ubah:

```javascript
const INSTALL_CONFIG = Object.freeze({
  DATABASE_NAME: 'DATABASE SIPENA KPU TANJUNG JABUNG TIMUR',
  ADMIN_USERNAME: 'admin',
  ADMIN_PASSWORD: 'PASSWORD_KUAT_ANDA',
  ADMIN_NAME: 'Administrator SIPENA',
  ADMIN_NIP: 'NIP ADMIN',
  ADMIN_EMAIL: 'email@kpu.go.id'
});
```

6. Pilih fungsi `setupSIPENA`, lalu klik **Run**.
7. Setujui izin Spreadsheet, Drive, dan koneksi eksternal.
8. Jalankan `diagnoseSIPENA`. Hasil normal:

```json
{
  "status": "success",
  "database_connected": true,
  "active_admins_count": 1
}
```

9. Untuk melihat URL Spreadsheet, jalankan `getDatabaseInfo` lalu buka **Execution log**.

## B. Deploy backend sebagai Web App

1. Klik **Deploy → New deployment**.
2. Pilih **Web app**.
3. Atur:

```text
Execute as      : Me
Who has access : Anyone
```

4. Klik **Deploy**.
5. Salin URL Web App yang berakhiran `/exec`, misalnya:

```text
https://script.google.com/macros/s/AKfycbxxxxxxxxxxxxxxxx/exec
```

6. Uji backend dengan membuka:

```text
URL_EXEC_ANDA?health=1
```

## C. Menghubungkan frontend GitHub

Buka `config.js`, kemudian ganti:

```javascript
GAS_EXEC_URL: 'PASTE_URL_GOOGLE_APPS_SCRIPT_EXEC_DI_SINI'
```

menjadi URL `/exec` yang diperoleh dari Apps Script:

```javascript
GAS_EXEC_URL: 'https://script.google.com/macros/s/AKfycbxxxxxxxxxxxxxxxx/exec'
```

URL deployment bukan data rahasia. Token Fonnte tidak dimasukkan ke GitHub; token disimpan melalui dashboard admin dan tersimpan di Spreadsheet.

## D. Upload ke GitHub Pages

1. Buat repository GitHub, misalnya `sipena-kpu-tjt`.
2. Upload seluruh isi folder ini ke root repository.
3. Buka **Settings → Pages**.
4. Atur:

```text
Source : Deploy from a branch
Branch : main
Folder : / (root)
```

5. Klik **Save**.
6. Tunggu sampai URL GitHub Pages aktif.

Contoh:

```text
https://username.github.io/sipena-kpu-tjt/
```

## E. Pengaturan pertama di SIPENA

1. Buka URL GitHub Pages.
2. Klik tombol **Login Petugas**.
3. Masuk menggunakan akun yang dibuat melalui `INSTALL_CONFIG`.
4. Buka **Pengaturan Kontak**.
5. Isi:
   - alamat kantor;
   - nomor WhatsApp kantor;
   - email kantor;
   - URL frontend GitHub Pages;
   - token Fonnte;
   - nomor atau ID grup petugas;
   - pilihan notifikasi otomatis.
6. Klik **Simpan Kontak & WA**.

## Cara komunikasi frontend–backend

Google Apps Script Web App tidak selalu dapat dipanggil dengan `fetch()` lintas domain karena mekanisme redirect dan kebijakan CORS. Paket ini menggunakan metode berikut:

1. frontend membuat form POST tersembunyi;
2. form dikirim ke URL `/exec` dalam iframe tersembunyi;
3. backend memproses permintaan;
4. backend mengirim hasil ke frontend dengan `window.parent.postMessage()`;
5. frontend mencocokkan `requestId` dan sumber iframe sebelum menerima data.

Karena itu, login dan seluruh operasi admin tetap dapat bekerja dari GitHub Pages tanpa menaruh password atau token API pada URL.

## Database Spreadsheet yang dibuat otomatis

- `Reports`
- `Users`
- `Categories`
- `Settings`
- `Sessions`
- `AuditLog`

Jangan mengubah nama header pada baris pertama. Data boleh dilihat atau diekspor langsung dari Spreadsheet.

## Memperbarui backend

Setelah mengubah `Code.gs`:

```text
Deploy → Manage deployments → Edit → New version → Deploy
```

URL `/exec` tetap sama selama deployment yang sama diperbarui.

## Memperbaiki login admin

1. Ubah `ADMIN_PASSWORD` pada `INSTALL_CONFIG`.
2. Jalankan `repairAdminLogin()`.
3. Deploy sebagai **New version**.
4. Sesi lama akan dihapus.

## Pengujian WhatsApp

Setelah token dan nomor kantor disimpan melalui dashboard admin, jalankan fungsi:

```text
testWhatsAppGateway
```

Hasil pengujian dapat dilihat pada Execution log.

## Catatan keamanan

- Jangan menaruh token Fonnte di `config.js` atau repository GitHub.
- Gunakan password administrator yang kuat.
- Jangan membagikan sheet `Users`, `Sessions`, dan `Settings` kepada publik.
- Deployment GAS harus dijalankan sebagai pemilik aplikasi.
- Sheet `AuditLog` menyimpan aktivitas pengelolaan aplikasi.
- Token sesi disimpan dalam bentuk hash pada Spreadsheet dan berlaku delapan jam.
