# Paso 6: Taller Práctico

### Lista de Verificación para Testing

* [ ] ✅ **Estructura creada** correctamente
* [ ] ✅ **Archivos de seguridad** en todos los directorios
* [ ] ✅ **Módulo instalable** sin errores
* [ ] ✅ **Tabla creada** en base de datos
* [ ] ✅ **Hook de autenticación** funcionando
* [ ] ✅ **Mensaje visible** solo para usuarios logueados
* [ ] ✅ **Desinstalación limpia** sin residuos

### Procedimiento de Pruebas

**1. Preparación e Instalación**

```bash
bash# Crear el archivo ZIP
cd /ruta/a/tu/modulo
zip -r welcomelogin.zip welcomelogin/
```

**En PrestaShop:**

1. Ve a `Módulos > Gestor de Módulos`
2. Haz clic en "Subir un módulo"
3. Selecciona `welcomelogin.zip`
4. Confirma que se instala sin errores

**2. Verificación de Funcionalidad**

**Paso A - Estado Sin Login:**

* Visita la página de inicio de tu tienda
* **Resultado esperado**: No debería aparecer ningún mensaje de bienvenida

**Paso B - Primer Login:**

* Inicia sesión con una cuenta de cliente existente
* **Resultado esperado**: El sistema registra el login en la base de datos

**Paso C - Verificación Post-Login:**

* Regresa a la página de inicio
* **Resultado esperado**: Debe aparecer la caja de bienvenida con tu nombre y fecha/hora actual

**Paso D - Segundo Login:**

* Cierra la sesión del cliente
* Espera unos minutos
* Vuelve a iniciar sesión

**Paso E - Verificación de Histórico:**

* Visita nuevamente la página de inicio
* **Resultado esperado**: La caja debe mostrar la fecha del login anterior (del Paso B)

**3. Verificación de Base de Datos**

```sql
sql-- Verificar que la tabla se creó correctamente
SELECT * FROM ps_customer_login_log;

-- Debería mostrar registros con id_customer y last_login
```

**4. Prueba de Desinstalación**

* Desinstala el módulo desde el gestor
* Refresca la página de inicio
* **Resultado esperado**: El mensaje debe desaparecer
* Verifica en la base de datos que la tabla `ps_customer_login_log` fue eliminada



### Ejemplo Práctico (Básico): Modal con Descuento al Iniciar Sesión

**Objetivo del Módulo (`loginmodal`)**

Este módulo es un ejercicio básico diseñado para que los alumnos lo programen desde cero. Su funcionalidad es:

1. Detectar el inicio de sesión: Usando el hook `actionAuthentication`.
2. Activar una bandera: Al detectar el login, pondrá una "bandera" en la sesión del usuario para indicar que se le debe mostrar el modal.
3. Mostrar un modal: En la siguiente página que cargue el usuario, se mostrará una única vez una ventana modal con un código de descuento de bienvenida.
4. Ser autocontenido: El modal se mostrará y funcionará usando su propio CSS y JavaScript, que se cargarán a través de un hook de `display`.

Este ejercicio cubre perfectamente:

* Ciclo de vida: `install()` y `uninstall()` sencillos.
* Hook de acción: `actionAuthentication` para una lógica de fondo simple.
* Hooks de display: `displayHeader` para cargar assets y `displayFooter` para inyectar el HTML del modal.
