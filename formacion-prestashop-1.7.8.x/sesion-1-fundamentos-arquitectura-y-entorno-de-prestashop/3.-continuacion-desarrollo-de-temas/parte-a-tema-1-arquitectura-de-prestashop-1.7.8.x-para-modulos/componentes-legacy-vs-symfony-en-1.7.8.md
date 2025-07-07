# Componentes Legacy vs Symfony en 1.7.8

* **Conceptos Clave:**
  * **Legacy:** Código más antiguo, a menudo usando patrones como ObjectModels estáticos, acceso global al `Context`, y plantillas Smarty. Muchas partes del BO y FO aún dependen de esto.
  * **Symfony:** Implementación progresiva del framework Symfony para modernizar PrestaShop. Esto incluye el uso de Controladores Symfony, Servicios, Inyección de Dependencias, Doctrine (parcialmente) y plantillas Twig para nuevas secciones del BO (como la página de Producto, Módulos) y partes del FO (como el checkout).
  * La carpeta `/src/PrestaShopBundle` contiene gran parte del código Symfony del Core. Los `Adapter` en `/src/Adapter` sirven de puente entre el mundo legacy y el nuevo.
* **Actividad en Clase / Ejemplo Práctico:**
  * **Navegación Guiada por el Back Office:** Abriremos la página de gestión de "Productos" (basada en Symfony) y la compararemos con la página de "Clientes" (más legacy). Discutiremos las diferencias en estructura de URL, apariencia y comportamiento.
  * Observaremos brevemente el código fuente de un controlador legacy y uno Symfony del Core para ver las diferencias estructurales.

<figure><img src="../../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

#### Tabla Comparativa: Páginas Legacy vs. Páginas Symfony

| Característica      | Páginas "Legacy" (ej. Clientes)                 | Páginas "Symfony" (ej. Productos)                     |
| ------------------- | ----------------------------------------------- | ----------------------------------------------------- |
| Framework           | Framework propio y "casero" de PrestaShop.      | Symfony (Full-Stack).                                 |
| Routing             | Basado en el parámetro controller.              | Sistema de Rutas de Symfony (@Route o YAML).          |
| Controlador Base    | Hereda de AdminController.                      | Hereda de FrameworkBundleAdminController.             |
| Gestión Petición    | Métodos monolíticos (initContent, postProcess). | Acciones específicas (listAction, editAction).        |
| Motor de Plantillas | Smarty (archivos .tpl).                         | Twig (archivos .html.twig).                           |
| Formularios         | HelperForm (rígido y limitado).                 | Componente Symfony Forms (flexible y potente).        |
| Acceso a Datos      | Clases ObjectModel, DbQuery, HelperList.        | Doctrine ORM (Repositorios, Entidades).               |
| Dependencias        | Acceso global con $this->context.               | Inyección de Dependencias en el constructor.          |
| Experiencia Usuario | Recargas de página completas. Más lento.        | Reactivo (AJAX, a veces Vue.js). Más rápido y fluido. |
| Mantenibilidad      | Código más acoplado y difícil de testear.       | Código desacoplado, más fácil de mantener y testear.  |

## PrestaShop: Legacy vs Moderno - Comparación Práctica

### 🏛️ CÓDIGO LEGACY (Forma antigua)

#### Estructura de archivos legacy

```
prestashop/
├── classes/
│   ├── Product.php           ← Clase sin namespace
│   ├── Cart.php              ← Clase sin namespace  
│   ├── Customer.php          ← Clase sin namespace
│   └── Order.php             ← Clase sin namespace
└── modules/
    └── mi_modulo_viejo/
        ├── mi_modulo_viejo.php
        └── classes/
            └── MiClaseVieja.php  ← Sin namespace
```

#### Ejemplo de clase legacy

