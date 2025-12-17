[package.json](https://github.com/user-attachments/files/24213382/package.json)[Uploadi{
  "name": "bot-de-bilhetes",
  "version": "1.0.0",
  "description": "Bot de tickets Discord",
  "main": "src/index.js",
  "type": "commonjs",
  "scripts": {
    "start": "tsx src/index.ts",
    "dev": "tsx watch src/index.ts"
  },
  "dependencies": {
    "discord.js": "^14.14.1",
    "dotenv": "^16.4.5",
    "tsx": "^4.7.1",
    "typescript": "^5.3.3"
  }
}ng package.json…]()

node_modules
.env

[16/12/2025, 19:59:36] Ticket criado por rodrigobarros0267
[16/12/2025, 19:59:47] Ticket fechado: ticket-rodrigobarros0267
[16/12/2025, 20:45:06] Ticket criado por camera133297
[16/12/2025, 20:45:12] Ticket fechado: ticket-camera133297
[16/12/2025, 21:47:19] Ticket criado por camera133297
[16/12/2025, 21:47:50] Ticket fechado: ticket-camera133297

require('dotenv').config();
const { Client, GatewayIntentBits } = require('discord.js');

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMessages,
    GatewayIntentBits.MessageContent
  ]
});

require('./handlers/events')(client);
require('./handlers/tickets')(client);

client.login(process.env.TOKEN);
