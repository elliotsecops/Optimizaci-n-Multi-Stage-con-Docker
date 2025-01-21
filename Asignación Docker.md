### **Docker:**

**Tarea: Optimizar una Imagen Docker Multi-Stage**
En esta tarea, optimizarás una imagen Docker existente utilizando construcciones multi-stage. El objetivo es reducir el tamaño de la imagen final, lo que resulta en descargas más rápidas, menor uso de almacenamiento e **mejorada seguridad**.

**Pasos:**

1. **Seleccionar una Imagen Base**: Elige una imagen base popular (por ejemplo, Node.js, Python, Java) que tenga un tamaño considerable.
2. **Crear un Dockerfile**: Crea un Dockerfile que construya una aplicación simple (por ejemplo, un "Hola Mundo") utilizando la imagen base seleccionada.
3. **Optimizar con Construcciones Multi-Stage**: Modifica el Dockerfile para usar construcciones multi-stage. Copia solo los archivos necesarios para la ejecución de la aplicación a una imagen más ligera (por ejemplo, alpine).
4. **Comparar Tamaños**: Compara el tamaño de la imagen original con el tamaño de la imagen optimizada.
5. **Documentar el Proceso**: Crea un archivo README.md en un repositorio de GitHub explicando el proceso de optimización, incluyendo los Dockerfiles originales y optimizados, y los resultados de la comparación de tamaños.

Vamos a abordar esto paso a paso. Comenzaremos con el **Paso 1: Seleccionar una Imagen Base**.

---
### **Paso 1: Seleccionar una Imagen Base**
Para esta tarea, usaremos **Node.js** como la imagen base, ya que es ampliamente utilizada y tiene un tamaño de imagen predeterminado relativamente grande. La imagen oficial de Node.js (`node`) es una buena candidata para la optimización.

#### **¿Por qué Node.js?**
- Es un entorno de ejecución popular para construir aplicaciones web.
- La imagen predeterminada `node` se basa en una distribución completa de Linux (por ejemplo, Debian), lo que la hace grande.
- Es un gran ejemplo para demostrar cómo las construcciones multi-stage pueden reducir el tamaño de la imagen.

---

### **Paso 1.1: Elegir la Versión de Node.js**
Usaremos la última versión LTS (Long-Term Support) de Node.js, que es **Node.js 18** en el momento de escribir este artículo. Puedes verificar las etiquetas disponibles en la [página de Docker Hub de Node.js](https://hub.docker.com/_/node).

La imagen predeterminada `node:18` se basa en Debian y es bastante grande. Aquí tienes cómo puedes verificar su tamaño:

1. Descarga la imagen:
   ```bash
   docker pull node:18
   ```

2. Verifica el tamaño de la imagen:
   ```bash
   docker images node:18
   ```

   Verás algo como esto:
   ```
   REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
   node         18        abcdef123456   2 weeks ago   1.02GB
   ```

   Como puedes ver, la imagen predeterminada `node:18` tiene más de **1GB** de tamaño. Nuestro objetivo es reducir esto significativamente utilizando construcciones multi-stage. **Las imágenes más pequeñas son más eficientes, se despliegan más rápido y presentan una superficie de ataque más pequeña.**

---

### **Paso 1.2: Planificar la Optimización**
Para optimizar la imagen, usaremos una **construcción multi-stage**. Esto implica dos etapas:

1. **Etapa 1 (Construcción)**: Usamos la imagen completa `node:18` para construir nuestra aplicación. Esta etapa tendrá todas las herramientas de desarrollo necesarias, como compiladores y dependencias de construcción. La aplicación se empaqueta entonces típicamente en un formato listo para producción usando un comando como `npm run build`. Este comando compila y optimiza el código de la aplicación y las dependencias para producción.

2. **Etapa 2 (Producción)**: Usamos una imagen base mucho más pequeña, como `node:18-alpine`. **Alpine Linux** es una distribución mínima, ideal para crear contenedores pequeños y seguros. Copiaremos solo los archivos *necesarios* (por ejemplo, la aplicación compilada y las dependencias de tiempo de ejecución) de la etapa de construcción a esta imagen más pequeña.

Aquí un diagrama simple para visualizar el proceso:

```
Stage 1 (Build)                    Stage 2 (Production)
+-----------------------+          +-------------------+
| node:18               |          | node:18-alpine    |
| - Build tools         |          | - Minimal OS      |
| - Source code         |  ----->  | - Application     |
| - Dependencies        |  Copy    | - Runtime deps    |
+-----------------------+          +-------------------+
      (Large Image)                     (Small Image) 
```

---

Ahora crearemos una simple aplicación "Hola Mundo" de Node.js y un Dockerfile para construirla (**Paso 2**). 

¡Gracias por tus excelentes sugerencias! Son muy útiles para hacer la guía aún más clara y accesible, especialmente para principiantes. Vamos a incorporar tus sugerencias en la explicación. Aquí está la versión actualizada del **Paso 2** con las mejoras que mencionaste:

---

### **Paso 2: Crear un Dockerfile**

#### **2.1 Crear una aplicación simple de Node.js**
Primero, necesitamos una aplicación básica de Node.js para trabajar. Vamos a crear un proyecto simple con un archivo `index.js` que muestre "Hello World".

1. Crea una carpeta para el proyecto:
   ```bash
   mkdir node-hello-world
   cd node-hello-world
   ```

2. Inicializa un proyecto de Node.js (la bandera `-y` acepta todos los valores predeterminados):
   ```bash
   npm init -y
   ```

3. Crea un archivo `index.js` con el siguiente contenido:
   ```javascript
   // index.js
   const http = require('http');

   const server = http.createServer((req, res) => {
     res.writeHead(200, { 'Content-Type': 'text/plain' });
     res.end('Hello World\n');
   });

   const PORT = 3000;
   server.listen(PORT, () => {
     console.log(`Server running at http://localhost:${PORT}/`);
   });
   ```

4. Prueba la aplicación localmente:
   ```bash
   node index.js
   ```

   Visita `http://localhost:3000` en tu navegador. Deberías ver "Hello World".

