<div align="center">

# Guía - Instalación de Docker en Ubuntu

<img style="margin: 0 5px" width="88" src="../public/docker-logo.png">
<img style="margin: 0 5px" width="88" src="../public/ubuntu-logo.png">

</div>

## Instalación paso a paso

### 0. **Actualiza el sistema**

Antes de empezar con la instalación asegurate de actualizar tu sistema para evitar posibles errores a futuro.
```bash
sudo apt update
sudo apt upgrade -y
```

### 1. **Eliminar versiones anteriores (sí existen)**

Elimina las versiones antiguas o anteriores que estén almacenadas en tu sistema, esto permite una instalación mas limpia y segura, evitando posibles conflictos.

```bash
sudo apt remove -y docker docker-engine docker.io containerd runc || true
```

### 2. **Instalar dependencias necesarias**

Instala las dependencias necesarias para la instalación.

```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
```

### 3. **Añadir la llave GPG oficial**

Añade la llave GPG oficial para Docker para asegurarse de que los paquetes sean seguros y no hayan sido modificados por terceros.

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

### 4. **Añadir el repositorio de Docker**

Añade el repositorio oficial de Docker para obtener los recursos desde su sitio oficial y poder descargarlos.

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 5. **Actualizar e instalar Docker Engine**

Actualizamos el sistema mientras que a su vez instalamos el **Docker Engine**.

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 6. **Habilitar y ejecutar el servicio**

Permite inicializar y ejecutar el sistema de **Docker** para poder ser utilizado mediante la terminal.

```bash
sudo systemctl enable --now docker
```

### 7. **Probar la imagen "hello-world"**

Una vez ya instalado Docker pasaremos a probar una imagen base que viene por defecto con Docker, esta imagen es "hello-world" cuyo mensaje de espera será "Hello from Docker ...etc".

```bash
sudo docker run --rm hello-world
```

### 8. **Configuración opcional**

Permite ejecutar Docker sin necesidad del comando `sudo`, haciendo el proceso mas comodo, una vez ejecutado deberás reiniciar el sistema.

```bash
sudo usermod -aG docker $USER
```

---

## Notas finales y recomendaciones

- Para desarrollo local: Docker con los paquetes anteriores es suficiente.

- En producción: considerar seguridad (rootless, control de accesos, escaneo de imágenes, políticas de actualización).

- Para usar docker compose, usa `docker compose up` (Compose V2 se instala como plugin en los pasos anteriores).