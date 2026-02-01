# Mis Apuntes de Big Data

Bienvenido a mi wiki personal. Aquí iré subiendo todo lo que aprenda sobre Hadoop, Spark y el ecosistema Big Data.

## Estructura del curso

Haz clic en los enlaces para ir a cada tema:

* **Módulo 1:** [Teoría y Fundamentos](modulo1/intro.md)
* **Módulo 2:** [Ecosistema Hadoop](modulo2/core.md)
* **Módulo 3:** [Instalación y Práctica](modulo3/instalacion.md)
* **Módulo 4:** [YARN y MapReduce](modulo4/yarn_mapreduce.md)
* **Módulo 5:** [YARN Scheduler](modulo5/yarn_scheduler.md)

## 📝 Chuleta: Cómo actualizar estos apuntes

Cada vez que se añada contenido nuevo, hay que seguir estos pasos en la terminal para que se vean reflejados en [GitHub Pages](https://gemacastells-afk.github.io/mis-apuntes/):

!!! tip "Pasos para publicar cambios"
    1. **Guardar cambios en local:**
       ```bash
       git add .
       git commit -m "Añadida unidad 5 de YARN Scheduler"
       ```
    2. **Subir al repositorio de código:**
       ```bash
       git push origin main
       ```
    3. **Desplegar en la web (GitHub Pages):**
       ```bash
       mkdocs gh-deploy
       ```

!!! info "Nota sobre el Scheduler"
    Tener en cuenta que los cambios en el menú se configuran siempre en el archivo raíz `mkdocs.yml`. [cite: 13, 190]