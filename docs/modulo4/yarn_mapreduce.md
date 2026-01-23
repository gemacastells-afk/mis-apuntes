# YARN y MapReduce

## 1. Evolución: De Hadoop 1.0 a 2.0
Antes de entender cómo funciona hoy, hay que entender el problema del pasado.

### El Problema de Hadoop 1.0 (MR1)
En la versión antigua, existía un único "dictador" llamado **JobTracker**. Este componente tenía que hacerlo todo:
1.  **Gestionar recursos:** Decidir qué ordenadores tenían memoria libre.
2.  **Controlar tareas:** Vigilar que el código no fallara.

Esto creaba un **cuello de botella**. Si el clúster crecía a miles de ordenadores, el JobTracker colapsaba. Además, solo permitía ejecutar trabajos de tipo MapReduce (nada de Spark o streaming).

### La Solución: Hadoop 2.0 (YARN)
La solución fue **dividir responsabilidades**. Nace **YARN** (Yet Another Resource Negotiator), que actúa como el Sistema Operativo del clúster.

* **YARN** solo gestiona el hardware (RAM/CPU).
* Las aplicaciones (como MapReduce o Spark) se ejecutan *encima* de YARN como si fueran programas instalados.

---

## 2. Arquitectura de YARN
YARN funciona con una jerarquía de tres niveles:

### A. Resource Manager (El Dueño de la Empresa)
* **Dónde está:** En el nodo maestro.
* **Misión:** Es el que tiene autoridad máxima de los recursos. Gestiona la RAM y CPU de todo el clúster.
* **Función:** No sabe *qué* estás ejecutando. Solo recibe peticiones ("Necesito 4GB de RAM") y las aprueba o deniega.

### B. Node Manager (El Encargado de Sala)
* **Dónde está:** En cada nodo esclavo (trabajador).
* **Misión:** Vigila su propia máquina.
* **Función:** Informa al Resource Manager de cuánta capacidad libre tiene y se encarga de crear los **Contenedores** (cajas virtuales con recursos) donde se ejecutan las tareas.

### C. Application Master (El Jefe de Proyecto Temporal)
* **Dónde está:** Se crea uno específico para cada trabajo (Job) que lanzas.
* **Misión:** Dirigir la ejecución de *tu* programa.
* **Función:** Negocia los recursos con el Resource Manager y luego da las órdenes a los Node Managers. Cuando el trabajo termina, este jefe "desaparece" [.

---

## 3. Configuración de YARN
Para activar este sistema, debemos editar dos archivos en `/opt/hadoop/etc/hadoop/`.

**1. `mapred-site.xml`**
Aquí le decimos a Hadoop que deje de usar el sistema antiguo y use YARN.
```xml
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
```

**2. `yarn-site.xml`**
Aquí configuramos quién es el jefe (hostname) y activamos el servicio auxiliar para que funcione el Shuffle.

```xml
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>NOMBRE_DE_TU_MAQUINA</value> </property>
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
```
## 4. MapReduce: El Flujo de Trabajo

MapReduce es el paradigma de programación para procesar datos masivos en paralelo. Se divide en tres fases estrictas.

### Fase 1: Mapper (El Clasificador)
* **Entrada:** Recibe datos en bruto (líneas de texto, emails, etc.).
* **Proceso:** Filtra la información útil.
* **Salida:** Emite pares **Clave-Valor**.
* **Importante:** El Mapper **NO suma** ni cuenta totales. Es "tonto" y rápido.Solo dice: "He visto un pepe" (`pepe, 1`).

### Fase 2: Shuffle & Sort (El Organizador)
* **Proceso:** Es automático (caja negra de Hadoop).
* **Acción:** Recoge todas las salidas de los miles de Mappers, las **ordena** alfabéticamente y las **agrupa** por clave.
* **Resultado:** Al Reducer no le llegan datos sueltos, le llega la clave con todos sus valores juntos (Ej: `pepe, [1, 1, 1]`).

### Fase 3: Reducer (El Contador)
* **Entrada:** Recibe los datos ya ordenados y agrupados.
* **Proceso:** Itera sobre la lista de valores y realiza la operación final (sumar, calcular media, máximo...).
* **Salida:** El resultado final que se guarda en HDFS.
