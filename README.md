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
```sql
CREATE TABLE courses (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    instructor VARCHAR(255) NOT NULL,
    duration INT NOT NULL,
    price FLOAT (10, 2) NOT NULL
);

INSERT INTO courses (title, instructor, duration, price) VALUES 
('Dasar-Dasar Basis Data', 'Dr. Budi Santoso', 40, 500000.00), 
('Pengembangan Web Dasar', 'Ibu Siti Aminah', 60, 350000.00), 
('Ilmu Data dengan Python', 'Dr. Ani Wulandari', 50, 750000.00), 
('Pembelajaran Mesin untuk Pemula', 'Bapak Joko Widodo', 45, 600000.00), 
('Keamanan Siber untuk Pemula', 'Ibu Rina Sari', 35, 450000.00);

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
```
## 3. Membuat File PHP untuk Web Service
1. Buka text editor Anda seperti Visual Studio Code
2. Buat file baru dan simpan sebagai courses.php di dalam folder courses.
3. Salin dan tempel kode berikut ke dalam courses.php:
```php
   <?php
header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With");

$method = $_SERVER['REQUEST_METHOD'];
$request = [];

if (isset($_SERVER['PATH_INFO'])) {
    $request = explode('/', trim($_SERVER['PATH_INFO'], '/'));
}

function getConnection() {
    $host = 'localhost';
    $db = 'courses'; // Nama database Anda
    $user = 'root';
    $pass = ''; // Ganti dengan password MySQL Anda jika ada
    $charset = 'utf8mb4';

    $dsn = "mysql:host=$host;dbname=$db;charset=$charset";
    $options = [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES => false,
    ];

    try {
        return new PDO($dsn, $user, $pass, $options);
    } catch (\PDOException $e) {
        throw new \PDOException($e->getMessage(), (int)$e->getCode());
    }
}

function response($status, $data = null) {
    header("HTTP/1.1 " . $status);
    if ($data) {
        echo json_encode($data);
    }
    exit();
}

$db = getConnection();

switch ($method) {
    case 'GET':
        if (!empty($request) && isset($request[0])) {
            $id = $request[0];
            $stmt = $db->prepare("SELECT * FROM courses WHERE id = ?");
            $stmt->execute([$id]);
            $course = $stmt->fetch();
            if ($course) {
                response(200, $course);
            } else {
                response(404, ["message" => "Course not found"]);
            }
        } else {
            $stmt = $db->query("SELECT * FROM courses");
            $courses = $stmt->fetchAll();
            response(200, $courses);
        }
        break;

    case 'POST':
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->title) || !isset($data->instructor) || !isset($data->duration) || !isset($data->price)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "INSERT INTO courses (title, instructor, duration, price) VALUES (?, ?, ?, ?)";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->title, $data->instructor, $data->duration, $data->price])) {
            response(201, ["message" => "Course created", "id" => $db->lastInsertId()]);
        } else {
            response(500, ["message" => "Failed to create course"]);
        }
        break;

    case 'PUT':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "Course ID is required"]);
        }
        $id = $request[0];
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->title) || !isset($data->instructor) || !isset($data->duration) || !isset($data->price)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "UPDATE courses SET title = ?, instructor = ?, duration = ?, price = ? WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->title, $data->instructor, $data->duration, $data->price, $id])) {
            response(200, ["message" => "Course updated"]);
        } else {
            response(500, ["message" => "Failed to update course"]);
        }
        break;

    case 'DELETE':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "Course ID is required"]);
        }
        $id = $request[0];
        $sql = "DELETE FROM courses WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$id])) {
            response(200, ["message" => "Course deleted"]);
        } else {
            response(500, ["message" => "Failed to delete course"]);
        }
        break;

    default:
        response(405, ["message" => "Method not allowed"]);
        break;
}
?>
   ```
