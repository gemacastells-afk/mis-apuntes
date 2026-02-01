# Unidad 5 — Práctica P5: Partición de Recursos por Colas en YARN (Capacity Scheduler)

## 0. Resumen
En esta práctica vas a **configurar el Capacity Scheduler** para crear una **jerarquía de colas** en YARN, **repartir recursos por porcentajes**, comprobar la estructura en la interfaz del ResourceManager y **resolver un fallo típico** al lanzar trabajos en una cola que no es hoja.

---

## 1. Objetivos
Al terminar, deberías ser capaz de:

- Definir colas principales bajo `root`.
- Crear subcolas (jerarquía) dentro de una cola intermedia.
- Asignar **capacidades** coherentes (que sumen 100% en cada nivel).
- Aplicar cambios con `yarn rmadmin -refreshQueues`.
- Verificar colas y porcentajes en la UI del ResourceManager.
- Entender por qué **solo se puede enviar jobs a colas hoja (leaf queues)**.

---

## 2. Árbol objetivo (estructura de colas)
La jerarquía a implementar es:

- **root**
  - **warehouse**: 20%
  - **datalake**: 20%
  - **olap**: 60%
    - **facts**: 70% (del total de `olap`)
    - **dimensions**: 30% (del total de `olap`)
    ![Árbol de colas YARN](../images/arbolScheduler.png)



> Nota: `facts` y `dimensions` son porcentajes **dentro de `olap`**, no del clúster completo.

---

## 3. Prerrequisitos
- Cluster YARN funcionando (ResourceManager y NodeManagers activos).
- Acceso al directorio de configuración de Hadoop:
  - Normalmente: `$HADOOP_HOME/etc/hadoop`
- Scheduler activo: **Capacity Scheduler**.
  - Comprobación rápida (si hace falta):
    - Revisa `yarn-site.xml` y verifica que el scheduler sea el Capacity Scheduler (propiedad `yarn.resourcemanager.scheduler.class`).

---

## 4. Archivo a editar
El reparto de colas y capacidades se configura en:

- **`capacity-scheduler.xml`**
- Ruta típica: `$HADOOP_HOME/etc/hadoop/capacity-scheduler.xml`

> Consejo: antes de tocar nada, haz una copia:
>
> ```bash
> cp capacity-scheduler.xml capacity-scheduler.xml.bak
> ```

---

## 5. Configuración paso a paso

### PASO 1 — Definir las colas hijas de `root`
Busca (o añade) la propiedad que define las colas bajo `root` y sustituye `default` por las colas objetivo:

```xml
<property>
  <name>yarn.scheduler.capacity.root.queues</name>
  <value>warehouse,datalake,olap</value>
</property>
```

> Importante: si existía `default` y la quitas, asegúrate de **reemplazar también** la configuración de `root.default` por la de tus nuevas colas (para no dejar valores huérfanos).

---

### PASO 2 — Asignar capacidad a `warehouse`, `datalake` y `olap`
Añade (o ajusta) las propiedades de capacidad para cada cola hija de `root`.

```xml
<!-- warehouse: 20% -->
<property>
  <name>yarn.scheduler.capacity.root.warehouse.capacity</name>
  <value>20</value>
</property>

<!-- datalake: 20% -->
<property>
  <name>yarn.scheduler.capacity.root.datalake.capacity</name>
  <value>20</value>
</property>

<!-- olap: 60% -->
<property>
  <name>yarn.scheduler.capacity.root.olap.capacity</name>
  <value>60</value>
</property>
```

✅ Comprobación: en el nivel `root`, las capacidades **deben sumar 100** (20 + 20 + 60 = 100).

---

### PASO 3 — Convertir `olap` en cola padre y crear `facts` y `dimensions`
Para que `olap` tenga subcolas, decláralas así:

```xml
<property>
  <name>yarn.scheduler.capacity.root.olap.queues</name>
  <value>facts,dimensions</value>
</property>
```

Ahora asigna capacidades **dentro de `olap`** (también deben sumar 100):

```xml
<!-- facts: 70% de la capacidad de olap -->
<property>
  <name>yarn.scheduler.capacity.root.olap.facts.capacity</name>
  <value>70</value>
</property>

<!-- dimensions: 30% de la capacidad de olap -->
<property>
  <name>yarn.scheduler.capacity.root.olap.dimensions.capacity</name>
  <value>30</value>
</property>
```

