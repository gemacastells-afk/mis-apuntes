# El Ecosistema Hadoop

## 1. Arquitectura: Maestro - Esclavo
Hadoop funciona como un ejército. Un nodo manda y los demás obedecen.

* **Master (Maestro):** Gestiona, coordina y sabe dónde está todo.
* **Slave (Esclavo):** Hace el trabajo sucio (guardar datos o procesar cálculos).
* **Commodity Hardware:** Hadoop está diseñado para ejecutarse en ordenadores "baratos" y normales, no en superordenadores.

## 2. Los Dos Pilares de Hadoop

### A. HDFS (Almacenamiento)
Es el sistema de archivos distribuido. Rompe los archivos en bloques y los reparte.
* **NameNode (Maestro):** El índice. Sabe en qué nodo está cada trozo de archivo.
* **DataNode (Esclavo):** El almacén. Guarda los bloques de datos físicamente.
* **Replicación:** Por defecto, cada bloque se guarda 3 veces en distintos nodos para evitar pérdidas si un disco se rompe.

### B. YARN (Procesamiento)
Es el sistema operativo del clúster. Reparte la RAM y la CPU.
* **ResourceManager (Maestro):** Decide cuántos recursos le da a cada tarea.
* **NodeManager (Esclavo):** Vigila el uso de CPU/RAM en cada máquina.

## 3. MapReduce vs Spark
Son las formas de "trabajar" con los datos.

!!! warning "MapReduce (El Clásico)"
    * Trabaja escribiendo mucho en disco duro.
    * Es lento y por lotes (Batch).
    * Bueno para procesos nocturnos que no tienen prisa.

!!! success "Apache Spark (El Moderno)"
    * Trabaja en memoria RAM (hasta 100 veces más rápido).
    * Permite procesamiento en tiempo real (Streaming).
    * Es la evolución natural de MapReduce.

## 4. El "Zoológico" (Herramientas)
* **Hive:** Para lanzar consultas SQL sobre Hadoop.
* **Pig:** Lenguaje de script para procesar datos (ETL).
* **Sqoop:** Mueve datos entre Hadoop y Bases de Datos SQL (Oracle, MySQL).
* **Flume:** Mueve datos de streaming (logs, Twitter).

---
## 3. Arquitectura Interna: La Memoria del NameNode

El NameNode es el cerebro de Hadoop. Para no "perder la memoria" si se apaga, utiliza dos archivos críticos que se guardan en el disco duro (normalmente en `/datos/namenode/current`).

### A. Los Archivos de Metadatos
1.  **`fsimage` (La Foto Fija):**
    * Es una copia completa ("snapshot") del estado del sistema de ficheros en un momento concreto.
    * Contiene el inventario de todos los directorios y archivos.
    * **Analogía:** Es el "Inventario Anual" de una biblioteca.

2.  **`edits` (El Diario de Cambios):**
    * Es un registro log de cada pequeña operación que ocurre después del último `fsimage` (crear un archivo, borrarlo, etc.).
    * **Analogía:** Es la libreta de notas donde el bibliotecario apunta lo que pasa día a día .

3.  **`VERSION`:**
    * Contiene identificadores únicos como el `clusterID`. Es el "DNI" del clúster. Si formateas el NameNode, este ID cambia y los DataNodes dejan de reconocer al jefe.

### B. El Proceso de Checkpoint (Punto de Control)
Cuando el NameNode arranca, tiene que leer el `fsimage` y aplicar todos los cambios del `edits`. Si el `edits` es gigante, el arranque es lentísimo.

* **¿Qué es el Checkpoint?** Es el proceso de fusionar el `fsimage` viejo + el `edits` actual para crear un **nuevo `fsimage` actualizado** y vaciar el registro de cambios.
* **Safe Mode (Modo Seguro):** Es un estado de "solo lectura". El clúster se pone en pausa (no se puede escribir) para realizar tareas de mantenimiento o cuando detecta problemas 

---

## 4. Guía de Operaciones Básicas (Start/Stop)

Para trabajar con Hadoop, primero debemos "levantar" los servicios (demonios). Si no lo hacemos, recibiremos errores de `Connection Refused`.

### 🟢 Encender el Clúster (HDFS)
Se debe ejecutar siempre que encendamos la máquina virtual.

```bash
start-dfs.sh
```

* **Qué hace:** Arranca el NameNode (maestro), los DataNodes (esclavos) y el SecondaryNameNode.
* **Cuándo usarlo:** Al inicio de la sesión.

### 🔴 Apagar el Clúster

Es recomendable hacerlo antes de apagar la máquina virtual para evitar que los archivos de metadatos se corrompan.

```bash

stop-dfs.sh
```

* **Qué hace**: Detiene todos los procesos de forma ordenada.


🔍 **Verificar el Estado (JPS)**
El comando jps (Java Virtual Machine Process Status Tool) es el "médico" que nos dice qué procesos están vivos.

```bash

jps
```
**Salida Correcta (Deben aparecer estos 3):**

1. **NameNode:** El jefe. Si no está, no funciona nada.

2. **DataNode:** El trabajador. Si no está, no tenemos dónde guardar datos.

3. **SecondaryNameNode:** El ayudante para los checkpoints.

**Nota:** Si al hacer jps solo sale el número de proceso (ej: 1234 Jps) y nada más, significa que Hadoop está APAGADO.