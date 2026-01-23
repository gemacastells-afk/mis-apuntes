# Práctica 2.1: Commit Manual de Metadatos

Esta práctica tiene como objetivo entender cómo HDFS gestiona la persistencia de los datos del NameNode (`fsimage` y `edits`) y cómo forzar un guardado seguro.

## 1. Estado Inicial
Navegamos al directorio de metadatos del NameNode para ver el estado actual.

```bash
cd /datos/namenode/current
ls -l
``` 
**Conceptos Clave:**

- fsimage_...: Es una "foto fija" del sistema de ficheros en un momento concreto.
- edits_...: Registro de cambios (logs) ocurridos después de la última foto.
- VERSION: Contiene identificadores únicos del clúster (ClusterID, BlockPoolID).

## 2. Proceso de Checkpoint Manual

### Paso A: Entrar en Modo Seguro (Safe Mode)
Para guardar un estado consistente, debemos "congelar" el clúster para que nadie escriba datos nuevos.

```bash
hdfs dfsadmin -safemode enter
```

Verificación: Podemos ir a la interfaz web (http://localhost:9870) y veremos que el "Safe mode is ON".

### Paso B: Guardar el Namespace (El "Commit")
Este comando fuerza la unión de la imagen antigua + los cambios recientes (edits) para crear una imagen nueva.

```bash

hdfs dfsadmin -saveNamespace
```

Resultado: Se crea un nuevo archivo fsimage actualizado y se limpian los logs de edits.

### Paso C: Salir del Modo Seguro

Volvemos a abrir el clúster para operaciones normales.

```bash

hdfs dfsadmin -safemode leave
```
## 3. Conclusiones
Al listar de nuevo el directorio /datos/namenode/current, observamos que:

1. Ha aparecido un archivo fsimage con un número de transacción más alto.

2. El archivo edits_inprogress se ha reiniciado.

Esto garantiza que, si el NameNode se apaga ahora mismo, el arranque será mucho más rápido porque ya tiene una "foto" reciente y no tiene que procesar miles de cambios antiguos.