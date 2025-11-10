# Laboratorio: ExpressJS - Servidor Web y API REST

## Objetivo
Crear un servidor web con Express, implementar rutas, middleware, manejo de errores y desarrollar una API REST completa.

---

## Configuración inicial

### Inicializar proyecto
```bash
mkdir express-lab
cd express-lab
npm init -y
npm install express ejs
```

### Crear servidor básico
Crear `server.js`:
```javascript
const express = require('express');
const app = express();
const PORT = 3000;

app.use(express.json());

// Middleware personalizado de logging
const logger = (req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next();
};

app.use(logger);

// Configurar motor de plantillas
app.set('view engine', 'ejs');
app.set('views', './views');

// Datos simulados
let productos = [
  { id: 1, nombre: 'Laptop', precio: 999 },
  { id: 2, nombre: 'Mouse', precio: 25 },
  { id: 3, nombre: 'Teclado', precio: 75 }
];

// Ruta principal
app.get('/', (req, res) => {
  res.send('¡Servidor Express funcionando!');
});

// Ruta con rendering
app.get('/home', (req, res) => {
  res.render('index', { 
    title: 'Express App', 
    productos: productos 
  });
});

// API REST - CRUD

// READ - Obtener todos los productos
app.get('/api/productos', (req, res) => {  
  res.json(productos);  
});

// READ - Obtener un producto por ID
app.get('/api/productos/:id', (req, res) => {
  const producto = productos.find(p => p.id === parseInt(req.params.id));
  if (!producto) {
    return res.status(404).json({ error: 'Producto no encontrado' });
  }
  res.json(producto);
});

// CREATE - Crear un nuevo producto
app.post('/api/productos', (req, res) => {
  const { nombre, precio } = req.body;
  
  if (!nombre || !precio) {
    return res.status(400).json({ error: 'Nombre y precio son requeridos' });
  }

  const nuevoProducto = {
    id: productos.length > 0 ? Math.max(...productos.map(p => p.id)) + 1 : 1,
    nombre,
    precio: parseFloat(precio)
  };

  productos.push(nuevoProducto);
  res.status(201).json(nuevoProducto);
});

// UPDATE - Actualizar un producto
app.put('/api/productos/:id', (req, res) => {
  const producto = productos.find(p => p.id === parseInt(req.params.id));
  
  if (!producto) {
    return res.status(404).json({ error: 'Producto no encontrado' });
  }

  const { nombre, precio } = req.body;
  
  if (nombre) producto.nombre = nombre;
  if (precio) producto.precio = parseFloat(precio);

  res.json(producto);
});

// DELETE - Eliminar un producto
app.delete('/api/productos/:id', (req, res) => {
  const index = productos.findIndex(p => p.id === parseInt(req.params.id));
  
  if (index === -1) {
    return res.status(404).json({ error: 'Producto no encontrado' });
  }

  const productoEliminado = productos.splice(index, 1);
  res.json({ message: 'Producto eliminado', producto: productoEliminado[0] });
});

// Manejo de rutas no encontradas
app.use((req, res) => {
  res.status(404).json({ error: 'Ruta no encontrada' });
});

// Global Exception Handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(err.status || 500).json({
    error: {
      message: err.message || 'Error interno del servidor',
      status: err.status || 500
    }
  });
});

app.listen(PORT, () => {
  console.log(`Servidor corriendo en http://localhost:${PORT}`);
});
```

---

## Crear vista EJS

Crear carpeta `views` y archivo `views/index.ejs`:
```html
<!DOCTYPE html>
<html>
<head>
    <title><%= title %></title>
    <style>
        body { font-family: Arial; margin: 40px; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #4CAF50; color: white; }
    </style>
</head>
<body>
    <h1>Bienvenido a <%= title %></h1>
    <h2>Lista de Productos</h2>
    <table>
        <tr>
            <th>ID</th>
            <th>Nombre</th>
            <th>Precio</th>
        </tr>
        <% productos.forEach(producto => { %>
        <tr>
            <td><%= producto.id %></td>
            <td><%= producto.nombre %></td>
            <td>$<%= producto.precio %></td>
        </tr>
        <% }); %>
    </table>
</body>
</html>
```

---

## Ejecutar el servidor

```bash
node server.js
```

---

## Pruebas con cURL o Postman

### Obtener todos los productos
```bash
curl http://localhost:3000/api/productos
```

### Obtener un producto específico
```bash
curl http://localhost:3000/api/productos/1
```

### Crear un nuevo producto
```bash
curl -X POST http://localhost:3000/api/productos \
  -H "Content-Type: application/json" \
  -d '{"nombre":"Monitor","precio":299}'
```

### Actualizar un producto
```bash
curl -X PUT http://localhost:3000/api/productos/1 \
  -H "Content-Type: application/json" \
  -d '{"nombre":"Laptop Gaming","precio":1299}'
```

### Eliminar un producto
```bash
curl -X DELETE http://localhost:3000/api/productos/2
```

---

## Referencias

- [Express.js Official Documentation](https://expressjs.com/)
- [EJS Template Engine](https://ejs.co/)
- [Express Middleware Guide](https://expressjs.com/en/guide/using-middleware.html)
- [Express Error Handling](https://expressjs.com/en/guide/error-handling.html)
- [RESTful API Design Best Practices](https://restfulapi.net/)
