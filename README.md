<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Backend Interaction Example</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h1>Backend Interaction Example</h1>

    <h2>User Registration</h2>
    <form id="userRegisterForm">
        <input type="text" id="username" placeholder="Username" required><br>
        <input type="email" id="userEmail" placeholder="Email" required><br>
        <input type="password" id="userPassword" placeholder="Password" required><br>
        <button type="submit">Register User</button>
    </form>
    <div id="userRegisterResponse"></div>

    <h2>Store Registration</h2>
    <form id="storeRegisterForm">
        <input type="text" id="storeName" placeholder="Store Name" required><br>
        <input type="email" id="storeEmail" placeholder="Email" required><br>
        <input type="password" id="storePassword" placeholder="Password" required><br>
        <button type="submit">Register Store</button>
    </form>
    <div id="storeRegisterResponse"></div>

    <h2>Login User</h2>
    <form id="userLoginForm">
        <input type="text" id="loginUsername" placeholder="Username" required><br>
        <input type="password" id="loginPassword" placeholder="Password" required><br>
        <button type="submit">Login User</button>
    </form>
    <div id="userLoginResponse"></div>

    <h2>Login Store</h2>
    <form id="storeLoginForm">
        <input type="text" id="loginStoreName" placeholder="Store Name" required><br>
        <input type="password" id="loginStorePassword" placeholder="Password" required><br>
        <button type="submit">Login Store</button>
    </form>
    <div id="storeLoginResponse"></div>

    <script>
        document.getElementById('userRegisterForm').addEventListener('submit', function(event) {
            event.preventDefault();
            const username = document.getElementById('username').value;
            const email = document.getElementById('userEmail').value;
            const password = document.getElementById('userPassword').value;

            fetch('/api/register/user', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ username, email, password })
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('userRegisterResponse').textContent = data.message;
            })
            .catch(error => console.error('Error:', error));
        });

        document.getElementById('storeRegisterForm').addEventListener('submit', function(event) {
            event.preventDefault();
            const store_name = document.getElementById('storeName').value;
            const email = document.getElementById('storeEmail').value;
            const password = document.getElementById('storePassword').value;

            fetch('/api/register/store', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ store_name, email, password })
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('storeRegisterResponse').textContent = data.message;
            })
            .catch(error => console.error('Error:', error));
        });

        document.getElementById('userLoginForm').addEventListener('submit', function(event) {
            event.preventDefault();
            const username = document.getElementById('loginUsername').value;
            const password = document.getElementById('loginPassword').value;

            fetch('/api/login/user', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ username, password })
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('userLoginResponse').textContent = data.message;
            })
            .catch(error => console.error('Error:', error));
        });

        document.getElementById('storeLoginForm').addEventListener('submit', function(event) {
            event.preventDefault();
            const store_name = document.getElementById('loginStoreName').value;
            const password = document.getElementById('loginStorePassword').value;

            fetch('/api/login/store', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ store_name, password })
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('storeLoginResponse').textContent = data.message;
            })
            .catch(error => console.error('Error:', error));
        });
    </script>
</body>
</html>
}
}
<?php
include 'db.php';
include 'password_utils.php';

// Utility function for handling JSON responses
function jsonResponse($data, $status = 200) {
    header('Content-Type: application/json');
    http_response_code($status);
    echo json_encode($data);
    exit();
}

// Routing mechanism
$requestMethod = $_SERVER['REQUEST_METHOD'];
$requestUri = $_SERVER['REQUEST_URI'];

switch (true) {
    case ($requestUri == '/api/register/user' && $requestMethod == 'POST'):
        registerUser();
        break;
    case ($requestUri == '/api/register/store' && $requestMethod == 'POST'):
        registerStore();
        break;
    case ($requestUri == '/api/login/user' && $requestMethod == 'POST'):
        loginUser();
        break;
    case ($requestUri == '/api/login/store' && $requestMethod == 'POST'):
        loginStore();
        break;
    case ($requestUri == '/api/transaction' && $requestMethod == 'POST'):
        recordTransaction();
        break;
    case (preg_match('/\/api\/credits\/user\/(\d+)/', $requestUri, $matches) && $requestMethod == 'GET'):
        getUserCredits($matches[1]);
        break;
    case (preg_match('/\/api\/credits\/store\/(\d+)/', $requestUri, $matches) && $requestMethod == 'GET'):
        getStoreCredits($matches[1]);
        break;
    case (preg_match('/\/api\/notify\/user\/(\d+)/', $requestUri, $matches) && $requestMethod == 'GET'):
        notifyUser($matches[1]);
        break;
    case ($requestUri == '/api/redeem' && $requestMethod == 'POST'):
        redeemCredits();
        break;
    default:
        jsonResponse(['message' => 'Endpoint Not Found'], 404);
}

// Define functions for each endpoint
function registerUser() {
    global $conn;
    $data = json_decode(file_get_contents('php://input'), true);
    $username = $data['username'];
    $email = $data['email'];
    $password = password_hash($data['password'], PASSWORD_BCRYPT);

    $sql = "INSERT INTO users (username, email, password) VALUES ('$username', '$email', '$password')";
    if ($conn->query($sql) === TRUE) {
        jsonResponse(['message' => 'User registered successfully']);
    } else {
        jsonResponse(['message' => 'Error: ' . $conn->error], 500);
    }
}

