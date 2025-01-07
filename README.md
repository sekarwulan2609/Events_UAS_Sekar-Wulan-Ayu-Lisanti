# UAS_21.01.55.0005_SEKAR-WULAN-AYU-LISANTI #

### CONFIG.PHP ###
```
<?php
$host = "localhost";
$dbuser = "root";
$dbpass = "";
$dbname = "events";

$conn = mysqli_connect($host, $dbuser, $dbpass, $dbname);
if (!$conn) {
    die("Connection failed: " . mysqli_connect_error());
}
?>
```
### EVENTS_API ###
```
<?php
header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With");

$method = $_SERVER['REQUEST_METHOD'];
$request = [];

if (isset($_SERVER['PATH_INFO'])) {
    $request = explode('/', trim($_SERVER['PATH_INFO'],'/'));
}

function getConnection() {
    $host = 'localhost';
    $db   = 'events';
    $user = 'root';
    $pass = ''; // Ganti dengan password MySQL Anda jika ada
    $charset = 'utf8mb4';

    $dsn = "mysql:host=$host;dbname=$db;charset=$charset";
    $options = [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ];
    try {
        return new PDO($dsn, $user, $pass, $options);
    } catch (\PDOException $e) {
        throw new \PDOException($e->getMessage(), (int)$e->getCode());
    }
}

function response($status, $data = NULL) {
    header("HTTP/1.1 " . $status);
    if ($data) {
        echo json_encode($data);
    }
    exit();
}
function validateevents($name, $date, $location, $price) {
    $errors = [];
    if (empty($name)) {
        $errors[] = "name is required";
    }
    if (empty($date)) {
        $errors[] = "date is required";
    }
    if (empty($location)) {
        $errors[] = "location is required";
    }
    if (empty($price)) {
        $errors[] = "price is required";
    }
    return $errors;
}
$db = getConnection();

switch ($method) {
    case 'GET':
        if (!empty($request) && isset($request[0])) {
            $id = $request[0];
            $stmt = $db->prepare("SELECT * FROM events WHERE id = ?");
            $stmt->execute([$id]);
            $events = $stmt->fetch();
            if ($events) {
                response(200, $events);
            } else {
                response(404, ["message" => "events not found"]);
            }
        } else {
            $stmt = $db->query("SELECT * FROM events");
            $events = $stmt->fetchAll();
            response(200, $events);
        }
        break;
        // if (!empty($request) && isset($request[0])) {
        //     if ($request[0] === 'search') {
        //         // 5.1 Search functionality
        //         $searchTerm = $_GET['term'] ?? '';
        //         $stmt = $db->prepare("SELECT * FROM events WHERE name LIKE ? OR location LIKE ?");
        //         $searchTerm = "%$searchTerm%";
        //         $stmt->execute([$searchTerm, $searchTerm]);
        //         $events = $stmt->fetchAll();
        //         response(200, $events);
        //     } else {
        //         // Get specific events
        //         $id = $request[0];
        //         $stmt = $db->prepare("SELECT * FROM events WHERE id = ?");
        //         $stmt->execute([$id]);
        //         $events = $stmt->fetch();
        //         if ($events) {
        //             response(200, $events);
        //         } else {
        //             response(404, ["message" => "events not found"]);
        //         }
        //     }
        // } else {
        //     // 5.2 Pagination
        //     $page = isset($_GET['page']) ? (int)$_GET['page'] : 1;
        //     $limit = isset($_GET['limit']) ? (int)$_GET['limit'] : 10;
        //     $offset = ($page - 1) * $limit;

        //     $stmt = $db->prepare("SELECT * FROM events LIMIT ? OFFSET ?");
        //     $stmt->bindValue(1, $limit, PDO::PARAM_INT);
        //     $stmt->bindValue(2, $offset, PDO::PARAM_INT);
        //     $stmt->execute();
        //     $events = $stmt->fetchAll();

        //     $totalStmt = $db->query("SELECT COUNT(*) FROM events");
        //     $total = $totalStmt->fetchColumn();

        //     response(200, [
        //         'events' => $events,
        //         'total' => $total,
        //         'page' => $page,
        //         'limit' => $limit
        //     ]);
        // }
        // break;
    
    case 'POST':
        $data = json_decode(file_get_contents("php://input"));
        $errors = validateevents($data->name ?? '', $data->date ?? '', $data->location ?? '', $data->price ?? '' );
        if (!empty($errors)) {
            response(400, ["errors" => $errors]);
        }
        $sql = "INSERT INTO events (name, date, location, price) VALUES (?, ?, ?, ?)";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->name, $data->date, $data->location, $data->price])) {
            response(201, ["message" => "events created", "id" => $db->lastInsertId()]);
        } else {
            response(500, ["message" => "Failed to create events"]);
        }
        break;
    
    case 'PUT':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "events ID is required"]);
        }
        $id = $request[0];
        $data = json_decode(file_get_contents("php://input"));
        $errors = validateevents($data->name ?? '', $data->date ?? '', $data->location ?? '', $data->price ?? '');
        if (!empty($errors)) {
            response(400, ["errors" => $errors]);
        }
        $sql = "UPDATE events SET name = ?, date = ?, location = ?, price = ? WHERE id =?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->name, $data->date, $data->location, $data->price, $id])) {
            response(200, ["message" => "events updated"]);
        } else {
            response(500, ["message" => "Failed to update events"]);
        }
        break;
    
    case 'DELETE':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "events ID is required"]);
        }
        $id = $request[0];
        $sql = "DELETE FROM events WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$id])) {
            response(200, ["message" => "events deleted"]);
        } else {
            response(500, ["message" => "Failed to delete events"]);
        }
        break;
    
    default:
        response(405, ["message" => "Method not allowed"]);
        break;
}
?>
```
### LOGIN.PHP ###
```
<?php
session_start();
require_once 'config.php';

if(isset($_POST['login'])) {
    $username = $_POST['username'];
    $password = $_POST['password'];
    
    // Gunakan fungsi PASSWORD() untuk mencocokkan dengan hasil di database
    $query = "SELECT * FROM users WHERE username='$username' AND password=PASSWORD('$password')";
    $result = mysqli_query($conn, $query);
    
    if(mysqli_num_rows($result) == 1) {
        $row = mysqli_fetch_assoc($result);
        $_SESSION['user_id'] = $row['id'];
        $_SESSION['username'] = $row['username'];
        $_SESSION['name'] = $row['name'];
            
        header("Location: dashboard.php");
        exit();
    } else {
        $error = "Username atau Password salah!";
    }
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
    <style>
        body {
            background-color: #e6e6fa; /* Warna latar belakang ungu muda */
            font-family: Arial, sans-serif; /* Font yang lebih modern */
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh; /* Mengisi tinggi viewport */
            margin: 0;
            color: #333; /* Warna teks default */
        }
        .login-container {
            background-color: rgba(255, 255, 255, 0.9); /* Warna latar belakang untuk formulir dengan transparansi */
            padding: 20px;
            border-radius: 8px; /* Sudut yang membulat */
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2); /* Bayangan untuk efek kedalaman */
            width: 300px; /* Lebar tetap untuk formulir */
        }
        h1 {
            text-align: center; /* Pusatkan judul */
            color: #6f42c1; /* Warna teks judul ungu */
            margin-bottom: 20px; /* Jarak bawah judul */
        }
        p {
            margin: 10px 0; /* Jarak antar elemen */
        }
        input[type="text"],
        input[type="password"] {
            width: 100%; /* Lebar penuh */
            padding: 10px; /* Padding untuk ruang dalam */
            border: 1px solid #ccc; /* Border abu-abu */
            border-radius: 5px; /* Sudut yang membulat */
            box-sizing: border-box; /* Menghitung padding dan border dalam lebar */
        }
        input[type="submit"] {
            background-color: #6f42c1; /* Warna latar belakang tombol ungu */
            color: white; /* Warna teks tombol */
            border: none; /* Tanpa border */
            padding: 10px; /* Padding untuk tombol */
            border-radius: 5px; /* Sudut yang membulat */
            cursor: pointer; /* Kursor pointer saat hover */
            width: 100%; /* Lebar penuh */
            transition: background-color 0.3s; /* Transisi untuk efek hover */
        }
        input[type="submit"]:hover {
            background-color: #5a31a8; /* Warna saat hover */
        }
        .error {
            color: red; /* Warna merah untuk pesan error */
            text-align: center; /* Pusatkan pesan error */
        }
    </style>
</head>
<body>
<div class="login-container">
        <h1>Login Form</h1> <!-- Judul dipindahkan ke sini -->
        <?php if(isset($error)) { ?>
            <p class="error"><?php echo $error; ?></p>
        <?php } ?>
    
    <form action="" method="POST">
        <p>
            Username: <input type="text" name="username" required>
        </p>
        <p>
            Password: <input type="password" name="password" required>
        </p>
        <p>
            <input type="submit" name="login" value="Login">
        </p>
    </form>
</body>
</html>
```
### DASHBOARD.PHP ###
```
<?php
session_start();

// Cek apakah user sudah login
if(!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Daftar events</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <!-- <style>
        .btn-group-action {
            white-space: nowrap;
        }
    </style> -->
    <style>
        body {
            background-color: #e6d8f0; /* Warna latar belakang ungu */
            color: rgb(5, 5, 5); /* Mengubah warna teks menjadi putih untuk kontras yang lebih baik */
        }
         /* Styling Navbar */
    .navbar {
        background-color: #6f42c1; /* Dark Blue */
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        width: 100%; /* Memastikan navbar memenuhi layar penuh */
        z-index: 1000;
        padding: 15px 0; /* Padding navbar */
    }

    .navbar-brand {
        font-size: 2rem;
        font-weight: bold;
        color: white !important;
    }

    .navbar-brand:hover {
        color: #ffcc00 !important;
    }

    .navbar-nav .nav-link {
        color: white !important;
        font-size: 1.2rem;
        padding: 12px 25px;
        transition: background-color 0.3s ease;
    }

    /* Hover effect for navbar links */
    .navbar-nav .nav-link:hover {
        color: #ffcc00 !important;
        background-color: #00509e;
        border-radius: 5px;
    }

    .navbar-nav .nav-link.logout-btn {
        background-color: #dc3545;  /* Red color for logout */
        color: white;
        padding: 10px 25px;
        border-radius: 25px;
        font-size: 1.2rem;
        transition: background-color 0.3s ease;
    }

    .navbar-nav .nav-link.logout-btn:hover {
        background-color: #c82333;
    }

    /* Container - Full width */
    .container {
        width: 100%; /* Memastikan container menggunakan lebar penuh */
        padding: 20px;
        box-sizing: border-box; /* Menyesuaikan padding tanpa mempengaruhi ukuran */
    }

    h1 {
        font-size: 3rem;
        color: #495057;
        text-align: center;
        margin-bottom: 40px;
    }

    /* Card */
    .card {
        border-radius: 15px;
        box-shadow: 0 2px 15px rgba(0, 0, 0, 0.1);
        margin-bottom: 40px;
    }
        h1 {
            margin-bottom: 20px;
        }
        /* Table */
    .table {
        background-color: white;
        border-radius: 10px;
        overflow: hidden;
        width: 100%; /* Memastikan tabel memenuhi lebar layar */
    }

    .table th, .table td {
        vertical-align: middle;
        padding: 15px 20px; /* Padding tabel agar lebih luas */
    }

    .table th {
        background-color: #003366;
        color: white;
    }

    .table td {
        background-color: #f8f9fa;
    }

        th, td {
            text-align: center;
        }
        .btn-group-action {
            white-space: nowrap;
        }
        .btn {
            transition: background-color 0.3s, transform 0.3s;
        }
        .btn:hover {
            transform: translateY(-2px);
        }
        .modal-content {
            border-radius: 8px;
            background-color: #f8f9fa; /* Warna latar belakang untuk modal */
        }
        .modal-header {
            background-color: #007bff;
            color: white;
        }
        .form-control {
            border-radius: 5px;
        }
        .form-control:focus {
            box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
            border-color: #80bdff;
        }
        .modal-footer .btn {
            border-radius: 5px;
        }
    </style>

</head>
<body class="container py-4">
     <!-- Navbar with Home Link -->
   <nav class="navbar navbar-expand-lg navbar-light">
    <a class="navbar-brand" href="dashboard.php">Daftar Events</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarNav">
        <ul class="navbar-nav ms-auto">
            <li class="nav-item">
                <a class="nav-link active" href="home.php">Home</a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="dashboard.php">Daftar Events</a>
            </li>
            <!-- <li class="nav-item">
                <a class="nav-link" href="contact.php">Kontak</a>
            </li> -->
            <!-- Tambahkan tombol logout di sini -->
            <li class="nav-item">
                <a class="nav-link logout-btn" href="javascript:logout()">Logout</a>
            </li>
        </ul>
    </div>
</nav>
<h1>Daftar events</h1>  
    <h1>Selamat Datang di Daftar Events</h1>

    <div class="card p-4 mb-4">
        <h5>"Temukan ribuan acara seru di sekitarmu!"</h5>
        <p> Jelajahi daftar lengkap event menarik, mulai dari konser musik hingga workshop kreatif. Cari berdasarkan nama event, tanggal, atau lokasi. Tambahkan event baru, edit detail, atau hapus yang sudah lewat.</p>
    </div>
 <!-- <div class="card p-4 mb-4">
        <div class="search-bar">
            <div class="col">
                <input type="text" id="searchInput" class="form-control" placeholder="Cari berdasarkan ID">
            </div>
            <div class="btn-group">
                <button onclick="searchClinic()" class="btn btn-primary">Cari</button>
                <button type="button" class="btn btn-success" data-bs-toggle="modal" data-bs-target="#clinicModal">
                    Tambah Klinik
                </button>
            </div>
        </div>
    </div> -->
    
    
    <div class="row mb-3">
        <div class="col">
            <input type="text" id="searchInput" class="form-control" placeholder="Cari berdasarkan ID">
        </div>
        <div class="col-auto">
            <button onclick="searchevents()" class="btn btn-primary">Cari</button>
        </div>
        <div class="col-auto">
            <button type="button" class="btn btn-success" data-bs-toggle="modal" data-bs-target="#eventsModal">
                Tambah events
            </button>
        </div>
    </div>

    <table class="table table-striped">
        <thead class="table-dark">
            <tr>
                <th>ID</th>
                <th>Nama Events</th>
                <th>Date</th>
                <th>Location</th>
                <th>Price</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody id="eventsList">
        </tbody>
    </table>

    <!-- Modal untuk Tambah/Edit events -->
    <div class="modal fade" id="eventsModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="modalTitle">Tambah events</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <div class="modal-body">
                    <form id="eventsForm">
                        <input type="hidden" id="eventsId">
                        <div class="mb-3">
                            <label for="name" class="form-label">Nama</label>
                            <input type="text" class="form-control" id="name" required>
                        </div>
                        <div class="mb-3">
                            <label for="date" class="form-label">Date</label>
                            <input type="date" class="form-control" id="date" required>
                        </div>
                        <div class="mb-3">
                            <label for="location" class="form-label">Location</label>
                            <input type="text" class="form-control" id="location" required>
                        </div>
                        <div class="mb-3">
                            <label for="price" class="form-label">Price</label>
                            <input type="number" class="form-control" id="price" required>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                    <button type="button" class="btn btn-primary" onclick="saveevents()">Simpan</button>
                </div>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script>
        const API_URL = 'http://localhost/events/events_api.php';
        let eventsModal;

        document.addEventListener('DOMContentLoaded', function() {
            eventsModal = new bootstrap.Modal(document.getElementById('eventsModal'));
            loadevents(); // Memuat semua events saat halaman pertama kali dibuka
        });

        function loadevents() {
            fetch(API_URL)
                .then(response => response.json())
                .then(events => {
                    const eventsList = document.getElementById('eventsList');
                    eventsList.innerHTML = '';
                    events.forEach(events => {
                        eventsList.innerHTML += 
                            `<tr>
                                <td>${events.id}</td>
                                <td>${events.name}</td>
                                <td>${events.date}</td>
                                <td>${events.location}</td>
                                <td>${events.price}</td>
                                <td class="btn-group-action">
                                    <button class="btn btn-sm btn-warning me-1" onclick="editevents(${events.id})">Edit</button>
                                    <button class="btn btn-sm btn-danger" onclick="deleteevents(${events.id})">Hapus</button>
                                </td>
                            </tr>`;
                    });
                })
                .catch(error => alert('Error loading events: ' + error));
        }
        function logout() {
            // Logic for logging out (clear session, redirect, etc.)
            alert('You have been logged out');
            window.location.href = 'login.php';  // Redirect to login page
        }

        function searchevents() {
            const id = document.getElementById('searchInput').value.trim();
            if (!id) {
                loadevents(); // Jika input ID kosong, tampilkan semua events
                return;
            }

            // URL untuk mencari events berdasarkan ID
            const url = `${API_URL}/${id}`;

            fetch(url)
                .then(response => response.json())
                .then(events => {
                    const eventsList = document.getElementById('eventsList');
                    eventsList.innerHTML = '';

                    if (events.message) {
                        alert('events not found');
                        return;
                    }

                    eventsList.innerHTML = 
                        `<tr>
                            <td>${events.id}</td>
                            <td>${events.name}</td>
                            <td>${events.date}</td>
                            <td>${events.location}</td>
                            <td>${events.price}</td>
                            <td class="btn-group-action">
                                <button class="btn btn-sm btn-warning me-1" onclick="editevents(${events.id})">Edit</button>
                                <button class="btn btn-sm btn-danger" onclick="deleteevents(${events.id})">Hapus</button>
                            </td>
                        </tr>`;
                })
                .catch(error => alert('Error searching events: ' + error));
        }

        function editevents(id) {
            fetch(`${API_URL}/${id}`)
                .then(response => response.json())
                .then(events => {
                    document.getElementById('eventsId').value = events.id;
                    document.getElementById('name').value = events.name;
                    document.getElementById('date').value = events.date;
                    document.getElementById('location').value = events.location;
                    document.getElementById('price').value = events.price;
                    document.getElementById('modalTitle').textContent = 'Edit events';
                    eventsModal.show();
                })
                .catch(error => alert('Error loading events details: ' + error));
        }

        function deleteevents(id) {
            if (confirm('Are you sure you want to delete this events?')) {
                fetch(`${API_URL}/${id}`, { method: 'DELETE' })
                    .then(response => response.json())
                    .then(data => {
                        alert('events deleted successfully');
                        loadevents();
                    })
                    .catch(error => alert('Error deleting events: ' + error));
            }
        }

        function saveevents() {
            const eventsId = document.getElementById('eventsId').value;
            const eventsData = {
                name: document.getElementById('name').value,
                date: document.getElementById('date').value,
                location: document.getElementById('location').value,
                price: document.getElementById('price').value
            };

            const method = eventsId ? 'PUT' : 'POST';
            const url = eventsId ? `${API_URL}/${eventsId}` : API_URL;

            fetch(url, {
                method: method,
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(eventsData)
            })
            .then(response => response.json())
            .then(data => {
                alert(eventsId ? 'events updated successfully' : 'events added successfully');
                eventsModal.hide();
                loadevents();
                resetForm();
            })
            .catch(error => alert('Error saving events: ' + error));
        }

        function resetForm() {
            document.getElementById('eventsId').value = '';
            document.getElementById('eventsForm').reset();
            document.getElementById('modalTitle').textContent = 'Tambah events';
        }

        // Reset form when modal is closed
        document.getElementById('eventsModal').addEventListener('hidden.bs.modal', resetForm);
    </script>
</body>
</html>
    <!-- Bootstrap JS -->
    <!-- <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    <a href="logout.php">Logout</a> -->
</body>
</html>
```
### HOME.PHP ###
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Home - Deskripsi Events</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body {
            background-color: #e6d8f0; /* Warna latar belakang ungu muda */
            color: #343a40; /* Warna teks gelap */
        }
         /* Styling Navbar */
    .navbar {
        background-color:#6f42c1; /* Dark purple */
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        width: 100%; /* Memastikan navbar memenuhi layar penuh */
        z-index: 1000;
        padding: 15px 0; /* Padding navbar */
    }

    .navbar-brand {
        font-size: 2rem;
        font-weight: bold;
        color: white !important;
    }

    .navbar-brand:hover {
        color: #ffcc00 !important;
    }

    .navbar-nav .nav-link {
        color: white !important;
        font-size: 1.2rem;
        padding: 12px 25px;
        transition: background-color 0.3s ease;
    }

    /* Hover effect for navbar links */
    .navbar-nav .nav-link:hover {
        color: #ffcc00 !important;
        background-color: #00509e;
        border-radius: 5px;
    }

    .navbar-nav .nav-link.logout-btn {
        background-color: #dc3545;  /* Red color for logout */
        color: white;
        padding: 10px 25px;
        border-radius: 25px;
        font-size: 1.2rem;
        transition: background-color 0.3s ease;
    }

    .navbar-nav .nav-link.logout-btn:hover {
        background-color: #c82333;
    }

        h1, h2 {
            margin-bottom: 20px;
        }
        .card {
            margin-bottom: 20px;
            border: none; /* Menghilangkan border default */
            border-radius: 10px; /* Membuat sudut kartu lebih bulat */
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* Menambahkan bayangan */
        }
        .card-title {
            color: #6f42c1; /* Warna ungu untuk judul kartu */
        }
        /* .navbar {
            background-color: #6f42c1; /* Warna navbar ungu */
        
        .navbar-brand, .nav-link {
            color: white !important; /* Mengubah warna teks navbar menjadi putih */
        }
        .navbar-brand:hover, .nav-link:hover {
            color: #d1c4e9 !important; /* Warna saat hover */
        } */
        .btn-primary {
            background-color: #6f42c1; /* Warna tombol */
            border: none; /* Menghilangkan border */
        }
        .btn-primary:hover {
            background-color: #5a31a1; /* Warna tombol saat hover */
        }
    </style>
