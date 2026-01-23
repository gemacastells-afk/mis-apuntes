# 🗺️ Mapa Mental: Linux vs. HDFS

Es vital diferenciar dónde estamos trabajando. Tenemos dos sistemas de archivos paralelos que no se tocan directamente.

## 1. Los Dos Universos

| Universo | **Linux Local (Tu PC)** | **Hadoop (HDFS)** |
| :--- | :--- | :--- |
| **¿Qué es?** | El disco duro físico de tu máquina virtual. | Un sistema virtual repartido entre muchos nodos. |
| **¿Cómo empiezo?** | Comandos normales (`ls`, `cd`). | Siempre empieza por `hdfs dfs ...` |
| **¿Puedo entrar (`cd`)?** | ✅ Sí. Te mueves por las carpetas. | ❌ **NO**. No puedes "entrar". Solo puedes listar desde fuera. |

## 2. Diccionario de Traducción

Si quieres hacer algo en Hadoop, busca su equivalente:

| Acción | En Linux (Local) | En Hadoop (HDFS) |
| :--- | :--- | :--- |
| **Listar archivos** | `ls -l` | `hdfs dfs -ls /ruta` |
| **Crear carpeta** | `mkdir carpeta` | `hdfs dfs -mkdir /ruta/carpeta` |
| **Borrar archivo** | `rm archivo` | `hdfs dfs -rm /ruta/archivo` |
| **Ver contenido** | `cat archivo` | `hdfs dfs -cat /ruta/archivo` |
| **Cambiar permisos**| `chmod 777` | `hdfs dfs -chmod 777 /ruta` |

## 3. El "Puente" (Subir y Bajar datos)

Como son dos mundos separados, necesitamos comandos para mover archivos de uno a otro.

* **Subir (De Linux ➡ HDFS):**
    * `put`: `hdfs dfs -put mi_foto.jpg /fotos`
    * *(Significa: "Coge mi_foto.jpg de aquí y ponla en la carpeta /fotos de Hadoop")*

* **Bajar (De HDFS ➡ Linux):**
    * `get`: `hdfs dfs -get /fotos/mi_foto.jpg .`
    * *(Significa: "Trae la foto de Hadoop y déjala aquí mismo en mi Linux")*