function registerStore() {
    global $conn;
    $data = json_decode(file_get_contents('php://input'), true);
    $store_name = $data['store_name'];
    $email = $data['email'];
    $password = password_hash($data['password'], PASSWORD_BCRYPT);

    $sql = "INSERT INTO stores (store_name, email, password) VALUES ('$store_name', '$email', '$password')";
    if ($conn->query($sql) === TRUE) {
        jsonResponse(['message' => 'Store registered successfully']);
    } else {
        jsonResponse(['message' => 'Error: ' . $conn->error], 500);
    }
}

function loginUser() {
    global $conn;
    $data = json_decode(file_get_contents('php://input'), true);
    $username = $data['username'];
    $password = $data['password'];

    $sql = "SELECT * FROM users WHERE username='$username'";
    $result = $conn->query($sql);

    if ($result->num_rows > 0) {
        $user = $result->fetch_assoc();
        if (password_verify($password, $user['password'])) {
            jsonResponse(['message' => 'User logged in successfully']);
        } else {
            jsonResponse(['message' => 'Invalid credentials'], 401);
        }
    } else {
        jsonResponse(['message' => 'Invalid credentials'], 401);
    }
}

function loginStore() {
    global $conn;
    $data = json_decode(file_get_contents('php://input'), true);
    $store_name = $data['store_name'];
    $password = $data['password'];

    $sql = "SELECT * FROM stores WHERE store_name='$store_name'";
    $result = $conn->query($sql);

    if ($result->num_rows > 0) {
        $store = $result->fetch_assoc();
        if (password_verify($password, $store['password'])) {
            jsonResponse(['message' => 'Store logged in successfully']);
        } else {
            jsonResponse(['message' => 'Invalid credentials'], 401);
        }
    } else {
        jsonResponse(['message' => 'Invalid credentials'], 401);
    }
}

function recordTransaction() {
    global $conn;
    $data = json_decode(file_get_contents('php://input'), true);
    $user_id = $data['user_id'];
    $store_id = $data['store_id'];
    $amount_due = $data['amount_due'];
    $remaining_change = $data['remaining_change'];

    $sql = "INSERT INTO transactions (user_id, store_id, amount_due, remaining_change) VALUES ('$user_id', '$store_id', '$amount_due', '$remaining_change')";
    if ($conn->query($sql) === TRUE) {
        jsonResponse(['message' => 'Transaction recorded successfully']);
    } else {
        jsonResponse(['message' => 'Error: ' . $conn->error], 500);
    }
}

function getUserCredits($user_id) {
    global $conn;
    $sql = "SELECT remaining_change FROM transactions WHERE user_id='$user_id'";
    $result = $conn->query($sql);

    $total_credits = 0;
    while ($row = $result->fetch_assoc()) {
        $total_credits += $row['remaining_change'];
    }

    jsonResponse(['total_credits' => $total_credits]);
}

function getStoreCredits($store_id) {
    global $conn;
    $sql = "SELECT remaining_change FROM transactions WHERE store_id='$store_id'";
    $result = $conn->query($sql);

    $total_credits = 0;
    while ($row = $result->fetch_assoc()) {
        $total_credits += $row['remaining_change'];
    }

    jsonResponse(['total_credits' => $total_credits]);
}

function notifyUser($user_id) {
    global $conn;
    $sql = "SELECT username FROM users WHERE id='$user_id'";
    $result = $conn->query($sql);

    if ($result->num_rows > 0) {
        $user = $result->fetch_assoc();
        jsonResponse(['message' => "Notification for user {$user['username']}: You have credits available!"]);
    } else {
        jsonResponse(['message' => 'User not found'], 404);
    }
}

function redeemCredits() {
    global $conn;
    $data = json_decode(file_get_contents('php://input'), true);
    $user_id = $data['user_id'];
    $store_id = $data['store_id'];
    $amount_to_redeem = $data['amount_to_redeem'];

    $sql = "SELECT * FROM transactions WHERE user_id='$user_id' AND store_id='$store_id'";
    $result = $conn->query($sql);

    $total_credits = 0;
    $transactions = [];

    while ($row = $result->fetch_assoc()) {
        $total_credits += $row['remaining_change'];
        $transactions[] = $row;
    }

    if ($total_credits >= $amount_to_redeem) {
        foreach ($transactions as $transaction) {
            if ($transaction['remaining_change'] >= $amount_to_redeem) {
                $new_change = $transaction['remaining_change'] - $amount_to_redeem;
                $transaction_id = $transaction['id'];
                $sql = "UPDATE transactions SET remaining_change='$new_change' WHERE id='$transaction_id'";
                $conn->query($sql);
                jsonResponse(['message' => 'Credits redeemed successfully']);
                $conn->close();
                exit();
            } else {
                $amount_to_redeem -= $transaction['remaining_change'];
                $sql = "UPDATE transactions SET remaining_change=0 WHERE id='{$transaction['id']}'";
                $conn->query($sql);
            }
        }
        jsonResponse(['message' => 'Credits redeemed successfully']);
    } else {
        jsonResponse(['message' => 'Insufficient credits'], 400);
    }
}

$conn->close();
?>