> Resultado: `facts` y `dimensions` son **colas hoja**, y `olap` deja de ser una cola hoja (pasa a ser padre).

---

### PASO 4 — (Opcional recomendado) Estado y máximos de capacidad
Para evitar sorpresas, puedes marcar colas como activas y limitar o ampliar su máximo.

**Estado por defecto:**
```xml
<property>
  <name>yarn.scheduler.capacity.root.olap.facts.state</name>
  <value>RUNNING</value>
</property>
```

**Capacidad máxima (porcentaje):**
```xml
<property>
  <name>yarn.scheduler.capacity.root.olap.facts.maximum-capacity</name>
  <value>100</value>
</property>
```

> `maximum-capacity` controla hasta dónde puede “estirarse” una cola si las demás están vacías (según configuración y demanda).

---

## 6. Aplicar cambios y verificar

### 6.1 Recargar colas (sin reiniciar servicios)
Una vez guardado el archivo:

```bash
yarn rmadmin -refreshQueues
```

Si por alguna razón la UI no refleja cambios, un reinicio del ResourceManager puede ser necesario (depende de cómo esté gestionado el clúster).

---

### 6.2 Verificación en la UI
En la web del ResourceManager (típicamente puerto `8088`):

- Revisa la sección del Scheduler y comprueba:
  - Que existen `warehouse`, `datalake`, `olap`.
  - Que dentro de `olap` existen `facts` y `dimensions`.
  - Que los porcentajes coinciden con lo esperado.

---

## 7. Tabla de capacidades globales (respecto al clúster total)
Aunque `facts` y `dimensions` se configuran dentro de `olap`, su capacidad real sobre el clúster es:

| Cola | Capacidad respecto al clúster | Cálculo |
|---|---:|---|
| `warehouse` | 20% | asignación directa |
| `datalake` | 20% | asignación directa |
| `facts` | 42% | 70% de 60% → 0.70 × 60 = 42 |
| `dimensions` | 18% | 30% de 60% → 0.30 × 60 = 18 |

✅ Comprobación final: 20 + 20 + 42 + 18 = 100.

---

## 8. Ejecución de WordCount y resolución de error

### 8.1 El problema: job enviado a una cola que no es hoja
Si lanzas el trabajo indicando la cola `olap`:

```bash
hadoop jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.4.2.jar wordcount   -Dmapreduce.job.queuename=olap   /input/documento.txt /output/output_olap
```

**Resultado esperado:** el job falla (FAILED).

**Explicación:**
- En Capacity Scheduler, **solo se pueden enviar trabajos a colas hoja**.
- `olap` **no es hoja** porque tiene hijas: `facts` y `dimensions`.
- Por tanto, YARN rechaza el envío.

---

### 8.2 La solución: enviar el job a una cola hoja
Por ejemplo a `facts`:

```bash
hadoop jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.4.2.jar wordcount   -Dmapreduce.job.queuename=facts   /input/documento.txt /output/output_facts
```

**Verificación:**
- En la UI del ResourceManager, el job debería aparecer como **SUCCEEDED**.
- Comprueba también que el consumo de recursos se refleja en la cola correcta.

---

## 9. Cuestionario (respuestas)
1) **¿Qué variable XML gestiona la capacidad máxima de una cola?**  
`yarn.scheduler.capacity.<ruta_cola>.maximum-capacity`  
Ejemplo en `facts`:  
`yarn.scheduler.capacity.root.olap.facts.maximum-capacity`

2) **¿Qué variable XML indica el estado por defecto de una cola?**  
`yarn.scheduler.capacity.<ruta_cola>.state`  
Valores típicos: `RUNNING` (activa) / `STOPPED` (detenida)

3) **¿Qué variable gestiona la vida máxima de un proceso?**  
`yarn.scheduler.capacity.<ruta_cola>.maximum-application-lifetime`  
- Para deshabilitar (sin límite): `-1`  
- Para limitar a 10 minutos: `600` (segundos)

---

## 10. Entrega sugerida
Incluye en tu entrega:

- Captura o evidencia del árbol de colas en la UI.
- Fragmento relevante de `capacity-scheduler.xml` (colas y capacidades).
- Evidencia de:
  - Job fallando al usar `olap` como cola.
  - Job funcionando al usar `facts` o `dimensions`.

---
