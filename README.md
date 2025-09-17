<div align="center">

# Guía Basica - Conociendo Docker

<img style="margin: 0 5px" width="88" src="https://images.icon-icons.com/2699/PNG/512/docker_tile_logo_icon_168248.png">

</div>

## Introducción 
Te doy la bienvenida a mi esta guía basica la cual te introducirá en el uso de Docker desde cero, permitiendote conocer sobre los conceptos claves para entender sobre su funcionamiento, como el manejo de volúmenes, construir imagenes, orquestar contenedores mediante **Docker Compose**, etc.

---

## ¿Qué es Docker?
Docker es una plataforma para **empaquetar, distribuir y ejecutar** aplicaciones como **contenedores**. Un contenedor es un **proceso aislado** que se crea a partir de una **imagen** (plantilla inmutable).

**Ventajas clave**
- **Portabilidad:** misma imagen, mismo comportamiento en cualquier host.
- **Reproducibilidad:** entornos consistentes (menos "works on my machine").
- **Ligereza:** arranca en segundos (comparte kernel, no es una VM completa).
- **Dev/CI/CD:** builds, tests y despliegues más confiables.

Para su instalación depende de tu sistema operativo, si quieres aprender a como instalarlo correctamente en Windows dirigete aquí [Instalación para Windows](./instalacion/windows.md).

---

## Conceptos Básicos para Docker

### Imágenes
---
Una **imagen** en Docker se refiere a una plantilla inmutable compuesta de varias capas (layers) que describe un sistema de archivos y metadatos. Esta misma se versiona mediante **tags** (aplication:1.0, latest), guardandose posteriormente en un **registry** (Docker Hub o un registro privado). Toda esta imagen se genera mediante un archivo llamado `Dockerfile` y se comparte.

*ejemplo breve:*
```bash
# Obtiene la imagen y la versión de esta.
docker pull nginx:1.25
```

### Volúmenes
---
Los volúmenes son mecanismos de **almacenamiento de datos** o que permiten compartir archivos entre el **host** y **contenedor**. Tipos de volúmenes mas comunes: **named volumes** (gestionados por Docker) y **bind mounts** (ruta del host).

De forma mas simple, un **volumen** vendría siendo un sitio gestionado por Docker que vive totalmente fuera del sistema de archivos efímero de nuestro contenedor, sirviendo como un **almacén de datos** que deben persistir aún deteniendo, actualizando o borrando el contenedor.

- **Volúmenes gestionados por Docker**
    ```bash
    docker run -d -v nombre_volumen:/ruta/en/contenedor imagen
    ```

- **Bind mounts (enlaces a carpetas locales)**
    ```bash
    docker run -d -v /ruta/en/host:/ruta/en/contenedor imagen
    ```

Trabaja con volúmenes **gestionados por Docker** para datos críticos (bases de datos). Para desarrollo donde necesitas ver en tiempo real los cambios locales, es mejor la alternativa con **Bind mounts**.

**Volúmenes y rutas en Windows/WSL**
- **Recomendado:** trabajar en el filesystem de **WSL** (`~/proyecto`) para mejor I/O.
- Montajes típicos:
  - En WSL: `-v ~/mysite:/usr/share/nginx/html:ro`
  - En PowerShell (carpeta Windows): `-v "${PWD}\mysite:/usr/share/nginx/html:ro"`
- Evita mezclar rutas WSL ↔ Windows en proyectos con muchísimos archivos.
- Para bases de datos, usa volúmenes **nombrados** (persistentes entre recreaciones).

---

### Contenedores
---
Un **contenedor** se le conoce como la **instancia en ejecución** de una imagen, es decir, es un **proceso aislado** con su propio sistema de archivos, red y límites de recursos. Estos contenedores por defecto actúan de forma **efímera** (si se borrán, los cambios que estén en volúmenes se pierden).

### Redes
---
Son las que permiten que los contenedores se **descubran** y **comuniquen** entre sí y con el *host*. El tipo de red que se usa por defecto es **bridge**; también existiendo **host** o **none**.

### CLI/Daemon
---
Al ejecutar Docker mediante la terminal lo hacemos mediante el comando `docker`, este vendría siendo el **cliente** (CLI) que se comunica por medio de una **API REST** con el **Docker Engine** (Daemon, `dockerd`). En Linux el daemon corre en el host; en Mac/Windows se usa una **VM/WSL2** gestionada por **Docker Desktop**.

### Dockerfile
---
El `Dockerfile` es un archivo de texto con **instrucciones** para construir una imagen, sirviendo como una receta, cada instrucción **genera una capa cacheable**. Mediante el comando `docker build`, la CLI lee tu `Dockerfile`, envía el contexto de la **build** (tu carpeta) al daemon y produce una imagen lista para ejecutar.

*Estructura basica:*
```Dockerfile
FROM <imagen-base> # ej: node
WORKDIR /app
COPY . .
RUN <comandos-de-instalación/build>
EXPOSE 8080
USER 10001
CMD ["binario", "arg1"]
```

**Instrucciones más comunes**
- `FROM`: Describe la base de imagen para el proyecto.
- `WORKDIR`: Fija un directorio de trabajo dentro del contenedor.
- `COPY` / `ADD`: copia los archivos del proyecto (funcionando con **COPY**, mientras que **ADD** solo sí necesitas descomprimir o URLs).
- `RUN`: Ejecuta comandos en build (instalar paquetes, compilar...).
- `ENV`: Variables de entorno en la **imagen**.
- `ARG`: Variables **SOLO** durante el build (--build-arg).
- `EXPOSE`: Documenta el puerto que escucha la app (no publica el puerto por sí solo).
- `USER`: ejecuta el contenedor como usuario no root (mejor seguridad).
- `CMD`: Comando por defecto al arrancar el contenedor.
- `ENTRYPOINT`: Define un programa fijo que siempre se ejecuta, se usa junto con **CMD**.

