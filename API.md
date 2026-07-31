# Kontrak API SIPENA GAS

Frontend tidak memanggil endpoint JSON biasa. Semua request dikirim sebagai form POST ke URL Web App `/exec` dengan tiga field:

```text
requestId      ID unik request
frontendOrigin origin GitHub Pages
payload        JSON string
```

Backend mengembalikan HTML singkat yang menjalankan `window.parent.postMessage()`.

## Action publik

- `ping`
- `getSettings`
- `getPublicStats`
- `getCategories`
- `submitReport`
- `trackReport`
- `login`
- `getSession`
- `logout`
- `resetPassword`

## Action Admin dan Petugas

- `getAllReports`
- `updateReportStatus`
- `sendReporterNotification`

## Action khusus Admin

- `deleteReport`
- `getAdminSettings`
- `updateSettings`
- `getUsers`
- `saveUser`
- `deleteUser`
- `saveCategory`
- `deleteCategory`

Action yang dilindungi wajib membawa `session_token`. Frontend menambahkannya otomatis dari `localStorage`.
