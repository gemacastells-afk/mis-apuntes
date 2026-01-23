# Instalación de Hadoop

!!! info "Nota"
    Estos apuntes están basados en la UD 2.2 y la práctica de clase.

## 1. Requisitos Previos
* **Java 8:** Obligatorio. Ruta típica en mi máquina: `/usr/lib/jvm/java-8-openjdk-amd64`.
* **SSH:** Debe estar configurado para acceder sin contraseña (`ssh localhost`).

## 2. Variables de Entorno
Añadir al final del archivo `.bashrc`:

```bash
export HADOOP_HOME=/opt/hadoop
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

## 3. Configuración de Hadoop

Editar el archivo hadoop-env.sh:
```bash

export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

### 4. Comandos de Arranque

Para levantar los demonios desde la terminal:

    HDFS: start-dfs.sh

    YARN: start-yarn.sh (si lo usamos)


---

