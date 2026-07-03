# git-clone-https-github.com-hirentimbadiya-Video-Peers.git-cd-Video-Peers
/*
Multi-User Video Chat App – Full HTTPS with Custom Domain
Frontend: Vercel, Backend: Heroku/Render
*/

/* ========================= BACKEND ========================= */
// backend/server.js
const express = require('express');
const http = require('http');
const https = require('https');
const fs = require('fs');
const { Server } = require('socket.io');
const cors = require('cors');
const selfsigned = require('selfsigned');

const app = express();
app.use(cors());

let server;
if(process.env.NODE_ENV === 'production') {
  // Use real SSL certificates for custom domain
  const privateKey = fs.readFileSync('ssl/key.pem', 'utf8');
  const certificate = fs.readFileSync('ssl/cert.pem', 'utf8');
  const credentials = { key: privateKey, cert: certificate };
  server = https.createServer(credentials, app);
} else {
  // Generate self-signed certificate automatically for local dev
  const attrs = [{ name: 'commonName', value: 'localhost' }];
  const pems = selfsigned.generate(attrs, { days: 365 });
  server = https.createServer({ key: pems.private, cert: pems.cert }, app);
}

const io = new Server(server, { cors: { origin: process.env.FRONTEND_URL || '*' } });

io.on('connection', socket => {
  console.log('User connected:', socket.id);
  socket.on('join-room', roomId => { socket.join(roomId); socket.to(roomId).emit('user-connected', socket.id); });
  socket.on('offer', payload => io.to(payload.target).emit('offer', payload));
  socket.on('answer', payload => io.to(payload.target).emit('answer', payload));
  socket.on('ice-candidate', payload => io.to(payload.target).emit('ice-candidate', payload.candidate));
  socket.on('chat-message', payload => io.to(payload.roomId).emit('chat-message', payload));
  socket.on('disconnect', () => console.log('User disconnected:', socket.id));
});

const PORT = process.env.PORT || 5000;
server.listen(PORT, () => console.log(`Server running on port ${PORT}`));

/* ========================= FRONTEND ========================= */
// frontend/.env
REACT_APP_BACKEND_URL=https://your-backend-domain.com

/* ========================= README ========================= */
# Multi-User Video Chat App – Custom Domain HTTPS

## Features
- Multi-user video/audio chat via WebRTC over HTTPS
- Real-time text chat via Socket.io
- Dynamic rooms with custom Room IDs
- Automatic self-signed SSL for local dev, real SSL for production
- Frontend: Vercel, Backend: Heroku/Render or custom server

## Custom Domain Setup
### Backend
1. Obtain SSL certificates for your custom domain (e.g., via Let's Encrypt)
2. Place `key.pem` and `cert.pem` in `backend/ssl`
3. Set environment variable `FRONTEND_URL` to your frontend domain
4. Deploy backend to Heroku/Render/custom server

### Frontend
1. Set `REACT_APP_BACKEND_URL=https://your-backend-domain.com` in `.env`
2. Deploy frontend to Vercel with your custom domain

### Local Development
- Backend automatically generates self-signed SSL for HTTPS
- Frontend points to `https://localhost:5000` by default
- Open multiple browser tabs and join the same Room ID to test
