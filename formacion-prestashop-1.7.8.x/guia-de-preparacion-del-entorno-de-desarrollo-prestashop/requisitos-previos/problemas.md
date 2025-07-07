# ¿Problemas?

* **Docker no se inicia:** Verifica que Docker Desktop esté corriendo. Revisa los logs con `docker-compose logs ps178_web_curso` y `docker-compose logs ps178_mysql_curso`.
* **Error de Xdebug:** Revisa la configuración de `xdebug.ini` (en `conf/`), `launch.json` (en `.vscode/`), y especialmente `pathMappings`. Asegúrate de que el puerto `9003` no esté siendo usado por otra aplicación.
* **Error 500 en PrestaShop:** Activa el modo debug si no lo hiciste (variable `PS_DEV_MODE: 1` en `docker-compose.yml` y luego reconstruye o reinicia el contenedor: `docker-compose restart ps178_web_curso`). Revisa los logs de PHP/Apache dentro del contenedor o en `var/logs/` de PrestaShop.
* **Error en el dominio después de terminar la configuración de Prestashop:** Marcar "No" en "Activar SSL"

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

¡Listo! Si has seguido todos estos pasos, deberías tener un entorno de desarrollo PrestaShop completamente funcional y listo para empezar a crear módulos en nuestra primera clase. Si tienes algún problema, anota los errores e intentaremos resolverlos al inicio de la sesión.

¡Nos vemos en clase!
