# Guía de Despliegue Paso a Paso: Replicación de MISP v2.5 en Ubuntu 24.04 LTS

Esta guía detalla el procedimiento exacto para replicar la infraestructura del laboratorio de Threat Intelligence utilizando **Oracle VirtualBox** y **Ubuntu Server 24.04 LTS**. Al finalizar este procedimiento, contarás con una instancia completamente funcional, segura y alimentada por feeds OSINT globales.

---

## 🛠️ Fase 1: Requisitos de Hardware y Componentes Mínimos
Para evitar cuellos de botella durante la compilación de las dependencias de Python y el aprovisionamiento de las bases de datos relacionales, configura la máquina virtual en VirtualBox con los siguientes parámetros mínimos:

*   **SO:** Ubuntu 24.04 LTS (64-bits)
*   **Procesadores (vCPUs):** Mínimo 2 núcleos (Recomendado: 4 núcleos).
*   **Memoria RAM:** Mínimo 4 GB (Recomendado: 8 GB).
*   **Disco Duro Virtual:** 40 GB a 50 GB (Almacenamiento dinámico VDI).

### 🌐 Configuración Crítica de Red (Modo Bridge)
Para poder interactuar con la interfaz gráfica de MISP directamente desde el navegador de tu sistema operativo anfitrión (Windows/macOS), debes configurar el adaptador de red en caliente o antes de encender la máquina virtual:
1. Ve a **Configuración de la VM** ➔ **Red** ➔ **Adaptador 1**.
2. En el campo *Conectado a*, selecciona **Adaptador Puente (Bridged Adapter)**.
3. En *Nombre*, elige tu tarjeta de red activa del host (Wi-Fi o Ethernet).
4. Despliega *Avanzadas* y marca **Cable conectado**.

---

## 🚀 Fase 2: Configuración del Sistema y Acceso Remoto
Una vez completada la instalación por defecto de Ubuntu Server, inicia sesión e instala el servidor de OpenSSH para administrar el despliegue de forma cómoda desde la terminal de tu sistema anfitrión:

```bash
# 1. Actualizar el índice de paquetes e instalar OpenSSH
sudo apt update
sudo apt install openssh-server curl git coreutils openssl build-essential -y

# 2. Habilitar el daemon de SSH
sudo systemctl enable --now ssh

# 3. Identificar la dirección IP numérica asignada por el router por DHCP
ip a
```
*Identifica tu interfaz principal (ej. `enp0s3`) y anota la IP asignada (ej. `192.168.0.102`). Conéctate desde tu Windows ejecutando: `ssh tu_usuario@IP_DE_LA_VM`*.

---

## 📦 Fase 3: Despliegue Automatizado Core (Rama v2.5)
Para garantizar la compatibilidad con las librerías modernas de **PHP 8.3** y evitar los bucles infinitos de permisos cruzados de Git con submódulos como *PyMISP*, ejecuta el script nativo de la rama principal **sin anteponer la palabra `sudo`**:

```bash
# 1. Configurar la excepción global de seguridad en Git
sudo git config --system --add safe.directory /var/www/MISP

# 2. Clonar el repositorio oficial e ingresar al directorio de scripts
cd /tmp
git clone https://github.com
cd MISP/INSTALL/

# 3. Lanzar el instalador automatizado para Ubuntu 24.04
./INSTALL.ubuntu2404.sh
```
*Anota las credenciales del administrador administrador generadas al final del proceso (`/root/misp_settings.txt`).*

---

## 🔧 Fase 4: Mitigación de Errores Web y Ajuste de IP Dinámica
Si la IP de tu VM cambia debido al DHCP de tu router tras un reinicio, o si el navegador bloquea el inicio de sesión mostrando el recuadro rojo de **"Request failed due to an unexpected error"** (Violación de políticas CSP / Excepción Blackhole), ejecuta este bloque correctivo sustituyendo `TU_IP_ACTUAL` por tu dirección numérica de hoy:

```bash
# 1. Corregir las URL base en el archivo config.php
sudo sed -i "s/'baseurl' => '.*'/'baseurl' => 'https://TU_IP_ACTUAL'/g" /var/www/MISP/app/Config/config.php
sudo sed -i "s/'external_baseurl' => '.*'/'external_baseurl' => 'https://TU_IP_ACTUAL'/g" /var/www/MISP/app/Config/config.php

# 2. Desactivar temporalmente la directiva estricta de CSP para estabilizar el login
sudo sed -i "s/'csp_enforce' => true,/'csp_enforce' => false,/g" /var/www/MISP/app/Config/config.php

# 3. Inyectar la regla de preservación de host en la plantilla de Apache
sudo sed -i '/<VirtualHost \*:443>/a \    ProxyPreserveHost On' /etc/apache2/sites-available/misp-ssl.conf

# 4. Vaciar la caché persistente del framework y reiniciar el servidor web
sudo rm -rf /var/www/MISP/app/tmp/cache/persistent/*
sudo rm -rf /var/www/MISP/app/tmp/cache/models/*
sudo systemctl restart apache2 redis-server
```

---

## 🔑 Fase 5: Restablecimiento de Credenciales de Práctica
Si la contraseña aleatoria del administrador de fábrica falla en la interfaz web por un problema de sintaxis del formulario seguro, puedes forzar una clave alfanumérica limpia directamente desde la consola con el controlador moderno de CakePHP:

```bash
# Reemplaza 'ClaveSegura123!' por tu contraseña de elección (Mínimo 12 caracteres)
sudo -u www-data /var/www/MISP/app/Console/cake user change_pw admin@admin.test ClaveSegura123!

# Reiniciar los daemons para limpiar las sesiones previas en memoria
sudo systemctl restart apache2 redis-server
```

---

## 📊 Fase 6: Habilitación de Inteligencia OSINT
Una vez dentro del portal web (`https://TU_IP_ACTUAL`), ejecuta estas acciones para culminar la ingesta de inteligencia:
1. Ve a **Event Actions** ➔ **List Taxonomies** y haz clic en `(enable all)` en las filas de **`tlp`** y **`pap`** para activar las sub-etiquetas.
2. Ve a **Sync Actions** ➔ **Feeds**, haz clic en *Load default feed metadata*, selecciona **CIRCL OSINT Feed**, **Botvrij.eu** y **URLhaus**, y actívalos haciendo clic en *Enable selected* y *Cache all feeds*.
