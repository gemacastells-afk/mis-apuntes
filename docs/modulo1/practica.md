# Práctica Unidad 1: Instalación de Hadoop

Esta página documenta la entrega de la práctica de la Unidad 1, detallando el proceso de instalación base de Hadoop en modo Pseudo-distribuido.

## 🎯 Objetivos de la Práctica
Realizar una instalación funcional que cumpla con los siguientes requisitos:
1.  **Código fuente:** Ubicado en `/opt/hadoop`.
2.  **Java:** Instalación de JDK8 (Open o Privado).
3.  **SSH:** Instalación y configuración para acceso sin contraseña.
4.  **Variables de Entorno:** Configuración para ejecutar comandos desde cualquier ruta.

---

## 🛠️ Desarrollo de la Instalación

### 1. Despliegue del Código Fuente
Se ha descargado y descomprimido Hadoop en el directorio estándar de Linux para software opcional.

* **Ruta de instalación:** `/opt/hadoop`
* **Permisos:** El usuario actual es propietario de la carpeta.

```bash
# Verificación
ls -ld /opt/hadoop
```

### 2. Instalación de Java (JDK 8)
Hadoop requiere Java 8 para funcionar correctamente. Se ha instalado openjdk-8-jdk
``` bash
# Verificación de versión
java -version
# Salida esperada: openjdk version "1.8.0_..."
```
### 3. Configuración SSH
Para que los scripts de arranque (start-dfs.sh, start-yarn.sh) funcionen, el nodo debe poder conectarse a sí mismo sin pedir contraseña.

   * **Clave generada con**: ssh-keygen -t rsa

    * **Clave copiada con**: ssh-copy-id localhost

```

# Prueba de conexión (no debe pedir password)
ssh localhost
```
### 4. Variables de Entorno (.bashrc)
Se han añadido las rutas al archivo .bashrc para que el sistema reconozca los comandos de Hadoop globalmente.

**Configuración aplicada:**

```

export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/opt/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

```

**Verificación:**

```

# Ejecutado desde la carpeta HOME (~)
hadoop version

```