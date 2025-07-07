# Paso 3: Activar el Override

Este es el paso crucial que no se puede olvidar. Para que PrestaShop detecte nuestro nuevo override, debemos forzarlo a reconstruir su índice de clases.

**Acción:**

1. Navega a la carpeta `var/cache/` en la raíz de tu tienda.
2. Borra el archivo `class_index.php`.
3. Refresca cualquier página del Back Office o Front Office. El archivo se regenerará automáticamente, esta vez incluyendo tu override.

