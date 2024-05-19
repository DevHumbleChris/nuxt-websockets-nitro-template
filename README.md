# Nuxt 3 WebSocket Nitro Template

Welcome to the Nuxt 3 WebSocket Nitro Template! This repository provides a template for building real-time web applications using Nuxt 3, WebSockets, and the Nitro server engine.

## Table of Contents

- [Introduction](#introduction)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
  - [Development](#development)
- [WebSocket Setup](#websocket-setup)
  - [Enabling WebSocket Support](#enabling-websocket-support)
  - [Creating a WebSocket Handler](#creating-a-websocket-handler)
  - [Client-Side Connection](#client-side-connection)
    - [Install Vue Use in Nuxt](#install-vue-use-in-nuxt)
    - [WebSocket Client Component](#websocket-client-component)
- [Project Structure](#project-structure)
- [Configuration](#configuration)

## Introduction

This project demonstrates how to integrate WebSockets with a Nuxt 3 application using the Nitro server engine, enabling real-time communication between the client and server. WebSockets provide a persistent connection for real-time data exchange, which is essential for applications like chat apps, live notifications, and online gaming.

## Features

- **Nuxt 3**: Utilize the latest version of the Nuxt framework.
- **Nitro**: Leverage the Nitro server engine for universal deployments.
- **WebSockets**: Real-time data exchange using WebSockets.

## Prerequisites

Before you begin, ensure you have met the following requirements:

- Node.js (v14.x or later)
- npm or yarn or pnpm (i just love pnpm)

## Installation

1. Clone the repository:
    ```bash
    git clone https://github.com/DevHumbleChris/nuxt-websockets-nitro-template.git
    cd nuxt-websockets-nitro-template
    ```

2. Install dependencies:
    ```bash
    pnpm install
    ```

## Usage

### Development

To start the development server with hot module replacement:

```bash
pnpm run dev
```

## WebSocket Setup

### Enabling WebSocket Support

Nitro natively supports a cross-platform WebSocket API using CrossWS and H3 WebSocket. Note that WebSocket support is currently experimental. To enable WebSocket support, you need to set the experimental WebSocket feature flag in your nuxt.config.ts:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    experimental: {
      websocket: true
    }
  }
})
```

### Creating a WebSocket Handler

Create a WebSocket handler in `/server/api/api/websocket.ts` (or `server/routes/websocket.ts` for Nuxt). You can also use any route like `/server/api/chatroom.ts` to register an upgrade handler on `/chatroom`.

`websocket.ts`:

```typescript
export default defineWebSocketHandler({
  open(peer) {
    console.log("[ws] open", peer);
  },

  message(peer, message) {
    console.log("[ws] message", peer, message);
    if (message.text().includes("ping")) {
      peer.send("pong");
    }
  },

  close(peer, event) {
    console.log("[ws] close", peer, event);
  },

  error(peer, error) {
    console.log("[ws] error", peer, error);
  },
});

```

### Client-Side Connection

To connect to the server from the client side, you can use a WebSocket client component. First we will utilize the `VueUse`: Collection of Vue Composition Utilities. Mostly importantly we use the `useWebSocket` utility.

#### Install Vue Use in Nuxt

```bash
  pnpm add -D @vueuse/nuxt @vueuse/core
```

Add the `@vueuse/nuxt` to the `nuxt.config.ts` modules section.

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    '@vueuse/nuxt',
  ],
})
```

#### WebSocket Client Component

On the `components` folder create a client component named `WebSocket.client.vue`

```tsx
<script setup lang="ts">
// In prod: check if secure, then use wss://
const { status, data, send, open, close } = useWebSocket(
  `ws://${location.host}/api/websocket`
);

const history = ref<string[]>([]);
watch(data, (newValue: string) => {
  history.value.push(`server: ${newValue}`);
});

const message = ref("");

function sendData() {
  history.value.push(`client: ${message.value}`);
  send(message.value);
  message.value = "";
}
</script>

<template>
  <div>
    <h1>WebSocket - let's go!</h1>
    <form @submit.prevent="sendData">
      <input v-model="message" />
      <button type="submit">Send</button>
    </form>
    <div>
      <p v-for="entry in history">{{ entry }}</p>
    </div>
  </div>
</template>

```

This Vue component sets up a WebSocket connection using the `useWebSocket` utility from `vue-use`. It establishes a connection to the WebSocket server and sends messages entered by the user via an input field. Received messages are displayed in real-time, updating the history.

## Project Structure

```php
├── public          # Static files
├── src
│   ├── assets      # Asset files
│   ├── components  # Vue components
│   ├── layouts     # Layouts
│   ├── pages       # Pages
│   ├── plugins     # Plugins
│   ├── server      # Server-related code
│   │   ├── api     # API handlers
│   │   │   └── websocket.ts   # WebSocket server implementation
│   └── store       # Vuex store
├── nuxt.config.ts  # Nuxt configuration
├── package.json    # Project dependencies and scripts
└── README.md       # Project documentation
```

## Configuration

To configure the WebSocket server, edit the file `/server/api/websocket.ts`. Adjust the Nuxt configuration in `nuxt.config.ts` to suit your specific needs.

## Contributing

Contributions are welcome! Please follow these steps:

Fork the project.

1. Create your feature branch (git checkout -b feature/fooBar).
2. Commit your changes (git commit -m 'Add some fooBar').
3. Push to the branch (git push origin feature/fooBar).
4. Open a pull request.
