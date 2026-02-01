# UD 5: YARN Scheduler
**Asignatura:** Grado de Especialización Big Data e IA [cite: 5]
**Centro:** IES Pere Maria Orts [cite: 4]
**Autor:** Jose Manuel Barrera Arroyo [cite: 3]

---

## 1. Introducción al Scheduler de YARN
El **Scheduler** es el componente de YARN responsable de la partición de los recursos de todo el clúster y de la asignación de procesos a cada una de esas particiones[cite: 19].

### Evolución Histórica
* **MapReduce 1 (MR1):** La distribución de recursos era homogénea para todo el clúster[cite: 21]. Se utilizaba el *Fair Scheduler*, el cual no permitía realizar particiones de recursos de forma jerárquica[cite: 21].
* **YARN:** Implementa una gestión de recursos basada en una **estructura de árbol**[cite: 22]. Este modelo se conoce como **Capacity Scheduler**[cite: 22, 25].

---

## 2. Interfaz de Usuario y Monitorización
Es posible supervisar la gestión de recursos a través de la interfaz web de YARN[cite: 38, 39].

### Métricas de Configuración (Scheduler Metrics)
En la interfaz podemos observar límites críticos para el funcionamiento del clúster[cite: 75]:
* **Minimum Allocation:** El recurso mínimo que se asigna a cada contenedor (ejemplo: `<memory:1024, vCores:1>`)[cite: 85].
* **Maximum Allocation:** El límite máximo de recursos que puede solicitar un solo proceso (ejemplo: `<memory:8192, vCores:4>`)[cite: 86].

### Jerarquía por Defecto
Si no se realiza una configuración específica, YARN crea automáticamente[cite: 119, 120]:
1.  Una cola general llamada **root**.
2.  Una subcola llamada **default**, que tiene asignado el **100%** de los recursos del clúster.

---

## 3. Configuración: `capacity-scheduler.xml`
La distribución de recursos en forma de "colas" se define en el archivo de configuración `capacity-scheduler.xml`[cite: 190, 191]. Este archivo se encuentra junto al resto de archivos de configuración de Hadoop[cite: 192].

### Propiedades Principales
1.  **Definición de colas (`queues`):** Indica qué colas existen en un nivel determinado[cite: 197, 198].
    * *Ejemplo:* `yarn.scheduler.capacity.root.queues` con valor `default, prod, desa` creará tres colas que cuelgan del nodo raíz[cite: 221, 222, 237, 238].
2.  **Asignación de Capacidad (`capacity`):** Indica el porcentaje de recursos asignado a cada cola[cite: 199].
    * *Ejemplo:* `yarn.scheduler.capacity.root.default.capacity` con valor `30` asigna el 30% a esa cola[cite: 228].

> **Nota:** Para añadir elementos a un nivel, los nombres de las colas se separan por comas en la configuración[cite: 215, 217].

---

## 4. Gestión de Recursos y Colas
Una vez modificado el archivo de configuración, existen dos formas de aplicar los cambios[cite: 243, 244]:

1.  **Reinicio del servicio:** Parar y arrancar Hadoop[cite: 245].
2.  **Refresco en caliente:** Ejecutar el comando `yarn rmadmin -refreshQueues`[cite: 248].
    * *Riesgos:* Este comando puede fallar si la cola está en uso (estado `RUNNING`), especialmente si intentas convertir una cola de tipo "hoja" en una cola "padre"[cite: 249, 252, 253].

---

## 5. Asignación de Procesos a Colas
Para ejecutar un trabajo de MapReduce en una cola específica (distinta a la `default`), se debe añadir un parámetro en la línea de comandos[cite: 266, 267, 268]:

**Parámetro:** `-Dmapreduce.job.queuename=nombre_de_la_cola` [cite: 270]

**Ejemplo de ejecución:**
```bash
hadoop jar hadoop-mapreduce-examples.jar wordcount -Dmapreduce.job.queuename=facts/libros /entrada /salida
```
[cite: 272]

---

## 6. Resumen de Comandos Útiles
* **Listar colas por consola:** `mapred queue -list`[cite: 187, 188].
* **Actualizar configuración sin reiniciar:** `yarn rmadmin -refreshQueues`[cite: 248].