```php
<?php
// Archivo: classes/Product.php
// ⚠️ SIN NAMESPACE

class Product extends ObjectModel
{
    public $id;
    public $name;
    public $price;
    
    public function __construct($id = null)
    {
        parent::__construct($id);
    }
    
    public function save()
    {
        // Lógica para guardar
        return parent::save();
    }
}
```

#### Cómo se usa el código legacy

```php
<?php
// En cualquier parte de PrestaShop legacy

// 1. Instanciación directa (autoloader de PrestaShop)
$product = new Product(123);
$cart = new Cart(456);
$customer = new Customer(789);

// 2. Acceso a propiedades y métodos
$product->name = 'Mi producto';
$product->price = 29.99;
$product->save();

// 3. Sin imports necesarios
$order = new Order();
$order->id_customer = $customer->id;

// 4. Funciona por el autoloader legacy de PrestaShop
echo "Producto: " . $product->name;
```

***

### 🚀 CÓDIGO MODERNO (Con namespaces)

#### Estructura de archivos moderna

```
mi_modulo/
├── composer.json             ← Configuración de autoloading
├── src/
│   ├── Controller/
│   │   └── ProductController.php
│   ├── Service/
│   │   ├── ProductService.php
│   │   └── PriceCalculator.php
│   ├── Entity/
│   │   └── CustomProduct.php
│   └── Repository/
│       └── ProductRepository.php
├── mi_modulo.php
└── vendor/
    └── autoload.php          ← Generado por Composer
```

#### Configuración composer.json

```json
{
  "name": "mi-empresa/mi-modulo",
  "autoload": {
    "psr-4": {
      "MiEmpresa\\MiModulo\\": "src/"
    }
  },
  "require": {
    "php": ">=7.4"
  }
}
```

#### Ejemplo de clase moderna

```php
<?php
// Archivo: src/Service/ProductService.php

namespace MiEmpresa\MiModulo\Service;

use MiEmpresa\MiModulo\Entity\CustomProduct;
use MiEmpresa\MiModulo\Repository\ProductRepository;
use PrestaShop\PrestaShop\Adapter\Entity\Product; // Clase legacy

class ProductService
{
    private $repository;
    
    public function __construct(ProductRepository $repository)
    {
        $this->repository = $repository;
    }
    
    public function createCustomProduct(string $name, float $price): CustomProduct
    {
        $customProduct = new CustomProduct();
        $customProduct->setName($name);
        $customProduct->setPrice($price);
        
        return $this->repository->save($customProduct);
    }
    
    public function convertLegacyProduct(int $productId): CustomProduct
    {
        // Usando clase legacy dentro de código moderno
        $legacyProduct = new Product($productId);
        
        $customProduct = new CustomProduct();
        $customProduct->setName($legacyProduct->name);
        $customProduct->setPrice($legacyProduct->price);
        
        return $customProduct;
    }
}
```

#### Cómo se usa el código moderno

```php
<?php
// Archivo: mi_modulo.php

// 1. OBLIGATORIO: Incluir autoloader de Composer
require_once __DIR__ . '/vendor/autoload.php';

// 2. Imports explícitos (obligatorios)
use MiEmpresa\MiModulo\Service\ProductService;
use MiEmpresa\MiModulo\Service\PriceCalculator;
use MiEmpresa\MiModulo\Repository\ProductRepository;
use MiEmpresa\MiModulo\Entity\CustomProduct;

class MiModulo extends Module
{
    public function install()
    {
        // 3. Instanciación con namespaces
        $repository = new ProductRepository();
        $service = new ProductService($repository);
        $calculator = new PriceCalculator();
        
        // 4. Uso de métodos
        $product = $service->createCustomProduct('Producto Premium', 99.99);
        $finalPrice = $calculator->calculateWithTax($product->getPrice());
        
        return parent::install();
    }
    
    public function mixingLegacyAndModern()
    {
        // MEZCLANDO: Legacy + Moderno
        
        // Clase legacy (sin namespace)
        $legacyProduct = new Product(123);
        
        // Clases modernas (con namespace)
        $repository = new ProductRepository();
        $service = new ProductService($repository);
        
        // Convertir legacy a moderno
        $modernProduct = $service->convertLegacyProduct($legacyProduct->id);
        
        return $modernProduct;
    }
}
```

