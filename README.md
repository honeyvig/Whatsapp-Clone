# Whatsapp-Clone
Creating a complete WhatsApp clone involves multiple components, including real-time messaging, user authentication, file sharing, media handling (images, videos, etc.), and notifications. Additionally, building such a full-featured clone is quite complex and requires a mix of technologies, such as:

    Backend: Flask, Django (Python), or Node.js (JavaScript)
    Frontend: HTML, CSS, JavaScript (React, Vue, or plain HTML/CSS)
    Real-time communication: WebSockets, Socket.IO, or Firebase
    Database: MySQL, PostgreSQL, MongoDB, etc.
    File storage: AWS S3, Firebase Storage, or local storage for media files

For simplicity, I’ll show you how to create a basic real-time messaging application using Flask for the backend and Socket.IO for real-time communication.

Basic Requirements:

    Install necessary libraries:

    pip install Flask flask-socketio

Step 1: Create the Backend (Flask + Flask-SocketIO)
app.py

from flask import Flask, render_template, request, session
from flask_socketio import SocketIO, send, emit
import os

# Create Flask app and configure SocketIO
app = Flask(__name__)
app.secret_key = os.urandom(24)
socketio = SocketIO(app)

# Store the users and messages temporarily
users = {}
messages = []

@app.route('/')
def home():
    return render_template('index.html')

# Route for logging in (set the user's name in the session)
@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    session['username'] = username
    users[username] = request.sid  # Store the user's socket ID
    return redirect('/chat')

@app.route('/chat')
def chat():
    if 'username' not in session:
        return redirect('/')
    return render_template('chat.html', username=session['username'], messages=messages)

# Socket event: handle receiving and broadcasting messages
@socketio.on('message')
def handle_message(msg):
    # Broadcast the message to all users
    messages.append(msg)
    emit('message', msg, broadcast=True)

# Socket event: handle user disconnecting
@socketio.on('disconnect')
def handle_disconnect():
    if 'username' in session:
        del users[session['username']]
        print(f"{session['username']} disconnected")

if __name__ == '__main__':
    socketio.run(app, debug=True)

Step 2: Frontend (HTML + JavaScript with Socket.IO)
templates/index.html

This file provides a basic login page where users can input their username.

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WhatsApp Clone - Login</title>
</head>
<body>
    <h1>Welcome to WhatsApp Clone</h1>
    <form action="/login" method="POST">
        <label for="username">Enter Username: </label>
        <input type="text" id="username" name="username" required>
        <button type="submit">Login</button>
    </form>
</body>
</html>

templates/chat.html

This file contains the chat interface. It allows users to send messages, and it will display messages in real-time.

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WhatsApp Clone - Chat</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/4.3.2/socket.io.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 10px;
        }
        .chat-box {
            width: 100%;
            max-width: 600px;
            margin: 0 auto;
        }
        .messages {
            border: 1px solid #ddd;
            height: 300px;
            overflow-y: scroll;
            padding: 10px;
        }
        .input-container {
            display: flex;
            margin-top: 10px;
        }
        .input-container input {
            flex: 1;
            padding: 10px;
            font-size: 16px;
        }
        .input-container button {
            padding: 10px;
        }
    </style>
</head>
<body>
    <h1>Welcome, {{ username }}!</h1>
    <div class="chat-box">
        <div class="messages" id="messages">
            {% for message in messages %}
                <p><strong>{{ message['user'] }}:</strong> {{ message['text'] }}</p>
            {% endfor %}
        </div>
        <div class="input-container">
            <input type="text" id="message-input" placeholder="Type a message..." autofocus>
            <button onclick="sendMessage()">Send</button>
        </div>
    </div>

    <script>
        const socket = io.connect('http://127.0.0.1:5000');
        const messagesDiv = document.getElementById('messages');
        const messageInput = document.getElementById('message-input');

        // Send a message to the server
        function sendMessage() {
            const messageText = messageInput.value;
            if (messageText.trim() !== "") {
                const message = {
                    user: "{{ username }}",
                    text: messageText
                };
                socket.send(message);
                messageInput.value = ''; // Clear input field
            }
        }

        // Receive message from server and display it
        socket.on('message', function(msg) {
            const messageElement = document.createElement('p');
            messageElement.innerHTML = `<strong>${msg.user}:</strong> ${msg.text}`;
            messagesDiv.appendChild(messageElement);
            messagesDiv.scrollTop = messagesDiv.scrollHeight; // Auto-scroll to the bottom
        });
    </script>
</body>
</html>

Explanation:

    Backend (app.py):
        We use Flask to serve the pages and handle user login.
        Flask-SocketIO handles real-time communication. When a user sends a message, it gets broadcasted to all connected users via WebSockets.
        users is a dictionary storing connected users and their socket IDs. messages stores the chat history.

    Frontend (index.html & chat.html):
        index.html is a simple login page where users can enter their username.
        chat.html shows the chat interface where users can send and receive messages. The messages are displayed in real-time using Socket.IO.
        The Socket.IO library is used to send and receive messages asynchronously.

    How it works:
        The user enters their username and is redirected to the chat page.
        When a user types a message and clicks the "Send" button, the message is sent to the server using Socket.IO.
        The server receives the message and broadcasts it to all connected users, who then see the message appear in real-time.

Step 3: Run the Application

    Make sure you have the necessary libraries installed (Flask and Flask-SocketIO).
    Run the Flask app:

    python app.py

    Open your browser and navigate to http://127.0.0.1:5000/.
    Log in with a username, and you’ll be directed to the chat page.
    Open multiple browser tabs or separate devices to simulate real-time communication.

Limitations and Next Steps:

    Authentication: You can integrate user authentication (login/logout) with databases like SQLite or MySQL, or use more advanced systems like JWT (JSON Web Tokens).
    Persistent Storage: The above code stores messages temporarily. For a real application, you would save messages and user data in a database.
    File Sharing: You can implement file uploads using Flask’s file handling or integrate a service like AWS S3.
    Media Support: For sending images and videos, you would need to integrate file uploads and display media files in the chat.
    Deployment: For production, you should deploy the app to a server (like Heroku or AWS) and use a more robust database.

Building a fully functional WhatsApp clone requires many more advanced features, including security, real-time notifications, data encryption, and scalability. This simple prototype provides a foundation to build upon.