4. Buatlah file index.php untuk halaman utama dan salin kode berikut :
```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Daftar Kursus</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body {
            background-color: #f0f4f8; /* Latar belakang yang lembut */
            font-family: 'Arial', sans-serif;
        }

        h1 {
            color: #2c3e50; /* Warna judul yang elegan */
            font-size: 2.5rem;
            text-align: center;
            margin-bottom: 30px;
            font-weight: 700;
        }

        .btn-group-action {
            white-space: nowrap;
        }

        /* Styling untuk tombol logout */
        .logout-btn {
            position: absolute;
            top: 10px;
            right: 20px;
            background-color: #e74c3c; /* Merah yang lembut */
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 8px;
            font-size: 16px;
            cursor: pointer;
            transition: background-color 0.3s ease, transform 0.2s ease;
        }

        .logout-btn:hover {
            background-color: #c0392b; /* Merah gelap saat hover */
            transform: scale(1.1);
        }

        .logout-btn:focus {
            outline: none;
        }

        /* Tabel Styling */
        .table th, .table td {
            text-align: center;
        }

        .table th {
            background-color: #3498db; /* Biru cerah untuk header */
            color: white;
        }

        .table td {
            background-color: #ffffff;
            border: 1px solid #ddd;
        }

        .table-hover tbody tr:hover {
            background-color: #ecf0f1; /* Latar belakang tabel saat hover */
        }

        /* Modal Styling */
        .modal-content {
            border-radius: 10px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
            background-color: #ffffff;
        }

        .modal-header {
            background-color: #3498db; /* Warna biru untuk header modal */
            color: white;
            border-top-left-radius: 10px;
            border-top-right-radius: 10px;
        }

        .form-label {
            font-weight: 600;
            color: #34495e;
        }

        .form-control {
            border-radius: 8px;
            box-shadow: none;
            border: 1px solid #bdc3c7;
            padding: 12px;
            transition: border-color 0.3s ease;
        }

        .form-control:focus {
            border-color: #3498db; /* Biru cerah saat focus */
            box-shadow: 0 0 8px rgba(52, 152, 219, 0.3);
        }

        /* Add space between action buttons */
        .btn-group-action button {
            margin-right: 12px;
            font-size: 14px;
            transition: transform 0.2s ease;
        }

        .btn-group-action button:hover {
            transform: scale(1.05);
        }

        /* Styling tombol tambah */
        .btn-success {
            background-color: #2ecc71; /* Hijau cerah untuk tombol tambah */
            border: none;
            color: white;
            border-radius: 8px;
            transition: background-color 0.3s ease;
        }

        .btn-success:hover {
            background-color: #27ae60; /* Hijau lebih gelap saat hover */
        }
    </style>
</head>
<body class="container py-4">
    <button class="logout-btn" onclick="window.location.href='logout.php'">Logout</button>
    <h1>Daftar Kursus</h1>
    
    <div class="row mb-3">
        <div class="col">
            <input type="text" id="searchInput" class="form-control" placeholder="Cari berdasarkan ID Kursus">
        </div>
        <div class="col-auto">
            <button onclick="searchCourse()" class="btn btn-primary">Cari</button>
        </div>
        <div class="col-auto">
            <button type="button" class="btn btn-success" data-bs-toggle="modal" data-bs-target="#courseModal">
                Tambah Kursus
            </button>
        </div>
    </div>

    <table class="table table-striped table-hover">
        <thead>
            <tr>
                <th>ID</th>
                <th>Judul</th>
                <th>Instruktur</th>
                <th>Durasi (Menit)</th>
                <th>Harga (Rp)</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody id="courseList">
        </tbody>
    </table>

    <!-- Modal for Add/Edit Course -->
    <div class="modal fade" id="courseModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="modalTitle">Tambah Kursus</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <div class="modal-body">
                    <form id="courseForm">
                        <input type="hidden" id="courseId">
                        <div class="mb-3">
                            <label for="title" class="form-label">Judul</label>
                            <input type="text" class="form-control" id="title" required>
                        </div>
                        <div class="mb-3">
                            <label for="instructor" class="form-label">Instruktur</label>
                            <input type="text" class="form-control" id="instructor" required>
                        </div>
                        <div class="mb-3">
                            <label for="duration" class="form-label">Durasi (Menit)</label>
                            <input type="number" class="form-control" id="duration" required>
                        </div>
                        <div class="mb-3">
                            <label for="price" class="form-label">Harga (Rp)</label>
                            <input type="number" step="0.01" class="form-control" id="price" required>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                    <button type="button" class="btn btn-primary" onclick="saveCourse()">Simpan</button>
                </div>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script>
        const API_URL = 'http://localhost/courses/courses.php';
        let courseModal;

        document.addEventListener('DOMContentLoaded', function() {
            courseModal = new bootstrap.Modal(document.getElementById('courseModal'));
            loadCourses();
        });

        function loadCourses() {
            fetch(API_URL)
                .then(response => response.json())
                .then(courses => {
                    const courseList = document.getElementById('courseList');
                    courseList.innerHTML = '';
                    courses.forEach(course => {
                        courseList.innerHTML += `
                            <tr>
                                <td>${course.id}</td>
                                <td>${course.title}</td>
                                <td>${course.instructor}</td>
                                <td>${course.duration}</td>
                                <td>${course.price}</td>
                                <td class="btn-group-action">
                                    <button class="btn btn-sm btn-warning me-1" onclick="editCourse(${course.id})">Edit</button>
                                    <button class="btn btn-sm btn-danger" onclick="deleteCourse(${course.id})">Hapus</button>
                                </td>
                            </tr>
                        `;
                    });
                })
                .catch(error => alert('Error loading courses: ' + error));
        }

        function searchCourse() {
            const id = document.getElementById('searchInput').value;
            if (!id) {
                loadCourses();
                return;
            }
            
            fetch(`${API_URL}/${id}`)
                .then(response => response.json())
                .then(course => {
                    const courseList = document.getElementById('courseList');
                    if (course.message) {
                        alert('Kursus tidak ditemukan');
                        return;
                    }
                    courseList.innerHTML = `
                        <tr>
                            <td>${course.id}</td>
                            <td>${course.title}</td>
                            <td>${course.instructor}</td>
                            <td>${course.duration}</td>
                            <td>${course.price}</td>
                            <td class="btn-group-action">
                                <button class="btn btn-sm btn-warning me-1" onclick="editCourse(${course.id})">Edit</button>
                                <button class="btn btn-sm btn-danger" onclick="deleteCourse(${course.id})">Hapus</button>
                            </td>
                        </tr>
                    `;
                })
                .catch(error => alert('Error searching course: ' + error));
        }

        function editCourse(id) {
            fetch(`${API_URL}/${id}`)
                .then(response => response.json())
                .then(course => {
                    document.getElementById('courseId').value = course.id;
                    document.getElementById('title').value = course.title;
                    document.getElementById('instructor').value = course.instructor;
                    document.getElementById('duration').value = course.duration;
                    document.getElementById('price').value = course.price;
                    document.getElementById('modalTitle').textContent = 'Edit Kursus';
                    courseModal.show();
                })
                .catch(error => alert('Error loading course details: ' + error));
        }

        function deleteCourse(id) {
            if (confirm('Apakah Anda yakin ingin menghapus kursus ini?')) {
                fetch(`${API_URL}/${id}`, {
                    method: 'DELETE'
                })
                .then(response => response.json())
                .then(data => {
                    alert('Kursus berhasil dihapus');
                    loadCourses();
                })
                .catch(error => alert('Error deleting course: ' + error));
            }
        }

        function saveCourse() {
            const courseId = document.getElementById('courseId').value;
            const courseData = {
                title: document.getElementById('title').value,
                instructor: document.getElementById('instructor').value,
                duration: document.getElementById('duration').value,
                price: document.getElementById('price').value
            };

            const method = courseId ? 'PUT' : 'POST';
            const url = courseId ? `${API_URL}/${courseId}` : API_URL;

            fetch(url, {
                method: method,
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(courseData)
            })
            .then(response => response.json())
            .then(data => {
                alert(courseId ? 'Kursus berhasil diperbarui' : 'Kursus berhasil ditambahkan');
                courseModal.hide();
                loadCourses();
                resetForm();
            })
            .catch(error => alert('Error saving course: ' + error));
        }

        function resetForm() {
            document.getElementById('courseId').value = '';
            document.getElementById('courseForm').reset();
            document.getElementById('modalTitle').textContent = 'Tambah Kursus';
        }

        // Reset form when modal is closed
        document.getElementById('courseModal').addEventListener('hidden.bs.modal', resetForm);
    </script>
</body>
</html>
<div class="footer">
        <p>&copy; 2025 Daftar Kursus. Salsa Bila Naqiyyah.</p>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script>
        // JavaScript functions here (same as in your code)
    </script>
</body>
</html>
```
5. Buatlah file login.php dan logout.php, salin kode berikut :  
   LOGIN
