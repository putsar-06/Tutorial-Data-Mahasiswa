# Setup dan Proses Pembuatan Program

Tutorial ini untuk setup dan proses membuat program.

---

## 1. Install Node.js

### Cek Sudah Terinstall atau Belum

Buka terminal, ketik:

```bash
node --version
npm --version
```

Kalau muncul angka versi (misal: `v20.18.0` dan `10.8.0`) berarti sudah terinstall.

### Cara Install Node.js

1. Buka website: https://nodejs.org
2. Download versi **LTS** (Long Term Support) - yang ada tulisan "Recommended"
3. Install seperti aplikasi biasa

---

## 2. Buat Folder Project

```bash
# Buat folder baru
mkdir projects
cd projects

# Buat file package.json
npm init -y
```

Perintah `npm init -y` akan membuat file `package.json` otomatis dengan setting default.

Isi file `package.json` yang terbentuk:

```json
{
  "name": "projects",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

---

## 3. Install Library yang Dibutuhkan

Kita butuh 4 library (node_modules). Install satu per satu:

### 3.1 Install Express (Framework Web)

```bash
npm install express
```

### 3.2 Install Database SQLite

```bash
npm install better-sqlite3
```

### 3.3 Install Handlebars

```bash
npm install express-handlebars
```

### 3.4 Install Nodemon

```bash
npm install nodemon -D
```
---

## 4. Buat Struktur Folder dan File

```bash
# Buat folder untuk template
mkdir -p views/layouts
mkdir -p views/mahasiswa

# Buat file-file utama
touch index.js
touch views/layouts/main.hbs
touch views/index.hbs
touch views/mahasiswa/add.hbs
touch views/mahasiswa/edit.hbs
```
---

## 5. Isi File Kode


### File: index.js

```javascript
const express = require('express');
const path = require('path');
const bodyParser = require('body-parser');

const { engine } = require('express-handlebars');

const DB = require('better-sqlite3');
const db = new DB('mahasiswa.db');

db.exec(`
    CREATE TABLE IF NOT EXISTS mahasiswa (
        nim TEXT PRIMARY KEY,
        nama TEXT NOT NULL,
        alamat TEXT NOT NULL
    )
`);

const hbs = engine({
    extname: 'hbs', 
    defaultLayout: 'main', 
    layoutsDir: path.join(__dirname, 'views', 'layouts'), 
    helpers: {
        noUrut: (index) => index + 1
    }
});

const app = express();
app.use(bodyParser.urlencoded({ extended: false })); 

app.engine('hbs', hbs);
app.set('view engine', 'hbs');
app.set('views', path.join(__dirname, 'views')); 

app.get('/', (req, res) => {
    const mahasiswa = db.prepare('SELECT * FROM mahasiswa').all();  
    res.render('index', {
        mahasiswa 
    });
});

app.get('/mahasiswa/add', (req, res) => {
    res.render('mahasiswa/add');
});

app.get('/mahasiswa/edit/:nim', (req, res) => {
    const { nim } = req.params;

    const mahasiswa = db.prepare('SELECT * FROM mahasiswa WHERE nim = ?')
    .get(nim);

    res.render('mahasiswa/edit',{
        mahasiswa
    });
});

app.post('/mahasiswa/add', (req, res) => {
    const { nim, nama, alamat } = req.body;
    
    db.prepare(`INSERT INTO mahasiswa (nim, nama, alamat) 
            VALUES (?, ?, ?)`)
        .run(nim, nama, alamat);
    
    res.redirect('/'); 
});

app.post('/mahasiswa/edit/:nim', (req, res) => {
    const { nim } = req.params; 
    const { nama, alamat } = req.body;

    db.prepare(`UPDATE mahasiswa SET nama = ?, alamat = ? WHERE nim = ?`)
        .run(nama, alamat, nim);
    res.redirect('/');
});

app.post('/mahasiswa/delete/:nim', (req, res) => {
    const { nim } = req.params;
    
    db.prepare(`DELETE FROM mahasiswa WHERE nim = ?`)
        .run(nim);
    
    res.redirect('/');
});

