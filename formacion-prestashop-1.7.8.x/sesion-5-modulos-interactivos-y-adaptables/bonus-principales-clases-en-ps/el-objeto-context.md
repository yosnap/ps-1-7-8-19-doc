# ## El objeto "context"

En PrestaShop, `$this->context` es un objeto global que actúa como un "acceso directo" a toda la información importante sobre la situación actual de la tienda y del usuario que la visita.

Imagina que es el salpicadero de un coche: en lugar de tener que abrir el capó para ver el nivel de aceite o la temperatura, tienes todos los indicadores importantes a mano. `$this->context` es ese salpicadero para el desarrollador.

#### ¿Qué contiene el Contexto?

El objeto `$this->context` tiene varias propiedades, que a su vez son otros objetos. Las más importantes que usarás constantemente son:

* `$this->context->shop` 🏬 Contiene toda la información de la tienda que se está visualizando en ese momento. Es fundamental para el multitienda, ya que te permite saber el `id` de la tienda actual, su nombre, el tema que usa, etc.
* `$this->context->customer` 👤 Representa al cliente que está navegando por la tienda. Si el usuario no ha iniciado sesión, muchas de sus propiedades serán nulas. Puedes usarlo para:
  * Saber si un cliente ha iniciado sesión: `$this->context->customer->isLogged()`
  * Obtener su ID, nombre, email, etc.: `$this->context->customer->id`, `$this->context->customer->firstname`
* `$this->context->employee` 👨‍💼 Es el equivalente a `$this->context->customer` pero para el Back Office. Cuando un empleado está gestionando la tienda, este objeto contiene sus datos.
* `$this->context->language` 🌐 Contiene la información del idioma activo en ese momento (su `id`, `iso_code`, etc.). Esencial para módulos multi-idioma.
* `$this->context->cart` 🛒 Representa el carrito de la compra actual del visitante. Puedes usarlo para ver qué productos contiene, obtener su total, etc. Si el visitante no ha añadido nada al carrito, puede ser `null`.
* `$this->context->cookie` 🍪 Es el objeto que gestiona la cookie de sesión de PrestaShop. Aquí puedes guardar y leer datos temporales que persisten mientras el usuario navega (`$this->context->cookie->mi_variable = 1;`).
* `$this->context->link` 🔗 Un objeto ayudante (helper) muy potente para crear URLs amigables y correctas dentro de PrestaShop. Por ejemplo, para obtener el enlace a un controlador de tu módulo: `$this->context->link->getModuleLink(...)`.
* `$this->context->controller` 🖥️ Es una referencia al controlador que está gestionando la página actual. Es útil para acceder a sus métodos, como `$this->context->controller->registerJavascript()`.
* `$this->context->smarty` 🎨 Es el motor de plantillas Smarty, la tecnología que PrestaShop usa para renderizar los archivos `.tpl` y convertirlos en el HTML final. Su función principal es la de actuar como puente entre tu código PHP y la vista. Usas su método `assign()` para pasar variables desde tu controlador a la plantilla.
  * Ejemplo: `$this->context->smarty->assign('nombre_variable', $valor_en_php);`

#### ¿En qué "contextos" te puedes mover?

`$this->context` está disponible en la mayoría de las clases principales de PrestaShop (módulos, controladores, `ObjectModel`s, etc.). El "contexto" en el que te mueves depende principalmente de dónde se está ejecutando tu código:

* Contexto de Front Office: Es el que vive un cliente al visitar la tienda. Aquí, `$this->context->customer` es el objeto principal y `$this->context->employee` es nulo. Tus hooks de display (como `displayHome`) y tus `FrontControllers` se ejecutan siempre en este contexto.
* Contexto de Back Office: Es el que vive un administrador al gestionar la tienda. Aquí, `$this->context->employee` es el objeto principal y `$this->context->customer` suele ser nulo. Tus `AdminControllers` se ejecutan siempre en este contexto.
* Contexto de Tienda (Multitienda): Este es un "sub-contexto" que se aplica a los dos anteriores. Gracias a `$this->context->shop`, tu código puede saber si está operando para la "Tienda de Ropa" o la "Tienda de Zapatos" y actuar en consecuencia.

En resumen, dominar `$this->context` es fundamental porque te permite escribir código que reacciona de forma inteligente a quién está usando la tienda, desde dónde y en qué tienda e idioma, sin tener que buscar esa información por todas partes.