```php
   <?php
session_start();
require 'connection.php'; // File koneksi database

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $username = $_POST['username'];
    $password = $_POST['password'];

    $sql = "SELECT * FROM users WHERE username = ?";
    $stmt = $db->prepare($sql);
    $stmt->execute([$username]);
    $user = $stmt->fetch();

    if ($user && password_verify($password, $user['password'])) {
        // Jika password benar
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['username'] = $user['username'];
        header("Location: index.php");
        exit();
    } else {
        // Jika username/password salah
        $error = "Username atau password salah.";
    }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background: linear-gradient(135deg, #3498db, #8e44ad);
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            color: #fff;
        }
        .login-box {
            width: 360px;
            background-color: #fff;
            padding: 40px 30px;
            border-radius: 10px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
            text-align: center;
            color: #333;
        }
        h1 {
            font-size: 28px;
            margin-bottom: 20px;
            color: #333;
        }
        input[type="text"], input[type="password"] {
            width: 100%;
            padding: 12px;
            margin: 10px 0;
            border: 1px solid #ddd;
            border-radius: 5px;
            box-sizing: border-box;
            transition: border-color 0.3s ease, box-shadow 0.3s ease;
        }
        input[type="text"]:focus, input[type="password"]:focus {
            border-color: #3498db;
            box-shadow: 0 0 8px rgba(52, 152, 219, 0.5);
            outline: none;
        }
        button {
            width: 100%;
            padding: 12px;
            background: linear-gradient(135deg, #3498db, #8e44ad);
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            font-weight: bold;
            margin-top: 10px;
            transition: background 0.3s ease;
        }
        button:hover {
            background: linear-gradient(135deg, #8e44ad, #3498db);
        }
        p {
            font-size: 14px;
            margin-top: 15px;
            color: #666;
        }
        a {
            color: #3498db;
            text-decoration: none;
            font-weight: bold;
            transition: color 0.3s ease;
        }
        a:hover {
            color: #8e44ad;
        }
        .error-message {
            color: red;
            font-size: 14px;
            margin-bottom: 10px;
        }
    </style>
</head>
<body>
    <div class="login-box">
        <h1>Login</h1>
        <?php if (isset($error)) { echo "<p class='error-message'>$error</p>"; } ?>
        <form method="POST">
            <input type="text" id="username" name="username" placeholder="Username" required>
            <input type="password" id="password" name="password" placeholder="Password" required>
            <button type="submit">Login</button>
        </form>
        <p>Belum punya akun? <a href="register.php">Registrasi di sini</a>.</p>
    </div>
</body>
</html>
   ```
   LOGOUT
