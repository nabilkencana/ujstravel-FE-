# Setup Google Drive Upload (Google Apps Script)

Untuk mengaktifkan fitur simpan bukti transfer langsung ke Google Drive, ikuti langkah-langkah berikut:

## 1. Buat Google Apps Script
1. Buka [Google Apps Script](https://script.google.com/).
2. Klik **New Project**.
3. Hapus kode yang ada dan tempelkan kode di bawah ini:

```javascript
function doPost(e) {
  var result = {};
  try {
    var params = JSON.parse(e.postData.contents);

    // --- KONFIGURASI ---
    // Ganti dengan ID folder Google Drive Anda (opsional)
    // Jika dikosongkan, file akan disimpan di root Drive
    var folderId = "";
    // -------------------

    var folder = folderId ? DriveApp.getFolderById(folderId) : DriveApp.getRootFolder();

    var contentType = params.mimeType;
    var data = Utilities.base64Decode(params.file);
    var blob = Utilities.newBlob(data, contentType, params.fileName);

    var file = folder.createFile(blob);
    file.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);

    result = {
      success: true,
      url: file.getUrl(),
      fileId: file.getId()
    };
  } catch (error) {
    result = {
      success: false,
      error: error.toString()
    };
  }

  return ContentService.createTextOutput(JSON.stringify(result))
    .setMimeType(ContentService.MimeType.JSON);
}
```

## 2. Deploy sebagai Web App
1. Klik tombol **Deploy** > **New deployment**.
2. Pilih tipe: **Web app**.
3. Deskripsi: `UJS Travel Form Upload`.
4. Execute as: **Me** (Email Anda).
5. Who has access: **Anyone** (Ini penting agar form publik bisa mengirim data).
6. Klik **Deploy**.
7. Salin **Web App URL** yang muncul.

## 3. Update di Form.html
1. Buka file `Form.html`.
2. Cari variabel `GOOGLE_SCRIPT_URL` di dalam bagian `<script>`.
3. Tempelkan URL yang sudah Anda salin tadi di antara tanda kutip:
   `const GOOGLE_SCRIPT_URL = "URL_HASIL_DEPLOY_ANDA";`

## Dimana Filenya Tersimpan?
- **Lokasi Default:** Secara default, file akan tersimpan di halaman depan (**Root**) Google Drive Anda.
- **Kustomisasi:** Jika ingin menyimpan di folder tertentu, masukkan ID folder tersebut ke variabel `folderId` di dalam kode Google Apps Script di atas.
- **Akses:** Script ini otomatis mengatur file menjadi "Anyone with link can view", sehingga Admin bisa melihat bukti transfer melalui link yang dikirim ke WhatsApp.
