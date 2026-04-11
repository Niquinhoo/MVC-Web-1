## MVC en Express

Una buena formar de organizar nuestro proyecto Express es la que se presenta a continuación estructura:

- **Estructura de un proyecto Express con MVC**
    
    📄 app.js
    
    ⚙️ package.json
    
    ⚙️ package-lock.json
    
    **📂 controllers**
    
    tareasController.js
    
    **📂 models**
    
    tareasModel.js
    
    📂 **views**
    
    📂 **pages**
    
    📄 index.ejs
    
    📄 about.ejs
    
    📂 **partials**
    
    📄 header.ejs
    
    📄 footer.ejs
    
    📄 taskItem.ejs
    
    📂 **public**
    
    📂 **css**
    📂 **js**
    

Como podemos ver, esta estructura complementa lo que ya habíamos empezado a crear con la carpera **views**. Ahora, cada carpeta cumple un rol un poco más claro dentro del patrón MVC:

- 📂 **controllers** contiene la lógica que responde a las rutas,
- 📂 **models** contiene la lógica de datos,
- y 📂 **views** contiene las plantillas EJS.

Por último, hemos agregado una carpeta adicional llamada 📂 **public** que contiene archivos estáticos que usaremos de forma global.

<aside>
💡

Esta estructura no es obligatoria, pero es una convención muy extendida que facilita el mantenimiento y la lectura del código.

</aside>

Ahora, el flujo de nuestra aplicación es un poco más largo pero sencillo, escalable y ordenado. Podemos resumir el flujo de una solicitud en MVC de la siguiente manera:

1. El navegador hace una solicitud a una ruta.
2. Express ejecuta el controlador asociado.
3. El controlador pide datos al modelo.
4. El modelo devuelve los datos.
5. El controlador envía esos datos a la vista.
6. La vista genera el HTML final.
7. El servidor envía el HTML al navegador.


# Aplicando MVC en Express en un ejemplo

Ahora que ya comprendemos qué es una aplicación monolítica, cómo funciona el SSR y cómo organizar nuestro proyecto con el patrón MVC, podemos construir un ejemplo completo que combine todos estos conceptos en una pequeña aplicación Express.

La idea es implementar una **lista de tareas** muy simple, pero organizada correctamente en **Modelos**, **Vistas** y **Controladores**, usando **EJS** para generar el HTML en el servidor.

## Estructura del Proyecto

La estructura recomendada para este ejemplo es la siguiente:

```html
📦 express-mvc-1
├── ⚙️ app.js
├── 📋 package.json
├── 📋 package-lock.json
├── 🎮 **controllers/**
│   └── 📄 **tasks.controller.js**
├── 🗃️ **models/**
│   └── 📄 **tasks.model.js**
├── 🌐 public/
├── 🖼️ **views/**
│   └── 📁 tasks/
│       ├── 📄 index.ejs
│       └── 📄 details.ejs
└── 📦 node_modules/
```

## El Modelo: `tasks.model.js`

El modelo representa los datos y la lógica de negocio. En este ejemplo, usaremos un arreglo en memoria para simplificar.

```jsx
// models/tasks.model.js

let tasks = [
  { id: 1, title: "Aprender JavaScript", difficulty: 2, description: "Aprender los conceptos básicos de JavaScript" },
  { id: 2, title: "Aprender HTML", difficulty: 1, description: "Aprender a crear estructuras de página web con HTML" },
  { id: 3, title: "Aprender CSS", difficulty: 1, description: "Aprender a dar estilo a las páginas web con CSS" }
];

function getAll () {
  return tasks;
}

function getById (id) {
  return tasks.find(t => t.id === id);
}

module.exports = {
  getAll,
  getById
};
```

## El Controlador: `tasks.controller.js`

El controlador recibe la solicitud, pide datos al modelo y decide qué vista mostrar.

```jsx
// controllers/tasks.controller.js

const tasksModel = require("../models/tasks.model");

function list (req, res) {
  const tasks = tasksModel.getAll();
  res.render("tasks/index", { tasks });
}

function detail (req, res) {
  const id = Number(req.params.id);
  const task = tasksModel.getById(id);

  if (!task) {
    return res.status(404).send("Tarea no encontrada");
  }

  res.render("tasks/details", { task });
}

module.exports = {
  list,
  detail
};
```

## Las Vistas

### `views/tasks/index.ejs`

```
<h1>Lista de Tareas</h1>

<ul>
  <% tasks.forEach(t=> { %>
    <li>
      <a href="/tasks/<%= t.id %>">
        <%= t.title %> (Dificultad: <%= t.difficulty %>)
      </a>
    </li>
    <% }) %>
</ul>
```

### `views/tasks/details.ejs`

```
<h1>
  <%= task.title %>
</h1>

<p>Dificultad: <%= task.difficulty %>
</p>

<p><%= task.description %></p>

<a href="/tasks">Volver a la lista</a>
```

## Configuración de Express: `app.js`

```jsx
// app.js

const express = require("express");
const path = require("path");
const tareasController = require("./controllers/tareas.controller");

const app = express();

// Configuración de EJS
app.set("view engine", "ejs");
app.set("views", path.join(__dirname, "views"));

// Archivos estáticos
app.use(express.static(path.join(__dirname, "public")));

// Rutas
app.get("/tareas", tareasController.listar);
app.get("/tareas/:id", tareasController.detalle);

// Inicio del servidor
app.listen(3000, () => {
  console.log("Servidor funcionando en http://localhost:3000");
});
```

