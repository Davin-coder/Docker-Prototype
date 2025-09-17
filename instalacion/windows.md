
<div align="center">

# Guía - Instalación de Docker en Windows

<img style="margin: 0 5px" width="88" src="../public/docker-logo.png">
<img style="margin: 0 5px" width="88" src="../public/windows-logo.png">

</div>

## 1) ¿Qué es WSL 2?
**WSL 2** (Windows Subsystem for Linux 2) es la capa oficial de Windows que ejecuta un **kernel de Linux real** dentro de una VM ligera e integrada al sistema. Para Docker en Windows es importante porque la mayoría de imágenes y herramientas de contenedores están pensadas para Linux (usan namespaces, cgroups, overlayfs, etc.), y WSL 2 aporta justamente ese entorno sin tener que instalar una máquina virtual pesada aparte.

---

## 2) Arquitectura en Windows (WSL 2 + Docker Desktop)
- **Docker Desktop** trae el engine y la integración con **WSL 2** para contenedores Linux.
- Trabajarás dentro de una **distro WSL** (ej: *Ubuntu*).
- Mantente en **Linux containers** (solo cambia a *Windows containers* si realmente necesitas imágenes Windows).
- **No** instales `apt install docker` dentro de WSL: crea engines duplicados y conflictos. Usa **Docker Desktop + WSL Integration**.

**Comprobar distros y versión**
```powershell
wsl -l -v     # lista distros y versión (2 recomendado)
wsl --set-default Ubuntu
wsl --set-version Ubuntu 2
```
Dentro de WSL:
```bash
echo $WSL_DISTRO_NAME  # muestra la distro actual
```

**Activar integración en Docker Desktop**
- Settings → **General** → *Use the WSL 2 based engine*.
- Settings → **Resources → WSL Integration** → activa el switch de tu distro (p. ej. *Ubuntu*).

---

## 3) Instalación paso a paso (Windows 10/11)
1. **Habilitar WSL 2** (PowerShell *Admin*):
    ```powershell
    dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
    wsl --install -d Ubuntu
    wsl --set-default-version 2
    ```
2. **Instalar Docker Desktop** y activar WSL Integration para tu distro.
3. (Opcional) Inicia sesión en **Docker Hub** desde Docker Desktop.

> **Rendimiento:** guarda tus proyectos dentro de WSL. Acceso desde Explorer: `\\wsl$\Ubuntu\home\<tu_usuario>\...`

---

## 4) Configuración Uso de Memoría
Al ejecutar **Docker Desktop** este por defecto no tiene un limite de memoria asignada, provocando que el uso de la **memoria RAM** se siga incrementando constantemente, para esto se propone como solución principal implementar un **tope global** de memoria RAM dentro de nuestra distro, todo dentro de un archivo de configuración ya existente llamado `.wslconfig`.

**1. Cerrar Docker Desktop** si lo tienes abierto, asegurate de no tenerlo corriendo en segundo plano.

**2. Apagar WSL** desde una terminal PowerShell o CMD de la siguiente manera:
```cmd
wsl --shutdown
```

**3. Editar .wslconfig** en la siguiente ubicación `C:\Users\tu_usuario\.wslconfig`, puedes utilizar el comando `notepad` para editar el archivo, asegurate que contenga como minimo la siguiente información:
```ini
[wsl2]
memory=2GB      # este será el limite de RAM qué se podrá usar
processors=4      # define la cantidad de nucleos (opcional)
```
**Crearlo sí no existe:**

Asegurate de estar en la siguiente ubicación `C:\Users\tu_usuario`.
```cmd
notepad .wslconfig
```

**4. Abrir Docker Desktop** nuevamente para que a la hora de iniciar WSL 2 reconozca la configuración previa para el uso de la memoria RAM.

---

## 5) Comprobación rápida
```bash
docker --version
docker run --rm hello-world
```
**Prueba visible de puertos:**
```bash
docker run -d --name web -p 8080:80 nginx
# abrir http://localhost:8080
docker logs web | tail -n 20
docker rm -f web
```
**Qué queda corriendo**
- `hello-world`: finaliza solo (no deja nada activo).
- `-d` / Compose `up -d`: quedan **en segundo plano** hasta que los detengas.

---

### Conclusión
- Docker te da **contenedores** (procesos aislados) creados desde **imágenes**; en Windows se ejecutan vía **Docker Desktop + WSL 2**.
- Con **Dockerfile** construyes tus propias imágenes.
- Con **Docker Compose** declaras y levantas proyectos completos de forma reproducible.
- Trabaja en **WSL**, nombra tus contenedores, usa `.dockerignore` y limpia recursos periódicamente.
