---
hidden: true
---

# Paso 6: Verifica la Configuración de Xdebug en VS Code (Recordatorio)

1. Abre la carpeta `desarrollo-ps-1.7.8.10-inicial` en VS Code (si no lo has hecho ya).
2. Asegúrate de que el perfil "PrestaShop 1.7.8.x" está activo.
3. Ve a la pestaña "Ejecutar y Depurar" (icono de play con un insecto en la barra lateral izquierda).
4. En el desplegable de la parte superior, debería aparecer "Listen for Xdebug (PrestaShop Docker)" (viene del archivo `.vscode/launch.json` que está en el repositorio).
5. **Para probarlo:**

* Coloca un breakpoint (punto de interrupción) en la primera línea del archivo `index.php` en la raíz de tu proyecto PrestaShop (VS Code te lo mostrará como si fuera local gracias al mapeo de volúmenes y `pathMappings`).

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

* Haz clic en el icono de "Play" verde al lado de "Listen for Xdebug (PrestaShop Docker)".
* En tu navegador, instala la extensión "Xdebug helper" (para Chrome/Firefox) y actívala en modo "Debug".
* Refresca tu tienda `http://localhost:8080`.
* VS Code debería detenerse en el breakpoint.
