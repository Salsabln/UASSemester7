#  Project Ujian Akhir Semester 7
Nama : Salsa Bila Naqiyyah
NIM : 21.01.55.0018
#  Deskripsi Project
Project Ujian Akhir Semester 7 membuat sebuah web khursus dari halaman Login, Registrasi serta Halaman Utama Daftar Khursus yang di lengkapi dengan menu tambah khursus, edit, hapus dan cari. Tujuan project ini yaitu untuk memenuhi tugas Ujian Akhir Semester 7
#  Alat Yang Dibutuhkah
1. XAMPP (atau server web lain dengan PHP dan MySQL)
2. Text editor (misalnya Visual Studio Code, Notepad++, dll)
#  Cara Instalsasi dan Penggunaan
## 1. Persiapan Lingkungan
1. Jalankan XAMPP Control Panel dan aktifkan Apache dan MySQL
2. Buat folder baru bernama courses di dalam direktori htdocs XAMPP Anda.
## 2. Membuat Database
1. Buka phpMyAdmin http://localhost/phpmyadmin
2. Buat database baru bernama courses
3. Pilih database courses, lalu buka tab SQL
4. Jalankan query SQL berikut untuk membuat tabel dan menambahkan data sampel:
``
CREATE TABLE `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) NOT NULL,
  `email` varchar(100) NOT NULL,
  `alamat` text NOT NULL,
  `password` varchar(255) NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`),
  UNIQUE KEY `email` (`email`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

CREATE TABLE `courses` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(255) NOT NULL,
  `instructor` varchar(255) NOT NULL,
  `duration` int(11) NOT NULL COMMENT 'Duration in hours',
  `price` decimal(10,2) NOT NULL COMMENT 'Price in USD',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
