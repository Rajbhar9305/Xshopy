<?php
session_start();
$conn = new mysqli("localhost", "root", "", "xshop");

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

// Handle login and signup
if (isset($_POST['login'])) {
    // Login logic
    $username = $_POST['username'];
    $password = $_POST['password'];
    $sql = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
    $result = $conn->query($sql);
    if ($result->num_rows > 0) {
        $_SESSION['user'] = $username;
    } else {
        echo "Invalid login credentials";
    }
}

if (isset($_POST['signup'])) {
    // Signup logic
    $username = $_POST['username'];
    $password = $_POST['password'];
    $sql = "INSERT INTO users (username, password) VALUES ('$username', '$password')";
    $conn->query($sql);
}

if (isset($_POST['book'])) {
    // Booking logic
    $name = $_POST['name'];
    $email = $_POST['email'];
    $service = $_POST['service'];
    $date = $_POST['date'];
    $sql = "INSERT INTO bookings (name, email, service, date) VALUES ('$name', '$email', '$service', '$date')";
    $conn->query($sql);
}

// Fetch products
$sql = "SELECT * FROM products";
$products_result = $conn->query($sql);

// Handle cart
if (isset($_POST['add_to_cart'])) {
    $_SESSION['cart'][] = $_POST['product_id'];
}

// Checkout logic
if (isset($_POST['checkout'])) {
    // Example checkout process
    echo "Checkout successful!";
}
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>XShop</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin: 20px; }
        .product { display: inline-block; width: 250px; margin: 20px; padding: 10px; border: 1px solid #ddd; }
        img { width: 100%; height: 150px; }
        button { background: #007bff; color: white; padding: 10px; border: none; cursor: pointer; }
        .form { margin: 20px; padding: 20px; background-color: #f4f4f4; border-radius: 8px; }
    </style>
</head>
<body>

    <h1>Welcome to XShop</h1>

    <!-- User Login and Signup -->
    <div class="form">
        <h2>Login</h2>
        <form method="POST">
            <input type="text" name="username" placeholder="Username" required><br>
            <input type="password" name="password" placeholder="Password" required><br>
            <button type="submit" name="login">Login</button>
        </form>
        <hr>
        <h2>Signup</h2>
        <form method="POST">
            <input type="text" name="username" placeholder="Username" required><br>
            <input type="password" name="password" placeholder="Password" required><br>
            <button type="submit" name="signup">Signup</button>
        </form>
    </div>

    <!-- Booking Form -->
    <div class="form">
        <h2>Book an Appointment</h2>
        <form method="POST">
            <input type="text" name="name" placeholder="Full Name" required><br>
            <input type="email" name="email" placeholder="Email" required><br>
            <select name="service">
                <option value="Web Development">Web Development</option>
                <option value="Digital Marketing">Digital Marketing</option>
                <option value="Business Consulting">Business Consulting</option>
            </select><br>
            <input type="date" name="date" required><br>
            <button type="submit" name="book">Book Now</button>
        </form>
    </div>

    <!-- Product Listing -->
    <h2>Our Products</h2>
    <?php while ($product = $products_result->fetch_assoc()): ?>
        <div class="product">
            <img src="<?php echo $product['image']; ?>" alt="Product Image">
            <h3><?php echo $product['name']; ?></h3>
            <p>$<?php echo $product['price']; ?></p>
            <form method="POST">
                <input type="hidden" name="product_id" value="<?php echo $product['id']; ?>">
                <button type="submit" name="add_to_cart">Add to Cart</button>
            </form>
        </div>
    <?php endwhile; ?>

    <!-- AI Chatbot -->
    <h2>Ask our AI Chatbot</h2>
    <div id="chatbox" style="height: 300px; overflow-y: scroll; border: 1px solid #ddd; margin-bottom: 20px; padding: 10px;">
        <!-- Chat content will be here -->
    </div>
    <textarea id="userInput" placeholder="Ask something..." style="width: 80%; height: 50px;"></textarea>
    <button onclick="sendMessage()">Send</button>

    <script>
        function sendMessage() {
            var userInput = document.getElementById("userInput").value;
            var chatbox = document.getElementById("chatbox");
            chatbox.innerHTML += "<p>User: " + userInput + "</p>";

            fetch('chatbot.php', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({message: userInput})
            })
            .then(response => response.json())
            .then(data => {
                chatbox.innerHTML += "<p>AI: " + data.response + "</p>";
                document.getElementById("userInput").value = ''; // Clear the input
                chatbox.scrollTop = chatbox.scrollHeight; // Scroll to the bottom
            });
        }
    </script>

    <!-- Checkout Form -->
    <h2>Checkout</h2>
    <form method="POST">
        <button type="submit" name="checkout">Proceed to Payment</button>
    </form>

</body>
</html>

<?php $conn->close(); ?>
