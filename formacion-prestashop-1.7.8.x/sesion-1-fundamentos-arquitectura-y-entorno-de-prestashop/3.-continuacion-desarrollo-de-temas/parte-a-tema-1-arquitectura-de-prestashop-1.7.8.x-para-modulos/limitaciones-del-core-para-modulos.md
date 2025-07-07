# Limitaciones del Core para Módulos

**Conceptos Clave / Puntos de Discusión:**

* La principal limitación en 1.7+ es la imposibilidad de hacer `override` de clases que están dentro del nuevo Core basado en Symfony (directorio `/src`). Los overrides solo funcionan para el sistema legacy.
* La convivencia del código legacy y Symfony puede ser compleja.
* A veces, los hooks disponibles pueden no ser suficientes o no tener los parámetros necesarios, obligando a buscar soluciones más creativas (o menos ideales).
