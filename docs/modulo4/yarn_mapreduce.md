# 📚 Apuntes: Ecosistema Hadoop (YARN & MapReduce)

Este documento resume la evolución, arquitectura y funcionamiento del procesamiento de datos en Hadoop 2.0+.

---

## 1. Evolución: De MapReduce 1 a YARN 🚀

En la versión antigua de Hadoop (MR1), un solo componente (**JobTracker**) lo hacía todo, lo que generaba cuellos de botella al superar los **5,000 nodos**.  
Con la llegada de **YARN (Hadoop 2.0)**, se separaron las responsabilidades.

### 🔄 Tabla comparativa: El cambio de paradigma

| Concepto | MapReduce 1 (Obsoleto) | YARN (Moderno) |
|---|---|---|
| Gestión | **JobTracker**: Gestionaba recursos y monitoreaba tareas simultáneamente. | **ResourceManager**: Solo gestiona recursos globales (CPU/RAM). |
| Ejecución | **TaskTracker**: Ejecutaba tareas en nodos esclavos. | **NodeManager**: Gestiona los recursos de su nodo específico. |
| Recursos | **Slot**: Huecos fijos y rígidos para tareas. | **Container**: Paquete dinámico y abstracto de RAM y CPU. |

💡 **La clave:** YARN actúa como el “Sistema Operativo” del clúster, permitiendo que no solo corra MapReduce, sino también **Spark** o **Streaming**.

---

## 2. Arquitectura de YARN: Los 3 niveles de mando 🏛️
flowchart TB
  RM[👑 ResourceManager\n(gestiona recursos globales)] --> NM1[👷 NodeManager\n(nodo trabajador)]
  RM --> NM2[👷 NodeManager\n(nodo trabajador)]
  RM --> NM3[👷 NodeManager\n(nodo trabajador)]

  AM[🎼 ApplicationMaster\n(1 por aplicación)] --> RM
  AM --> NM1
  AM --> NM2
  AM --> NM3

  NM1 --> C1[📦 Containers\n(tareas)]
  NM2 --> C2[📦 Containers\n(tareas)]
  NM3 --> C3[📦 Containers\n(tareas)]

**YARN (Yet Another Resource Negotiator)** gestiona el hardware del clúster mediante tres componentes principales:

### 👑 ResourceManager (El Dueño)
- Ubicado en el nodo maestro.
- Tiene autoridad máxima sobre los recursos (RAM/CPU) de todo el clúster.
- Contiene un **Scheduler** (planificador) que decide quién recibe recursos.

### 👷 NodeManager (El Capataz)
- Hay uno en cada nodo esclavo (trabajador).
- Vigila su propia máquina y reporta el estado al ResourceManager.
- Crea los **Contenedores** donde se ejecutan las tareas finales.

### 🎼 ApplicationMaster (El Director de Orquesta)
- Se crea uno por cada aplicación o trabajo lanzado.
- Negocia recursos con el ResourceManager y da órdenes a los NodeManagers.
- Desaparece en cuanto el trabajo termina.

---

## 3. Configuración y administración ⚙️

Para que el sistema funcione, es necesario configurar y arrancar los servicios correctamente.

### 📄 Archivos de configuración (`/etc/hadoop`)
- `mapred-site.xml`: Se debe especificar que el framework es **yarn**.
- `yarn-site.xml`: Se define el hostname del ResourceManager y se activa el servicio `mapreduce_shuffle`.

### 🚀 Orden de arranque
1. **HDFS:** `start-dfs.sh` (Levanta NameNode y DataNodes).
2. **YARN:** `start-yarn.sh` (Levanta ResourceManager y NodeManagers).
3. **History Server (Opcional):** `mr-jobhistory-daemon.sh start historyserver`  
   - Utilidad: Permite ver los logs de trabajos ya terminados.

### 🌐 Interfaz web
Puedes monitorear todo en tiempo real en:  
`http://localhost:8088`

---

## 4. MapReduce: El flujo de trabajo 🛠️

flowchart LR
  IN[📥 Input (HDFS)] --> MAP[🔍 Mapper\n(clave, valor)]
  MAP --> SHUF[🔀 Shuffle & Sort\n(agrupa por clave)]
  SHUF --> RED[📊 Reducer\n(operación final)]
  RED --> OUT[📤 Output (HDFS)]
MapReduce es el paradigma para procesar datos en paralelo. Se divide en tres fases estrictas:

### Fase 1: Mapper (El Clasificador) 🔍
- **Entrada:** Datos en bruto (líneas de texto).
- **Acción:** Filtra y emite pares **Clave-Valor** (Ej: `pepe, 1`).
- **Nota:** No realiza cálculos globales, solo clasifica.

### Fase 2: Shuffle & Sort (El Organizador) 🔀
- **Acción:** Proceso automático que recoge las salidas de los mappers, las ordena alfabéticamente y las agrupa por clave.
- **Resultado:** El Reducer recibe algo como: `pepe, [1, 1, 1]`.

### Fase 3: Reducer (El Contador) 📊
- **Acción:** Itera sobre la lista de valores y realiza la operación final (suma, media, etc.).
- **Salida:** El resultado final se guarda en **HDFS**.

---

## 5. MapReduce con Python (Hadoop Streaming) 🐍

Aunque nativamente se usa Java, **Hadoop Streaming** permite usar Python mediante la entrada y salida estándar (`stdin/stdout`).  
Se usa principalmente por su potencia en IA y librerías de datos.

### 📝 Comando de ejecución (Bash)

```bash
hadoop jar /ruta/a/hadoop-streaming.jar \
  -files mapper.py,reducer.py \        # Envía los scripts a los nodos
  -mapper mapper.py \                  # Script para la fase Map
  -reducer reducer.py \                # Script para la fase Reduce
  -input /entrada/hdfs \               # Origen de datos
  -output /salida/hdfs                 # Destino de resultados

  ---

## 6. Estrategias de Ordenación (Sorting) 📉

MapReduce ordena por **clave** automáticamente. ¿Pero qué pasa si queremos ordenar por **valor** (ej: ranking de palabras más repetidas)?

### A. Ordenar en Linux (Archivos pequeños) 🐧
[cite_start]Si el resultado final es pequeño (<128MB), es más rápido bajarlo a local y ordenar con comandos de sistema [cite: 416-418].

```bash
hdfs dfs -cat /salida/part-* | sort -k2,2n > top_usuarios.txt

```Notas:

-k2,2n: ordena por la segunda columna (2), tratándola como número (n).

-r: añádelo si quieres orden inverso (descendente), por ejemplo:

hdfs dfs -cat /salida/part-* | sort -k2,2nr > top_usuarios.txt
B. Ordenar en Hadoop (archivos gigantes) 🐘

Para Big Data real, se debe configurar el trabajo para que ordene globalmente usando comparadores específicos.

Requiere usar la clase KeyFieldBasedComparator.

Se configura con opciones -D en el comando de ejecución.

Ejemplo (plantilla orientativa con -D):
```bash
hadoop jar /ruta/a/hadoop-streaming.jar \
  -D mapreduce.job.output.key.comparator.class=org.apache.hadoop.mapreduce.lib.partition.KeyFieldBasedComparator \
  -D mapreduce.partition.keycomparator.options="-k2,2nr" \
  -files mapper.py,reducer.py \
  -mapper mapper.py \
  -reducer reducer.py \
  -input /entrada/hdfs \
  -output /salida/hdfs
  ```

