# 2. Uso de Tokens y Prevención de CSRF

**Explicación:**&#x20;

Un ataque CSRF (Cross-Site Request Forgery) ocurre cuando un atacante engaña a un usuario autenticado (como un administrador) para que envíe una petición a PrestaShop sin saberlo (por ejemplo, haciendo clic en un enlace malicioso en un email). Para prevenir esto, PrestaShop utiliza tokens: una clave secreta y única para cada sesión de formulario.

Cuando se muestra un formulario, PrestaShop genera un token y lo incluye en un campo oculto. Al procesar el formulario, comprueba que el token recibido es el que esperaba. Si no coinciden, la petición se rechaza.