***

### 📊 COMPARACIÓN LADO A LADO

| Aspecto          | Legacy                      | Moderno                         |
| ---------------- | --------------------------- | ------------------------------- |
| **Namespaces**   | ❌ No usa                    | ✅ Usa PSR-4                     |
| **Autoloader**   | PrestaShop propio           | Composer + PSR-4                |
| **Imports**      | No necesarios               | `use` obligatorio               |
| **Estructura**   | `classes/NombreClase.php`   | `src/Categoria/NombreClase.php` |
| **Conflictos**   | Posibles (nombres globales) | Imposibles (namespaces)         |
| **Organización** | Carpeta plana               | Estructura jerárquica           |
| **Estándares**   | PrestaShop específico       | PSR-4 universal                 |
| **IDE Support**  | Limitado                    | Excelente                       |

***

### 🔄 EVOLUCIÓN EN LA PRÁCTICA

#### Antes (Legacy)

```php
// Todo en el espacio global
$product = new Product();
$cart = new Cart();
$helper = new Helper(); // ¿Cuál Helper? ¡Conflicto!
```

#### Ahora (Moderno)

```php
use MiModulo\Service\ProductService;
use MiModulo\Helper\PriceHelper;
use OtroModulo\Helper\DateHelper;

$productService = new ProductService();
$priceHelper = new PriceHelper();     // Sin conflictos
$dateHelper = new DateHelper();       // Sin conflictos
```

***

### 🎯 VENTAJAS DEL CÓDIGO MODERNO

#### 1. **Organización Clara**

```php
// Es obvio qué hace cada clase por su namespace
MiModulo\Controller\AdminProductController  → Controlador admin
MiModulo\Service\EmailService              → Servicio de email  
MiModulo\Repository\ProductRepository      → Repositorio de productos
```

#### 2. **Sin Conflictos**

```php
// Puedes tener múltiples clases "Product" sin problemas
use MiModulo\Entity\Product as CustomProduct;
use PrestaShop\PrestaShop\Adapter\Entity\Product as LegacyProduct;
use OtroModulo\Model\Product as ExternalProduct;

$custom = new CustomProduct();
$legacy = new LegacyProduct();
$external = new ExternalProduct();
```

#### 3. **Mejor IDE Support**

```php
use MiModulo\Service\ProductService;

$service = new ProductService();
$service->  // ← El IDE te sugiere todos los métodos disponibles
```

***

### 🚨 ERRORES COMUNES EN LA TRANSICIÓN

#### ❌ Olvidar el autoloader

```php
// Error: No incluir el autoloader
use MiModulo\Service\ProductService;
$service = new ProductService(); // ¡Fatal Error!
```

#### ✅ Correcto

```php
require_once __DIR__ . '/vendor/autoload.php';
use MiModulo\Service\ProductService;
$service = new ProductService(); // ✅ Funciona
```

#### ❌ Mezclar estilos incorrectamente

```php
// Error: Intentar usar namespace en clase legacy
use Product; // ¡Product no tiene namespace!
```

#### ✅ Correcto

```php
// Las clases legacy se usan directamente
$legacyProduct = new Product();

// Las modernas con namespace
use MiModulo\Entity\CustomProduct;
$modernProduct = new CustomProduct();
```

***

### 🎓 CONCLUSIÓN

La evolución de **legacy** → **moderno** representa:

* **Mejor organización** del código
* **Eliminación de conflictos** de nombres
* **Compatibilidad** con estándares universales
* **Mejor experiencia** de desarrollo
* **Código más mantenible** y escalable

**PrestaShop 1.7+** permite usar **ambos estilos**, facilitando la migración gradual.