---

## Comandos esenciales
- **Listar contenedores**
    ```bash
    docker ps        # solo en ejecución
    docker ps -a     # incluye detenidos
    ```
- **Imágenes**
    ```bash
    docker images
    docker pull nginx
    docker rmi nginx
    ```
- **Ejecutar / Inspeccionar**
    ```bash
    docker run -it --rm ubuntu bash
    docker logs -f <nombre>
    docker exec -it <nombre> sh
    ```
- **Limpiar**
    ```bash
    docker container prune
    docker image prune -a
    docker system prune -af --volumes   # cuidado: borra TODO lo no usado
    ```

---

## Ciclo de vida del contenedor
```bash
docker create --name demo nginx   # prepara (no inicia)
docker start demo                 # inicia
docker stop demo                  # detiene (graceful)
docker kill demo                  # forzado (SIGKILL)
docker restart demo               # reinicia
docker rm demo                    # elimina
```
**Puertos y exposición**
- `-p 8080:80` mapea el puerto 80 del contenedor al 8080 del host.
- Para restringir a la máquina local:
  ```bash
  docker run -d -p 127.0.0.1:8080:80 nginx
  ```

---

## Construir imágenes propias (Dockerfile)
**Ejemplo simple (muestra IP con ifconfig):**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y net-tools && rm -rf /var/lib/apt/lists/*
CMD ["ifconfig"]
```
**Build & Run**
```bash
docker build -t ubuntu-ifconfig .
docker run --rm -it ubuntu-ifconfig
# Sobrescribir CMD en runtime:
docker run --rm ubuntu-ifconfig ip a
```
**Buenas prácticas**
- Crea `.dockerignore` (ej. `.git`, `node_modules`, `*.log`) para no enviar basura al *build context*.
- Usa `WORKDIR` y `COPY` para apps reales.
- Aprovecha **caché de capas**; si cambias líneas altas, se invalidan las de abajo.

---

## Docker Compose - Proyectos Compose
Un **proyecto Compose** es todo lo definido en `compose.yaml` dentro de una carpeta: servicios, redes y volumenes. El **nombre del proyecto** por defecto es el nombre de la **carpeta** (puedes cambiarlo con `-p` o `COMPOSE_PROJECT_NAME`).

Es el conjunto completo que Docker Compose gestiona desde una carpeta: los servicios (contenedores), sus volumenes, redes, variables y dependencias definidos en un archivo compose.yaml (o docker-compose.yml).

**Ejemplo mínimo**
```yaml
# compose.yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./mysite:/usr/share/nginx/html:ro
```
```bash
mkdir -p mysite && echo "¡Hola Compose!" > mysite/index.html
docker compose up -d
docker compose ps
docker compose logs -f web
docker compose exec web sh
docker compose down           # + -v si quieres borrar volúmenes anónimos
```
**Notas**
- Usa el comando moderno: `docker compose` (con espacio). No `docker-compose`.
- `docker compose ps` lista **SOLO** los contenedores del proyecto actual.
- Variables de entorno: `.env` en la misma carpeta (Compose lo lee automaticamente).

---

## Diagnóstico y errores comunes
- **`docker: command not found` en WSL:** habilita *WSL Integration* para esa distro en Docker Desktop (Settings → Resources → WSL Integration). Reinicia con `wsl --shutdown`.
- **Varias distros:** verifica en qué estás (`echo $WSL_DISTRO_NAME`) y activa **esa** en Docker Desktop.
- **Usar `docker.exe` desde WSL:** útil para validar que Docker Desktop está instalado.
  ```bash
  docker.exe --version
  docker.exe run --rm hello-world
  ```
- **No instales `apt install docker`** dentro de WSL (duplica el engine).
- **Conflictos de puertos:** cambia el host o puerto (`-p 127.0.0.1:8081:80`).

---

## Limpieza y apagado rápido
```bash
# parar y borrar un contenedor
docker rm -f <nombre>

# parar todo lo que corre
docker stop $(docker ps -q)

# podar recursos no usados
docker system prune -af --volumes

# apagar WSL (corta contenedores)
wsl --shutdown
```
Salir de Docker Desktop: **Docker Desktop → Quit**.

---

## Cheatsheet esencial
```bash
# estado global
docker ps             # en ejecución
docker ps -a          # todos
docker images         # imágenes locales

# ejecución
docker run --rm hello-world
docker run -d --name web -p 8080:80 nginx

# inspección
docker logs -f web
docker exec -it web sh

# ciclo de vida
docker stop web && docker rm web
docker create --name demo nginx && docker start demo

# build
docker build -t miimagen .
docker run --rm -it miimagen

# compose
docker compose up -d
docker compose ps
docker compose logs -f
docker compose down -v
```

---

## Mini glosario
- **Imagen:** plantilla inmutable; se versiona con etiquetas (tags).
- **Contenedor:** proceso aislado creado a partir de una imagen.
- **Dockerfile:** archivo con instrucciones para construir una imagen.
- **Volumen:** almacenamiento persistente gestionado por Docker.
- **Proyecto Compose:** conjunto de servicios/recursos definidos en `compose.yaml`.
- **Registry:** servidor de imágenes (Docker Hub, GitHub Container Registry…).

---