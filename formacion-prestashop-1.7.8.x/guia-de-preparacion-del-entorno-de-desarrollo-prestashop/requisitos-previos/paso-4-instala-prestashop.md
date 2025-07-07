# Paso 4: Instala PrestaShop

Si es la primera vez que lanzas el entorno y la instalación automática no está configurada en `docker-compose.yml` (por defecto, `PS_INSTALL_AUTO` no está comentado):

1. **Abre tu navegador web** y ve a la dirección: `http://localhost:8080`
2. Deberías ver el **asistente de instalación de PrestaShop**.
3. Sigue los pasos del instalador:
   * **Idioma:** Selecciona tu idioma.
   * **Acuerdo de licencia:** Acepta.
   * **Información de la tienda:** Puedes poner datos de prueba (Nombre de la tienda, actividad principal, país, tu nombre, email, contraseña para el Back Office). **¡Anota la contraseña del Back Office!**
   * **Configuración del sistema:** Debería detectar que todo está OK.
   * **Configuración de la base de datos:**
     * **Dirección del servidor de la base de datos:** `mysql` (es el nombre del servicio definido en `docker-compose.yml`).
     * **Nombre de la base de datos:** `prestashop`
     * **Login de la base de datos:** `prestashop_user`
     * **Contraseña de la base de datos:** `prestashop_password`
     * **Prefijo de las tablas:** `ps_` (puedes dejar el por defecto).
     * Haz clic en "Probar la conexión con tu base de datos ahora". Debería salir exitoso.
   * La instalación comenzará y tardará unos minutos.

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

* **Configuración de la base de datos:**
  * **Dirección del servidor de la base de datos:** `db` (es el nombre del servicio definido en `docker-compose.yml`).
  * **Nombre de la base de datos:** `prestashop`
  * **Login de la base de datos:** `prestashop`
  * **Contraseña de la base de datos:** `prestashop`
  * **Prefijo de las tablas:** `ps_` (puedes dejar el por defecto).
  * Haz clic en "Probar la conexión con tu base de datos ahora". Debería salir exitoso.

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

* La instalación comenzará y tardará unos minutos.

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

4. **¡Instalación Completada! - Pasos Finales:**

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

*   Una vez finalizada, PrestaShop te indicará que, por razones de seguridad, **debes eliminar la carpeta `install`**. Para hacerlo, ejecuta en tu terminal (desde la raíz de `desarrollo-ps-1.7.8.10-inicial`):

    ```bash
    docker exec prestashop_web rm -rf /var/www/html/install
    ```

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

* PrestaShop también **renombrará automáticamente la carpeta `admin`** a algo como `adminXXX` (donde XXX es una secuencia aleatoria). Te mostrará el nuevo nombre. **¡Anótalo!** Lo necesitarás para acceder al Back Office.

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