```php
   <?php
session_start();
session_destroy();
header("Location: login.php");
exit();
?>
```
6. Buatlah file register.php dan salin kode berikut :
```php
<?php
require 'users.php'; // Pastikan file koneksi benar
$db = getConnection();

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $username = $_POST['username'] ?? '';
    $email = $_POST['email'] ?? '';
    $alamat = $_POST['alamat'] ?? '';
    $password = $_POST['password'] ?? '';
    $re_password = $_POST['re_password'] ?? '';

    // Validasi password dan re-password
    if ($password !== $re_password) {
        $error = "Password dan Re-password tidak cocok.";
    } else {
        // Periksa apakah username sudah ada
        $sql_check_username = "SELECT COUNT(*) FROM users WHERE username = ?";
        $stmt_check_username = $db->prepare($sql_check_username);
        $stmt_check_username->execute([$username]);
        $username_exists = $stmt_check_username->fetchColumn();

        if ($username_exists > 0) {
            $error = "Username sudah digunakan. Silakan gunakan username lain.";
        } else {
            // Periksa apakah email sudah ada
            $sql_check_email = "SELECT COUNT(*) FROM users WHERE email = ?";
            $stmt_check_email = $db->prepare($sql_check_email);
            $stmt_check_email->execute([$email]);
            $email_exists = $stmt_check_email->fetchColumn();

            if ($email_exists > 0) {
                $error = "Email sudah terdaftar. Silakan gunakan email lain.";
            } else {
                // Simpan data jika username dan email belum ada
                $hashed_password = password_hash($password, PASSWORD_BCRYPT);
                $sql = "INSERT INTO users (username, email, alamat, password) VALUES (?, ?, ?, ?)";
                $stmt = $db->prepare($sql);

                if ($stmt->execute([$username, $email, $alamat, $hashed_password])) {
                    header("Location: login.php"); // Redirect ke halaman login
                    exit();
                } else {
                    $error = "Terjadi kesalahan saat registrasi.";
                }
            }
        }
    }
}
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Form Registrasi</title>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500&display=swap" rel="stylesheet">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css" rel="stylesheet">
    <style>
        body {
            font-family: 'Roboto', sans-serif;
            background: linear-gradient(135deg, #2c3e50, #34495e, #2d3436);
            background-size: 400% 400%;
            animation: gradientAnimation 8s ease infinite;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }

        @keyframes gradientAnimation {
            0% { background-position: 0% 50%; }
            50% { background-position: 100% 50%; }
            100% { background-position: 0% 50%; }
        }

        .container {
            width: 100%;
            max-width: 450px;
            background-color: white;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
            padding: 35px 25px;
            box-sizing: border-box;
        }

        h1 {
            text-align: center;
            color: #34495e;
            margin-bottom: 30px;
            font-size: 30px;
            font-weight: 600;
        }

        .input-group {
            margin: 20px 0;
            position: relative;
        }

        .input-group label {
            font-size: 15px;
            color: #555;
            margin-bottom: 10px;
            display: block;
            transition: color 0.3s;
        }

        .input-group input {
            width: 100%;
            padding: 14px 40px 14px 14px;
            border: 1px solid #ddd;
            border-radius: 10px;
            background-color: #f4f6f9;
            font-size: 15px;
            transition: border-color 0.3s, box-shadow 0.3s;
            box-sizing: border-box;
        }

        .input-group input:focus {
            border-color: #3498db;
            box-shadow: 0 0 8px rgba(52, 152, 219, 0.4);
            outline: none;
        }

        .input-group i {
            position: absolute;
            top: 67%;
            right: 15px;
            transform: translateY(-50%);
            cursor: pointer;
            color: #888;
            transition: color 0.3s;
        }

        .input-group i:hover {
            color: #3498db;
        }

        button {
            width: 100%;
            padding: 14px;
            background-color: #3498db;
            border: none;
            color: white;
            font-size: 16px;
            font-weight: 600;
            border-radius: 10px;
            cursor: pointer;
            transition: background-color 0.3s ease, box-shadow 0.3s ease;
        }

        button:hover {
            background-color: #2980b9;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
        }

        p {
            text-align: center;
            font-size: 14px;
            color: #555;
        }

        a {
            color: #3498db;
            text-decoration: none;
            font-weight: 600;
        }

        a:hover {
            text-decoration: underline;
        }

        .error-message {
            color: #e74c3c;
            text-align: center;
            margin-bottom: 20px;
            font-size: 14px;
        }
    </style>
    <script>
        function togglePasswordVisibility(id, iconId) {
            const input = document.getElementById(id);
            const icon = document.getElementById(iconId);
            if (input.type === 'password') {
                input.type = 'text';
                icon.classList.remove('fa-eye');
                icon.classList.add('fa-eye-slash');
            } else {
                input.type = 'password';
                icon.classList.remove('fa-eye-slash');
                icon.classList.add('fa-eye');
            }
        }
    </script>
</head>
<body>
    <div class="container">
        <h1>Form Registrasi</h1>
        <?php if (!empty($error)) echo "<p class='error-message'>$error</p>"; ?>
        <form action="register.php" method="POST">
            <div class="input-group">
                <label for="username">Username:</label>
                <input type="text" id="username" name="username" placeholder="Masukkan username" required>
            </div>

            <div class="input-group">
                <label for="email">Email:</label>
                <input type="email" id="email" name="email" placeholder="Masukkan email" required>
            </div>

            <div class="input-group">
                <label for="password">Password:</label>
                <input type="password" id="password" name="password" placeholder="Masukkan password" required>
                <i id="togglePassword" class="fas fa-eye" onclick="togglePasswordVisibility('password', 'togglePassword')"></i>
            </div>

            <div class="input-group">
                <label for="re_password">Re-Password:</label>
                <input type="password" id="re_password" name="re_password" placeholder="Ulangi password" required>
                <i id="toggleRePassword" class="fas fa-eye" onclick="togglePasswordVisibility('re_password', 'toggleRePassword')"></i>
            </div>

            <button type="submit">Register</button>
        </form>
        <p>Sudah punya akun? <a href="login.php">Login di sini</a></p>
    </div>
</body>
</html>
```
7. Buatlah file users.php dan salin kode berikut :
```php
<?php
function getConnection() {
    $host = 'localhost';
    $db = 'courses'; // Nama database Anda
    $user = 'root';
    $pass = ''; // Password MySQL Anda
    $charset = 'utf8mb4';

    $dsn = "mysql:host=$host;dbname=$db;charset=$charset";
    $options = [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES => false,
    ];

    try {
        return new PDO($dsn, $user, $pass, $options);
    } catch (\PDOException $e) {
        throw new \PDOException($e->getMessage(), (int)$e->getCode());
    }
}
?>
```
8. Buatlah file connection.php untuk koneksi ke database dan salin kode berikut :
```php
<?php
$host = 'localhost';
$db = 'courses';
$user = 'root';
$pass = '';
$charset = 'utf8mb4';

$dsn = "mysql:host=$host;dbname=$db;charset=$charset";
$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES => false,
];

try {
    $db = new PDO($dsn, $user, $pass, $options);
} catch (\PDOException $e) {
    die("Koneksi gagal: " . $e->getMessage());
}
```
## 3. Pengujian Web
1. Buka Chrome dan salin alamat ini
``` php
http://localhost/courses/
```