</head>
<!-- <body> -->
    <!-- Navbar -->
    <body class="container py-4">
     <!-- Navbar with Home Link -->
   <nav class="navbar navbar-expand-lg navbar-light">
    <a class="navbar-brand" href="#">Daftar Events</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarNav">
        <ul class="navbar-nav ms-auto">
            <li class="nav-item">
                <a class="nav-link active" href="home.php">Home</a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="dashboard.php">Daftar Events</a>
            </li>
            <!-- <li class="nav-item">
                <a class="nav-link" href="contact.php">Kontak</a>
            </li> -->
            <!-- Tambahkan tombol logout di sini -->
            <li class="nav-item">
                <a class="nav-link logout-btn" href="javascript:logout()">Logout</a>
            </li>
        </ul>
    </div>
</nav>


    <div class="container py-4">
        <h2 class="text-center">Selamat Datang di Events</h2>
        <h2 class="text-center">Deskripsi Events</h2>
        
        <div class="card">
            <div class="card-body">
                <h5 class="card-title">Apa itu Events?</h5>
                 <p class="card-text">
                    Events adalah kegiatan yang dirancang untuk mengumpulkan orang-orang dengan tujuan tertentu, seperti seminar, konser, festival, dan banyak lagi. Kami menyediakan platform untuk mengelola dan mengorganisir berbagai jenis events agar lebih mudah diakses oleh semua orang.
                </p>
            </div>
        </div>
        <div class="card">
            <div class="card-body">
                <h5 class="card-title">Tujuan Kami</h5>
                <p class="card-text">
                    Tujuan kami adalah untuk memberikan pengalaman yang menyenangkan dan bermanfaat bagi semua peserta. Kami berkomitmen untuk menyelenggarakan events yang berkualitas tinggi dan memberikan nilai tambah bagi semua yang terlibat.
                </p>
            </div>
        </div>
        <div class="card">
            <div class="card-body">
                <h5 class="card-title">Cara Berpartisipasi</h5>
                <p class="card-text">
                    Anda dapat berpartisipasi dalam events kami dengan mendaftar melalui halaman daftar events. Pastikan untuk memeriksa jadwal dan lokasi untuk setiap event. Kami juga menyediakan informasi lebih lanjut tentang cara berpartisipasi di setiap event.
                </p>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script>
        function logout() {
            // Logic for logging out (clear session, redirect, etc.)
            alert('You have been logged out');
            window.location.href = 'login.php';  // Redirect to login page
        }
    </script>
</body>
</html>
```
### LOG OUT.PHP ###
```
<?php
session_start();  // Mulai sesi jika belum dimulai

// Hapus semua variabel sesi
session_unset();

// Hapus sesi itu sendiri
session_destroy();

// Kirimkan respons sukses
echo json_encode(['success' => true]);
?>

```
