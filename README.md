[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=19993233&assignment_repo_type=AssignmentRepo)
# Real-Time Chat Application with Socket.io

This assignment focuses on building a real-time chat application using Socket.io, implementing bidirectional communication between clients and server.

## Assignment Overview

You will build a chat application with the following features:
1. Real-time messaging using Socket.io
2. User authentication and presence
3. Multiple chat rooms or private messaging
4. Real-time notifications
5. Advanced features like typing indicators and read receipts

## Project Structure

```
socketio-chat/
├── client/                 # React front-end
│   ├── public/             # Static files
│   ├── src/                # React source code
│   │   ├── components/     # UI components
│   │   ├── context/        # React context providers
│   │   ├── hooks/          # Custom React hooks
│   │   ├── pages/          # Page components
│   │   ├── socket/         # Socket.io client setup
│   │   └── App.jsx         # Main application component
│   └── package.json        # Client dependencies
├── server/                 # Node.js back-end
│   ├── config/             # Configuration files
│   ├── controllers/        # Socket event handlers
│   ├── models/             # Data models
│   ├── socket/             # Socket.io server setup
│   ├── utils/              # Utility functions
│   ├── server.js           # Main server file
│   └── package.json        # Server dependencies
└── README.md               # Project documentation
```

## Getting Started

1. Accept the GitHub Classroom assignment invitation
2. Clone your personal repository that was created by GitHub Classroom
3. Follow the setup instructions in the `Week5-Assignment.md` file
4. Complete the tasks outlined in the assignment

## Files Included

- `Week5-Assignment.md`: Detailed assignment instructions
- Starter code for both client and server:
  - Basic project structure
  - Socket.io configuration templates
  - Sample components for the chat interface

## Requirements

- Node.js (v18 or higher)
- npm or yarn
- Modern web browser
- Basic understanding of React and Express

## Submission

Your work will be automatically submitted when you push to your GitHub Classroom repository. Make sure to:

1. Complete both the client and server portions of the application                                                                                                     {
  "name": "socketio-chat-server",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "socket.io": "^4.7.0",
    "cors": "^2.8.5"
  }
}const express = require('express');
const http = require('http');
const cors = require('cors');
const { setupSocket } = require('./socket/socket');

const app = express();
app.use(cors());
const server = http.createServer(app);

setupSocket(server);

const PORT = process.env.PORT || 5000;
server.listen(PORT, () => {
  console.log(`Server listening on port ${PORT}`);
});
2. Implement the core chat functionality
import React, { useContext, useEffect, useState } from 'react';
import { UserContext } from '../context/UserContext';
import { socket } from '../socket/socket';

export default function ChatPage() {
  const { username } = useContext(UserContext);
  const [users, setUsers] = useState([]);
  const [messages, setMessages] = useState([]);
  const [message, setMessage] = useState('');
  const [toUserId, setToUserId] = useState(null);

  useEffect(() => {
    socket.on('user_joined', ({ username }) => {
      setMessages(prev => [...prev, { system: true, text: `${username} joined.` }]);
    });

    socket.on('user_left', ({ username }) => {
      setMessages(prev => [...prev, { system: true, text: `${username} left.` }]);
    });

    socket.on('active_users', userList => {
      setUsers(userList);
    });

    socket.on('receive_message', msg => {
      setMessages(prev => [...prev, msg]);
    });

    return () => socket.off();
  }, []);

  const send = () => {
    if (!message.trim()) return;
    socket.emit('send_message', { toUserId, message });
    setMessage('');
  };

  return (
    <div style={{ display: 'flex' }}>
      <div style={{ width: 200, padding: 10, borderRight: '1px solid #ccc' }}>
        <h3>Users</h3>
        <button onClick={() => setToUserId(null)}>Public</button>
        {users.map(user => (
          <div key={user.userId}>
            {user.username}
            <button onClick={() => setToUserId(user.userId)}>PM</button>
          </div>
        ))}
      </div>
      <div style={{ flex: 1, padding: 10 }}>
        <div style={{ height: 300, overflowY: 'scroll', border: '1px solid #ccc', padding: 5 }}>
          {messages.map((m, i) =>
            m.system ? (
              <div key={i} style={{ color: 'gray' }}>{m.text}</div>
            ) : (
              <div key={i}>
                <strong>{m.from}</strong>{m.toUserId ? ' (private)' : ''}: {m.message}
              </div>
            )
          )}
        </div>
        <input
          style={{ width: '80%' }}
          value={message}
          onChange={e => setMessage(e.target.value)}
          onKeyDown={e => e.key === 'Enter' && send()}
        />
        <button onClick={send}>Send</button>
      </div>
    </div>
  );
}
3. Add at least 3 advanced features
Add Typing Events
In server/socket/socket.js, add:

js
Copy
Edit
socket.on('typing', () => {
  socket.broadcast.emit('user_typing', { username: users[socket.id] });
});

socket.on('stop_typing', () => {
  socket.broadcast.emit('user_stop_typing', { username: users[socket.id] });
});
⚛️ Client Side: Update ChatPage.jsx
jsx
Copy
Edit
const [typingUsers, setTypingUsers] = useState([]);

useEffect(() => {
  socket.on('user_typing', ({ username }) => {
    setTypingUsers(prev => [...new Set([...prev, username])]);
  });
  socket.on('user_stop_typing', ({ username }) => {
    setTypingUsers(prev => prev.filter(name => name !== username));
  });
}, []);
Then, inside the render section:

jsx
Copy
Edit
{typingUsers.length > 0 && (
  <div style={{ color: 'gray' }}>
    {typingUsers.join(', ')} typing...
  </div>
)}
In the <input>:

jsx
Copy
Edit
onChange={e => {
  setMessage(e.target.value);
  socket.emit(e.target.value ? 'typing' : 'stop_typing');
}}
💾 2. Chat History with MongoDB
🔧 Add Mongoose to server
bash
Copy
Edit
cd server
npm install mongoose
server/models/Message.js
js
Copy
Edit
const mongoose = require('mongoose');

const messageSchema = new mongoose.Schema({
  from: String,
  toUserId: String,
  message: String,
  timestamp: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Message', messageSchema);

4. Document your setup process and features in the README.md                                                                                                          # 🔌 Socket.IO Chat Application

A real-time chat application built with **React**, **Node.js**, **Express**, and **Socket.IO**, featuring private messaging, typing indicators, chat history,
5. Include screenshots or GIFs of your working application
socketio-chat/
├── assets/
│   ├── chat-screenshot.png
│   └── typing-indicator.gif
├── client/
├── server/
└── README.md
## 📷 Screenshots

### 🧑‍🤝‍🧑 Group Chat in Action

![Chat UI](./assets/chat-screenshot.png)

---

### ⌨️ Typing Indicator (Live)

![Typing Indicator](./assets/typing-indicator.gif)
6. Optional: Deploy your application and add the URLs to your README.md

## Resources

- [Socket.io Documentation](https://socket.io/docs/v4/)
- [React Documentation](https://react.dev/)
- [Express.js Documentation](https://expressjs.com/)
- [Building a Chat Application with Socket.io](https://socket.io/get-started/chat) 