---

#### **2.2 Crear el Dockerfile**
Ahora, crearemos un Dockerfile para construir una imagen de Docker para esta aplicación.

1. Crea un archivo llamado `Dockerfile` en la raíz del proyecto:
   ```bash
   touch Dockerfile
   ```

2. El archivo `Dockerfile` debería tener el siguiente contenido:
   ```Dockerfile
   # Usar la imagen base de Node.js 18
   FROM node:18

   # Establecer el directorio de trabajo dentro del contenedor
   WORKDIR /app

   # Copiar el archivo package.json y package-lock.json
   # (Esto copia tanto `package.json`, que lista las dependencias del proyecto,
   # como `package-lock.json`, que bloquea las versiones específicas de esas dependencias
   # para garantizar compilaciones consistentes).
   COPY package*.json ./

   # Instalar las dependencias
   RUN npm install

   # Copiar el resto de los archivos de la aplicación
   # (Esto copia todos los demás archivos y carpetas del directorio actual en tu máquina
   # al directorio `/app` dentro del contenedor).
   COPY . .

   # Exponer el puerto en el que la aplicación escucha
   EXPOSE 3000

   # Comando para ejecutar la aplicación
   CMD ["node", "index.js"]
   ```

---

#### **2.3 Explicación del Dockerfile**
- **`FROM node:18`:** Usa la imagen base de Node.js 18.
- **`WORKDIR /app`:** Establece el directorio de trabajo dentro del contenedor.
- **`COPY package*.json ./`:** Copia los archivos `package.json` y `package-lock.json` al directorio de trabajo.
- **`RUN npm install`:** Instala las dependencias del proyecto.
- **`COPY . .`:** Copia el resto de los archivos de la aplicación al directorio de trabajo.
- **`EXPOSE 3000`:** Expone el puerto 3000, que es donde la aplicación escucha.
- **`CMD ["node", "index.js"]`:** Define el comando para ejecutar la aplicación.

---

#### **2.4 Construir la imagen de Docker**
Ahora, construye la imagen de Docker usando el Dockerfile:

1. Asegúrate de estar en el directorio del proyecto `node-hello-world`, luego, construye la imagen de Docker:
   ```bash
   docker build -t node-hello-world:original .
   ```

2. Verifica que la imagen se haya creado:
   ```bash
   docker images
   ```
   
   Deberías ver algo como esto:
   ```
   REPOSITORY          TAG        IMAGE ID       CREATED          SIZE
   node-hello-world    original   abcdef123456   10 seconds ago   1.02GB
   ```

---

#### **2.5 Ejecutar el contenedor**
Finalmente, ejecuta un contenedor basado en la imagen que acabas de crear:

