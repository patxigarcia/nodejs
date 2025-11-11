# Laboratorio: Server-Sent Events (SSE) con Express

## Objetivo
Implementar comunicación unidireccional servidor-cliente usando Server-Sent Events con Express y crear una interfaz HTML/JS para consumir los eventos.

## Requisitos Previos
- Node.js instalado
- Conocimientos básicos de Express
- Comprensión de eventos y comunicación asíncrona

## Parte 1: Configuración del Proyecto

Crea el directorio y el proyecto:

```bash
mkdir sse-lab
cd sse-lab
npm init -y
npm install express cors
```

## Parte 2: Servidor Express con SSE

Crea `server.js`:

```javascript
import express from 'express';
import cors from 'cors';
import path from 'path';
import { fileURLToPath } from 'url';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

const app = express();
const PORT = 3000;

// Activar CORS para permitir conexiones desde cualquier origen
app.use(cors());

// Servir archivos estáticos
app.use(express.static('public'));

// Ruta principal para servir el index.html
app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

// Endpoint SSE
app.get('/events', (req, res) => {
  // Configurar headers para SSE
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  // Enviar un comentario inicial para establecer la conexión
  res.write(': connected\n\n');

  // Enviar eventos cada 2 segundos
  const intervalId = setInterval(() => {
    const data = {
      timestamp: new Date().toISOString(),
      message: 'Mensaje desde el servidor',
      random: Math.floor(Math.random() * 100)
    };

    res.write(`data: ${JSON.stringify(data)}\n\n`);
  }, 2000);

  // Limpiar cuando el cliente cierra la conexión
  req.on('close', () => {
    clearInterval(intervalId);
    res.end();
  });
});

// Endpoint para enviar eventos personalizados
app.get('/notification', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  // Evento con nombre específico
  const sendNotification = (type, message) => {
    res.write(`event: ${type}\n`);
    res.write(`data: ${JSON.stringify({ message })}\n\n`);
  };

  // Enviar diferentes tipos de notificaciones
  setTimeout(() => sendNotification('info', 'Información importante'), 3000);
  setTimeout(() => sendNotification('warning', 'Advertencia del sistema'), 6000);
  setTimeout(() => sendNotification('success', 'Operación completada'), 9000);

  req.on('close', () => {
    res.end();
  });
});

app.listen(PORT, () => {
  console.log(`Servidor SSE ejecutándose en http://localhost:${PORT}`);
});
```

Actualiza `package.json` para usar módulos ES6:

```json
{
  "type": "module",
  "scripts": {
    "start": "node server.js"
  }
}
```

## Parte 3: Cliente HTML

Crea el directorio `public` y dentro `index.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SSE Demo</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 800px;
      margin: 50px auto;
      padding: 20px;
    }
    .container {
      border: 1px solid #ddd;
      padding: 20px;
      border-radius: 8px;
      margin-bottom: 20px;
    }
    .event {
      background: #f4f4f4;
      padding: 10px;
      margin: 10px 0;
      border-left: 4px solid #007bff;
      border-radius: 4px;
    }
    .notification {
      padding: 10px;
      margin: 10px 0;
      border-radius: 4px;
    }
    .info { background: #d1ecf1; border-left: 4px solid #0c5460; }
    .warning { background: #fff3cd; border-left: 4px solid #856404; }
    .success { background: #d4edda; border-left: 4px solid #155724; }
    button {
      background: #007bff;
      color: white;
      border: none;
      padding: 10px 20px;
      border-radius: 4px;
      cursor: pointer;
      margin: 5px;
    }
    button:hover { background: #0056b3; }
    .status {
      display: inline-block;
      width: 12px;
      height: 12px;
      border-radius: 50%;
      margin-right: 8px;
    }
    .connected { background: green; }
    .disconnected { background: red; }
  </style>
</head>
<body>
  <h1>Server-Sent Events Demo</h1>
  
  <div class="container">
    <h2>Eventos Continuos</h2>
    <p>
      <span class="status disconnected" id="status1"></span>
      <span id="statusText1">Desconectado</span>
    </p>
    <button id="connectBtn">Conectar</button>
    <button id="disconnectBtn">Desconectar</button>
    <div id="events"></div>
  </div>

  <div class="container">
    <h2>Notificaciones</h2>
    <p>
      <span class="status disconnected" id="status2"></span>
      <span id="statusText2">Desconectado</span>
    </p>
    <button id="notificationBtn">Iniciar Notificaciones</button>
    <div id="notifications"></div>
  </div>

  <script src="client.js"></script>
</body>
</html>
```

## Parte 4: Cliente JavaScript

Crea `public/client.js`:

```javascript
let eventSource = null;
let notificationSource = null;

// Elementos del DOM
const eventsDiv = document.getElementById('events');
const notificationsDiv = document.getElementById('notifications');
const status1 = document.getElementById('status1');
const statusText1 = document.getElementById('statusText1');
const status2 = document.getElementById('status2');
const statusText2 = document.getElementById('statusText2');

// Función para actualizar estado de conexión
function updateStatus(statusElement, textElement, connected) {
  if (connected) {
    statusElement.className = 'status connected';
    textElement.textContent = 'Conectado';
  } else {
    statusElement.className = 'status disconnected';
    textElement.textContent = 'Desconectado';
  }
}

// Conectar a eventos continuos
document.getElementById('connectBtn').addEventListener('click', () => {
  if (eventSource) {
    return;
  }

  eventSource = new EventSource('http://localhost:3000/events');

  eventSource.onopen = () => {
    updateStatus(status1, statusText1, true);
    addEvent('Conexión establecida', 'system');
  };

  eventSource.onmessage = (event) => {
    const data = JSON.parse(event.data);
    addEvent(`${data.message} - Random: ${data.random}`, data.timestamp);
  };

  eventSource.onerror = () => {
    updateStatus(status1, statusText1, false);
    addEvent('Error en la conexión', 'error');
    eventSource.close();
    eventSource = null;
  };
});

// Desconectar eventos continuos
document.getElementById('disconnectBtn').addEventListener('click', () => {
  if (eventSource) {
    eventSource.close();
    eventSource = null;
    updateStatus(status1, statusText1, false);
    addEvent('Conexión cerrada', 'system');
  }
});

// Conectar a notificaciones
document.getElementById('notificationBtn').addEventListener('click', () => {
  if (notificationSource) {
    return;
  }

  notificationsDiv.innerHTML = '';
  notificationSource = new EventSource('http://localhost:3000/notification');

  notificationSource.onopen = () => {
    updateStatus(status2, statusText2, true);
  };

  // Escuchar eventos específicos
  notificationSource.addEventListener('info', (event) => {
    const data = JSON.parse(event.data);
    addNotification(data.message, 'info');
  });

  notificationSource.addEventListener('warning', (event) => {
    const data = JSON.parse(event.data);
    addNotification(data.message, 'warning');
  });

  notificationSource.addEventListener('success', (event) => {
    const data = JSON.parse(event.data);
    addNotification(data.message, 'success');
    // Cerrar después del último evento
    setTimeout(() => {
      notificationSource.close();
      notificationSource = null;
      updateStatus(status2, statusText2, false);
    }, 1000);
  });

  notificationSource.onerror = () => {
    updateStatus(status2, statusText2, false);
    notificationSource.close();
    notificationSource = null;
  };
});

// Funciones auxiliares para mostrar eventos
function addEvent(message, timestamp) {
  const eventDiv = document.createElement('div');
  eventDiv.className = 'event';
  eventDiv.innerHTML = `
    <strong>${new Date().toLocaleTimeString()}</strong><br>
    ${message}
  `;
  eventsDiv.insertBefore(eventDiv, eventsDiv.firstChild);
  
  // Mantener solo los últimos 10 eventos
  while (eventsDiv.children.length > 10) {
    eventsDiv.removeChild(eventsDiv.lastChild);
  }
}

function addNotification(message, type) {
  const notifDiv = document.createElement('div');
  notifDiv.className = `notification ${type}`;
  notifDiv.innerHTML = `
    <strong>${type.toUpperCase()}</strong><br>
    ${message}
  `;
  notificationsDiv.appendChild(notifDiv);
}
```

## Parte 5: Ejecución

Inicia el servidor:

```bash
npm start
```

Abre el navegador en `http://localhost:3000` y prueba:

1. Conectar a eventos continuos
2. Observar mensajes cada 2 segundos
3. Desconectar cuando desees
4. Iniciar notificaciones y ver los 3 tipos de eventos

## Verificación

- El servidor envía eventos cada 2 segundos
- Los eventos se muestran en tiempo real
- Las notificaciones aparecen en secuencia
- Los estados de conexión se actualizan correctamente
- CORS permite conexiones desde cualquier origen

## Referencias

- [MDN - Server-sent events](https://developer.mozilla.org/es/docs/Web/API/Server-sent_events)
- [Express.js Documentation](https://expressjs.com/)
- [EventSource API](https://developer.mozilla.org/es/docs/Web/API/EventSource)
- [CORS en Express](https://expressjs.com/en/resources/middleware/cors.html)