app.listen(3000, () => {
    console.log('Server is running on http://localhost:3000');
});
```
---
Disini berisi :
1. Import untuk memanggil Library express, sqlite, dan handlebars.
2. Buat file database mahasiswa.db + tabel mahasiswa (NIM, nama, alamat) jika belum ada.
![](/image/imgpg2.2.png)

---


### File: views/layouts/main.hbs

Template utama yang membungkus semua halaman:

```html
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Data Mahasiswa</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container">
        {{{body}}}
    </div>
</body>
</html>
```

### File: views/index.hbs

Halaman daftar mahasiswa (tabel dengan data).

```handlebars
<h1>List Data Mahasiswa</h1>
<hr>
<a href="/mahasiswa/add" class="btn btn-primary mb-3">Tambah Data</a>
<table class="table table-bordered">
    <thead>
        <tr>
            <th>No</th>
            <th>NIM</th>
            <th>Nama</th>
            <th>Alamat</th>
            <th>Aksi</th>
        </tr>
    </thead>
    <tbody>
        {{#each mahasiswa}}
        <tr>
            <td>{{noUrut @index}}.</td>
            <td>{{this.nim}}</td>
            <td>{{this.nama}}</td>
            <td>{{this.alamat}}</td>
            <td>
                <a href="/mahasiswa/edit/{{this.nim}}" class="btn btn-warning">Edit</a>
                {{!-- <a href="/mahasiswa/delete/{{this.nim}}" class="btn btn-danger" onclick="return confirm('Apakah Anda yakin ingin menghapus data ini?')">Delete</a> --}}
                <form action="/mahasiswa/delete/{{this.nim}}" 
                    method="POST" 
                    style="display: inline-block;">
                    <button type="submit" class="btn btn-danger" 
                    onclick="return confirm('Apakah Anda yakin ingin menghapus data ini?')">
                        Delete</button>
                </form>
            </td>
        </tr>
        {{/each}}
    </tbody>
</table>
```

### File: views/mahasiswa/add.hbs

Form untuk tambah data baru.
```handlebars
<h1>Tambah Data Mahasiswa</h1>
        <hr>
        <form action="/mahasiswa/add" method="POST">
            <div class="mb-3">
                <label for="nim" class="form-label">NIM</label>
                <input type="text" class="form-control" id="nim" name="nim" required>
            </div>
            <div class="mb-3">
                <label for="nama" class="form-label">Nama</label>
                <input type="text" class="form-control" id="nama" name="nama" required>
            </div>
            <div class="mb-3">
                <label for="alamat" class="form-label">Alamat</label>
                <input type="text" class="form-control" id="alamat" name="alamat" required>
            </div>
            <button type="submit" class="btn btn-primary">Simpan</button>
        </form>
```

### File: views/mahasiswa/edit.hbs

Form untuk edit data yang sudah ada.
```handlebars
<h1>Edit Data Mahasiswa</h1>
<hr>
<form action="/mahasiswa/edit/{{mahasiswa.nim}}" method="POST">
    <div class="mb-3">
        <label for="nim" class="form-label">NIM</label>
        <input type="text" class="form-control" id="nim" name="nim" required value="{{mahasiswa.nim}}" readonly>
    </div>
    <div class="mb-3">
        <label for="nama" class="form-label">Nama</label>
        <input type="text" class="form-control" id="nama" name="nama" required value="{{mahasiswa.nama}}">
    </div>
    <div class="mb-3">
        <label for="alamat" class="form-label">Alamat</label>
        <input type="text" class="form-control" id="alamat" name="alamat" required value="{{mahasiswa.alamat}}">
    </div>
    <button type="submit" class="btn btn-primary">Simpan</button>
</form>
```
---

## 6. Jalankan Project

```bash
npm run dev
```

Terminal akan menunjukkan:
![](/image/imgpg2.png)

---

## 7. Buka di Browser

Buka browser (Chrome, Firefox, Edge), ketik di search bar:

```
http://localhost:3000
```

Akan muncul halaman **List Data Mahasiswa**.

Di Form ini kita bisa tambah data, edit data, dan hapus data.

---
Halaman List Data Mahasiswa
![](/image/imgpg2.1.png)
---
Halaman Tambah Data Mahasiswa
![](/image/imgpg2.3.png)
---
Halaman Edit Data Mahasiswa
![](/image/imgpg2.4.png)
---
SELESAI.
---