1. Ejecuta el contenedor, mapeando el puerto 3000 en tu máquina local al puerto 3000 dentro del contenedor:
   ```bash
   docker run -p 3000:3000 node-hello-world:original
   ```

2. Visita `http://localhost:3000` en tu navegador. Deberías ver "Hello World".

---
### **Diagrama del Flujo**
Aquí tienes un diagrama simple que muestra el flujo del proceso:

```
Aplicación Node.js -> Dockerfile -> Imagen Docker -> Contenedor Docker
```

---

¡Gracias por el feedback y la sugerencia! Tienes toda la razón: es importante explicar cómo encontrar el `<container_id>` para que los principiantes no se sientan perdidos. Vamos a incorporar esa aclaración en el **Paso 6 (Limpieza)** para que la explicación sea aún más completa y fácil de seguir.

Aquí está la versión actualizada con tu sugerencia:

---

### **Paso 3: Optimizar con Multi-Stage Builds**

Ahora vamos a optimizar el Dockerfile utilizando **construcciones multi-etapa**. El objetivo es reducir el tamaño de la imagen final y mejorar la seguridad eliminando herramientas y archivos innecesarios.
#### **3.1 Modificar el Dockerfile**
Actualizaremos el Dockerfile para usar dos etapas:
1. **Etapa de construcción (build):** Usaremos la imagen completa de `node:18` para instalar dependencias y compilar la aplicación.
2. **Etapa de producción:** Usaremos una imagen más ligera (`node:18-alpine`) y copiaremos solo los archivos necesarios desde la etapa de construcción.

Aquí está el Dockerfile optimizado:

```Dockerfile
# Etapa 1: Construcción
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build  # Si tu aplicación necesita un paso de compilación

# Etapa 2: Producción
FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/index.js ./
COPY --from=build /app/package.json ./

# Exponer el puerto y ejecutar la aplicación
EXPOSE 3000
CMD ["node", "index.js"]
```

---

#### **3.2 Explicación del Dockerfile optimizado**
- **Etapa 1 (`build`):**
  - Usamos `node:18` para instalar dependencias y compilar la aplicación.
  - El comando `npm run build` se usa si tu aplicación necesita un paso de compilación (por ejemplo, para aplicaciones React o TypeScript).
- **Etapa 2 (`node:18-alpine`):**
  - Usamos `node:18-alpine`, una imagen mucho más pequeña basada en Alpine Linux.
  - Copiamos solo los archivos necesarios desde la etapa de construcción (`node_modules`, `index.js`, y `package.json`).
  - Esto reduce el tamaño de la imagen final al eliminar herramientas y archivos innecesarios.

---

#### **3.3 Construir la imagen optimizada**
1. Construye la imagen optimizada:
   ```bash
   docker build -t node-hello-world:optimized .
   ```

2. Verifica el tamaño de la imagen:
   ```bash
   docker images
   ```
   Deberías ver algo como esto:
   ```
   REPOSITORY          TAG         IMAGE ID       CREATED          SIZE
   node-hello-world    optimized   xyz123456789   10 seconds ago   120MB
   node-hello-world    original    abcdef123456   5 minutes ago    1.02GB
   ```

   ¡Notarás que la imagen optimizada es mucho más pequeña!

---

#### **3.4 Ejecutar el contenedor optimizado**
1. Ejecuta el contenedor basado en la imagen optimizada:
   ```bash
   docker run -p 3000:3000 node-hello-world:optimized
   ```

2. Visita `http://localhost:3000` en tu navegador. Deberías ver "Hello World".

---

#### **3.5 Comparar tamaños de imágenes**
Compara los tamaños de las imágenes original y optimizada:
```bash
docker images
```
Verás algo como esto:
```
REPOSITORY          TAG         IMAGE ID       CREATED          SIZE
node-hello-world    optimized   xyz123456789   10 seconds ago   120MB
node-hello-world    original    abcdef123456   5 minutes ago    1.02GB
```

La imagen optimizada es significativamente más pequeña (¡de ~1GB a ~120MB!).

---

#### **3.6 Limpieza (opcional)**
Si quieres liberar espacio, puedes eliminar las imágenes y contenedores que ya no necesitas.

1. Eliminar contenedores detenidos:
   ```bash
   docker container prune
   ```

2. Eliminar una imagen específica:
   ```bash
   docker rmi <image_id>
   ```

