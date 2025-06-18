// 📁 db.php

<?php
$conn = new mysqli('localhost', 'root', 'root', 'exam_site');

if ($conn->connect_error) {
    die('Ошибка подключения: ' . $conn->connect_error);
}
?>// 📁 header.php

<!DOCTYPE html><html>
<head>
  <meta charset="UTF-8">
  <title>Сайт</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>// 📁 footer.php

</body>
</html>// 📁 register.php

<?php
session_start();
require 'db.php';
include 'header.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $login = trim($_POST['login']);
    $email = trim($_POST['email']);
    $password = password_hash($_POST['password'], PASSWORD_DEFAULT);

    $stmt = $conn->prepare("SELECT id FROM users WHERE login = ? OR email = ?");
    $stmt->bind_param("ss", $login, $email);
    $stmt->execute();
    $stmt->store_result();

    if ($stmt->num_rows > 0) {
        echo "Пользователь уже существует";
    } else {
        $stmt = $conn->prepare("INSERT INTO users (login, email, password) VALUES (?, ?, ?)");
        $stmt->bind_param("sss", $login, $email, $password);
        $stmt->execute();
        echo "Регистрация успешна";
    }
}
?><form action="register.php" method="post">
  <h2>Регистрация</h2>
  <input type="text" name="login" placeholder="Логин" required>
  <input type="email" name="email" placeholder="Email" required>
  <input type="password" name="password" placeholder="Пароль" required>
  <button type="submit">Зарегистрироваться</button>
</form><?php include 'footer.php'; ?>// 📁 login.php

<?php
session_start();
require 'db.php';
include 'header.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $login = $_POST['login'];
    $password = $_POST['password'];

    $stmt = $conn->prepare("SELECT id, password FROM users WHERE login = ?");
    $stmt->bind_param("s", $login);
    $stmt->execute();
    $result = $stmt->get_result();

    if ($user = $result->fetch_assoc()) {
        if (password_verify($password, $user['password'])) {
            $_SESSION['user_id'] = $user['id'];
            echo "Вход выполнен";
        } else {
            echo "Неверный пароль";
        }
    } else {
        echo "Пользователь не найден";
    }
}
?><form action="login.php" method="post">
  <h2>Вход</h2>
  <input type="text" name="login" placeholder="Логин" required>
  <input type="password" name="password" placeholder="Пароль" required>
  <button type="submit">Войти</button>
</form><?php include 'footer.php'; ?>// 📁 application.php

<?php
session_start();
require 'db.php';
include 'header.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $fio = trim($_POST['fio']);
    $phone = trim($_POST['phone']);
    $address = trim($_POST['address']);
    $date = $_POST['date'];

    $stmt = $conn->prepare("INSERT INTO applications (fio, phone, address, date) VALUES (?, ?, ?, ?)");
    $stmt->bind_param("ssss", $fio, $phone, $address, $date);
    $stmt->execute();
    echo "Заявка отправлена";
}
?><form action="application.php" method="post">
  <h2>Подать заявку</h2>
  <input type="text" name="fio" placeholder="ФИО" required>
  <input type="text" name="phone" placeholder="Телефон" required>
  <input type="text" name="address" placeholder="Адрес" required>
  <input type="date" name="date" required>
  <button type="submit">Отправить</button>
</form><?php include 'footer.php'; ?>// 📁 style.css body { font-family: Arial, sans-serif; max-width: 500px; margin: 30px auto; padding: 20px; background: #f5f5f5; border-radius: 8px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }

form { display: flex; flex-direction: column; gap: 10px; }

input, button { padding: 10px; font-size: 16px; border: 1px solid #ccc; border-radius: 5px; }

button { background-color: #007BFF; color: white; cursor: pointer; transition: background-color 0.3s; }

button:hover { background-color: #0056b3; }

h2 { text-align: center; }

// 📄 SQL структура /* CREATE DATABASE exam_site; USE exam_site;

CREATE TABLE users ( id INT AUTO_INCREMENT PRIMARY KEY, login VARCHAR(50) NOT NULL, email VARCHAR(100) NOT NULL, password VARCHAR(255) NOT NULL );

CREATE TABLE applications ( id INT AUTO_INCREMENT PRIMARY KEY, fio VARCHAR(100) NOT NULL, phone VARCHAR(20) NOT NULL, address VARCHAR(255) NOT NULL, date DATE NOT NULL ); */
