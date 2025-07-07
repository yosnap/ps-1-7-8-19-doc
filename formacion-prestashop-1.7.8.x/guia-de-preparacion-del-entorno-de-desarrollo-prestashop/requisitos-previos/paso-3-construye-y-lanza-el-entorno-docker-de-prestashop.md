# Paso 3: Construye y Lanza el Entorno Docker de PrestaShop

**Ahora vamos a usar Docker para crear tu instancia local de PrestaShop 1.7.8.10 con PHP 7.4.**

1. **Asegúrate de estar en el directorio correcto:** En tu terminal, verifica que estás dentro de la carpeta `desarrollo-ps-1.7.8.10-inicial` que clonaste.
2.  **Revisa los archivos Docker (Opcional):**

    * `docker-compose.yml`: Define los servicios (PrestaShop, MySQL, phpMyAdmin).
    * `Dockerfile.prestashop`: Instrucciones para construir la imagen PHP personalizada con las extensiones necesarias y Xdebug.
    * `conf/`: Contiene archivos de configuración para Apache y Xdebug.
    * Docker tiene que estar abierto para ejecutar el siguiente paso


3.  **Construye las imágenes y lanza los contenedores:** Ejecuta el siguiente comando en tu terminal. Este proceso puede tardar varios minutos la primera vez, ya que descargará las imágenes base y construirá la imagen personalizada.

    Para abrir el terminal en VSC: Ctrl+Shitf+>  (Mac y Windows) &#x20;

    ```bash
    docker-compose up -d --build
    ```

    * `-d`: Ejecuta los contenedores en segundo plano (detached mode).
    * `--build`: Fuerza la reconstrucción de la imagen `prestashop` usando `Dockerfile.prestashop`.

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

4. **Verifica que los contenedores estén funcionando:**

Bash

```bash
docker-compose ps
```

Deberías ver tres servicios (`prestashop_mysql`, `prestashop_pma`, `prestashop_web`) con el estado "Up" o "running".

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>
