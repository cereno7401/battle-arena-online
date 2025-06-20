battle-arena-online/
├── client/
│   ├── index.html
│   ├── style.css
│   └── game.js
├── server/
│   └── server.js
├── assets/
│   ├── player.png
│   ├── map.png
│   └── ...
├── package.json
├── README.md
└── .gitignore

const express = require('express');
const http = require('http');
const socket = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = socket(server);

let players = {};

io.on('connection', socket => {
  console.log(`New connection: ${socket.id}`);
  players[socket.id] = { x: 100, y: 100, hp: 100 };

  socket.emit('currentPlayers', players);
  socket.broadcast.emit('newPlayer', players[socket.id]);

  socket.on('move', data => {
    if (players[socket.id]) {
      players[socket.id].x += data.x;
      players[socket.id].y += data.y;
      io.emit('playerMoved', { id: socket.id, ...players[socket.id] });
    }
  });

  socket.on('disconnect', () => {
    delete players[socket.id];
    io.emit('playerDisconnected', socket.id);
  });
});

server.listen(3000, () => console.log('Server running on port 3000'));

const socket = io();
let players = {};

socket.on('currentPlayers', (serverPlayers) => {
  players = serverPlayers;
});

socket.on('newPlayer', (player) => {
  players[player.id] = player;
});

socket.on('playerMoved', (data) => {
  if (players[data.id]) {
    players[data.id].x = data.x;
    players[data.id].y = data.y;
  }
});

document.addEventListener('keydown', (e) => {
  const move = { x: 0, y: 0 };
  if (e.key === 'ArrowUp') move.y = -5;
  if (e.key === 'ArrowDown') move.y = 5;
  if (e.key === 'ArrowLeft') move.x = -5;
  if (e.key === 'ArrowRight') move.x = 5;
  socket.emit('move', move);
});