## Flujo Completo de la Aplicación

### **1. Punto de entrada `app.js`**

- Crea el servidor Express.
- Configura **EJS** como motor de vistas (buscará archivos en views).
- Sirve archivos estáticos desde `public`.
- Registra dos rutas y las delega al controlador:
    - `GET` `/tasks` → `tasksController.list`
    - `GET` `/tasks/:id` → `tasksController.detail`
- Escucha en el puerto **3000**.

### **2. Controlador `tasks.controller.js`**

Recibe las peticiones HTTP y coordina modelo + vista.

- **list** (`GET` `/tasks`):
    1. Llama a `tasksModel.getAll()` para obtener todas las tareas.
    2. Renderiza la vista `tasks/index.ejs` pasándole el array `tasks`.
- **detail** (`GET` `/tasks/:id`):
    1. Extrae el `id` de la URL y lo convierte a número.
    2. Llama a `tasksModel.getById(id)` para buscar la tarea.
    3. Si no existe → responde con **404**.
    4. Si existe → renderiza `tasks/details.ejs` pasándole el objeto `task`.

### **3. Modelo `tasks.model.js`**

Es la fuente de datos (en memoria). Expone dos funciones:

- `getAll()` → devuelve el array completo de tareas.
- `getById(id)` → busca y devuelve una tarea por su `id`.

### **4. Vistas `tasks`**

Plantillas EJS que generan el HTML final:

- `index.ejs` → recorre el array `tasks` y muestra cada tarea como un enlace con su `title` y `difficulty`.
- `details.ejs` → muestra el `title` y `difficulty` de una sola tarea, con un enlace para volver.

# Agregando Estilos

En una aplicación Express organizada con el patrón **MVC**, donde las vistas se generan con **EJS** y se reutilizan fragmentos mediante **partials**, la incorporación de estilos requiere comprender cómo se integran tres piezas fundamentales: los *archivos estáticos*, la *estructura de vistas* y la *composición de plantillas*. El objetivo es mantener una arquitectura ordenada, coherente y fácil de mantener a medida que la aplicación crece.

## Archivos estáticos y la carpeta `public`

Express permite servir archivos estáticos —como hojas de estilo, imágenes o scripts del lado del cliente— a través de una carpeta dedicada. Esta carpeta suele llamarse `public` y se ubica en la raíz del proyecto. Dentro de ella se organizan los recursos según convenga, por ejemplo:

📂 public

📂 css

🎨 styles.css

📂 js

La aplicación debe estar configurada para exponer esta carpeta al navegador. Una vez hecho esto, cualquier archivo dentro de `public` puede ser referenciado directamente desde las vistas EJS.

<aside>
💡

**¿Cómo exponer la carpeta `public` en Express?**

Express incluye un mecanismo integrado para servir archivos estáticos. La configuración se realiza en el archivo principal de la aplicación (generalmente `app.js`) y consiste en una sola instrucción:

```json
app.use(express.static("public"))
```

</aside>

## Incorporación de Estilos en el `head`

El *partial* que contiene la sección `<head>` es el lugar natural para vincular las hojas de estilo. Allí se incluyen las etiquetas `<link>` que apuntan a los archivos dentro de la carpeta `public`. Como los archivos estáticos están expuestos desde la raíz, la referencia suele ser directa:

```html
<link rel="stylesheet" href="/css/estilos.css">
```

Este *partial* se incluye luego en cada vista o en un layout general, garantizando que todas las páginas compartan la misma base visual.

## Uso de Layouts para Estructura Global

En aplicaciones con múltiples vistas, es habitual definir un **layout principal** que actúa como plantilla base. Este layout incluye los *partials* esenciales:

- el `<head>` con los estilos,
- el encabezado o menú,
- el pie de página,
- y un espacio reservado para el contenido específico de cada vista.

Las vistas individuales se limitan a definir el contenido propio, mientras que la estructura general se mantiene en un único archivo. Esto asegura consistencia visual y facilita la incorporación de nuevos estilos o cambios globales.

## Estilos Específicos por Vista

Aunque la mayor parte del estilo suele ser global, algunas vistas pueden requerir reglas particulares. En esos casos, existen dos enfoques:

- agregar clases específicas en el HTML y definir sus estilos en la hoja principal;
- o incluir una hoja de estilo adicional desde el partial del `<head>` cuando la vista lo requiera.

El primer enfoque es más simple y mantiene el proyecto ordenado; el segundo puede ser útil cuando se trabaja con módulos visuales bien diferenciados.

## Integración con el Patrón MVC

La incorporación de estilos no modifica la estructura MVC, pero sí se beneficia de ella:

- **Modelos**: no intervienen en la presentación.
- **Controladores**: solo envían datos a las vistas.
- **Vistas**: reciben los estilos a través de partials y layouts.

Esta separación asegura que la lógica de negocio permanezca aislada de la presentación, permitiendo que los estilos evolucionen sin afectar el funcionamiento interno de la aplicación.

## Consideraciones de Mantenibilidad

Al trabajar con estilos en un proyecto Express con MVC y partials, conviene tener en cuenta:

- mantener una estructura clara dentro de `public/css`;
- evitar estilos embebidos en las vistas;
- reutilizar partials para evitar duplicación;
- nombrar las clases de forma coherente con la estructura del proyecto;
- documentar brevemente la función de cada hoja de estilo cuando el proyecto crezca.

Una organización consistente permite que el equipo pueda modificar la apariencia de la aplicación sin riesgo de romper la lógica ni afectar la estructura general.