3. **Encontrar el `<container_id>`:**
   Si necesitas eliminar un contenedor específico, puedes encontrar su ID usando:
   ```bash
   docker ps -a
   ```
   Esto lista todos los contenedores, incluidos los detenidos. Luego, usa:
   ```bash
   docker rm <container_id>
   ```

---

### **Paso 4: Comparar tamaños y documentar**

#### **4.1 Comparar tamaños de imágenes**
Primero, vamos a comparar los tamaños de las imágenes original y optimizada.

1. Ejecuta el siguiente comando para listar las imágenes:
   ```bash
   docker images -s
   ```

   Deberías ver algo como esto:
   ```
   REPOSITORY          TAG         IMAGE ID       CREATED          SIZE
   node-hello-world    optimized   xyz123456789   10 seconds ago   120MB
   node-hello-world    original    abcdef123456   5 minutes ago    1.02GB
   ```

   - La imagen **original** (`node-hello-world:original`) tiene un tamaño de ~1.02GB.
   - La imagen **optimizada** (`node-hello-world:optimized`) tiene un tamaño de ~120MB.

   ¡La imagen optimizada es aproximadamente **8.5 veces más pequeña** que la original!

---

#### **4.2 Crear un archivo `README.md`**
Ahora, documentaremos el proceso en un archivo `README.md`

   ```markdown
   # Optimización de una imagen de Docker usando Multi-Stage Builds

   Este proyecto demuestra cómo optimizar una imagen de Docker para una aplicación Node.js usando construcciones multi-etapa. El objetivo es reducir el tamaño de la imagen final y mejorar la seguridad.

   ## ¿Qué son las Construcciones Multi-Etapa?

   Las construcciones multi-etapa en Docker permiten usar múltiples imágenes `FROM` en un solo Dockerfile. Cada `FROM` inicia una nueva etapa de construcción. Esto permite copiar artefactos (como archivos compilados) de una etapa a otra, dejando atrás todo lo que no se necesita en la imagen final, lo que resulta en imágenes más pequeñas y eficientes.

   ## Pasos realizados

   1. **Selección de la imagen base**: Se eligió la imagen `node:18` como base debido a su popularidad y tamaño considerable.
   2. **Creación del Dockerfile original**: Se creó un Dockerfile para construir una aplicación simple de Node.js ("Hello World"). Este Dockerfile crea una imagen grande que incluye todas las herramientas de desarrollo.
   3. **Optimización con Multi-Stage Builds**: Se modificó el Dockerfile para usar dos etapas:
      - **Etapa de construcción**: Se usó `node:18` para instalar dependencias y compilar la aplicación.
      - **Etapa de producción**: Se usó `node:18-alpine` para copiar solo los archivos necesarios, resultando en una imagen mucho más pequeña.
   4. **Comparación de tamaños**: Se compararon los tamaños de las imágenes original y optimizada.

   ## Resultados

   | Imagen                  | Tamaño  |
   |-------------------------|---------|
   | Original (`node:18`)    | 1.02GB  |
   | Optimizada (`alpine`)   | 120MB   |

   La imagen optimizada es aproximadamente **8.5 veces más pequeña** que la original.

   ## Cómo ejecutar el proyecto

   1. **Asegúrate de haber construido la imagen *original* siguiendo los pasos del Paso 2.**
   2. Clona este repositorio:
      ```bash
      git clone <URL-del-repositorio>
      ```
   3. Construye la imagen optimizada:
      ```bash
      docker build -t node-hello-world:optimized .
      ```
   4. **Compara los tamaños de las imágenes (original y optimizada):**
      ```bash
      docker images -s
      ```
   5. Ejecuta el contenedor:
      ```bash
      docker run -p 3000:3000 node-hello-world:optimized
      ```
   6. Visita `http://localhost:3000` en tu navegador para ver la aplicación en funcionamiento.

   ## Limpieza

   Si deseas liberar espacio, puedes eliminar las imágenes y contenedores que ya no necesitas:

   - Eliminar contenedores detenidos:
     ```bash
     docker container prune
     ```

   - Eliminar una imagen específica:
     ```bash
     docker rmi <image_id>
     ```

   - Encontrar el `<container_id>`:
     ```bash
     docker ps -a
     ```

   - Eliminar un contenedor específico:
     ```bash
     docker rm <container_id>
     ```

   Las construcciones multi-etapa son una técnica poderosa para optimizar imágenes de Docker, reduciendo su tamaño y mejorando la seguridad. Este proyecto es un ejemplo práctico de cómo aplicarla en una aplicación Node.js.

