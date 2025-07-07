# Repositorios, Servicios y Controladores

PrestaShop 1.7.8.x presenta una **arquitectura híbrida** que combina componentes legacy con patrones modernos de Symfony. Esta guía explica los tres pilares fundamentales: **Controladores**, **Servicios** y **Repositorios**.\


<figure><img src="../../../../.gitbook/assets/Conceptos clave.png" alt=""><figcaption></figcaption></figure>

### 🎮 Controladores

#### 🔹 Función Principal

Los **Controladores** son los encargados de:

* **Gestionar las peticiones HTTP** del usuario
* **Procesar la lógica de presentación**
* **Devolver respuestas** (HTML, JSON, redirects)
* Actuar como **punto de entrada** para las interacciones

#### 🔹 Tipos de Controladores en PrestaShop

**1. ModuleAdminController (Back Office)**

```php
<?php
// Para el panel de administración
class AdminMiModuloController extends ModuleAdminController
{
    public function __construct()
    {
        $this->bootstrap = true;
        $this->table = 'mi_tabla';
        $this->className = 'MiClase';
        parent::__construct();
    }
    
    public function renderForm()
    {
        // Usa HelperForm y Smarty
        $helper = new HelperForm();
        // ...
    }
}
```

**2. ModuleFrontController (Front Office)**

```php
<?php
// Para la parte pública de la tienda
class MiModuloDisplayModuleFrontController extends ModuleFrontController
{
    public function initContent()
    {
        parent::initContent();
        
        // Lógica para mostrar contenido a clientes
        $this->context->smarty->assign([
            'products' => $this->getProducts(),
        ]);
        
        $this->setTemplate('module:mimodulo/views/templates/front/display.tpl');
    }
}
```

**3. Controladores Symfony (Modernos)**

```php
<?php
// src/Controller/Admin/AdminProductController.php
namespace MiModulo\Controller\Admin;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;

class AdminProductController extends AbstractController
{
    public function index(): Response
    {
        // Arquitectura moderna con inyección de dependencias
        return $this->render('@Modules/mimodulo/views/templates/admin/index.html.twig');
    }
}
```

***

### 🔧 Servicios

#### 🔹 Concepto

Los **Servicios** son clases PHP con un **propósito específico** que:

* **Encapsulan lógica de negocio** compleja
* Tienen **funcionalidades bien definidas**
* Son **reutilizables** en diferentes partes del código
* Se **inyectan** donde son necesarios (no se instancian con `new`)

#### 🔹 Gestión por Symfony

```yaml
# config/services.yml
services:
  mimodulo.product_service:
    class: MiModulo\Service\ProductService
    arguments:
      - '@mimodulo.product_repository'
      - '@doctrine.orm.entity_manager'
```

#### 🔹 Ejemplo de Servicio

```php
<?php
namespace MiModulo\Service;

class ProductService
{
    private $repository;
    private $mailer;
    
    public function __construct(ProductRepository $repository, MailerService $mailer)
    {
        $this->repository = $repository;
        $this->mailer = $mailer;
    }
    
    public function processSpecialOffer(Product $product): bool
    {
        // Lógica de negocio compleja
        $discount = $this->calculateDiscount($product);
        $this->repository->updatePrice($product, $discount);
        $this->mailer->sendOfferNotification($product);
        
        return true;
    }
}
```

#### 🔹 Ejemplos por Tipos de Módulos

| Módulo                   | Uso de Servicios        | Ejemplo                                          |
| ------------------------ | ----------------------- | ------------------------------------------------ |
| **🔍 ps\_facetedsearch** | ✅ **Extensivo**         | Servicios para indexación, filtros, API frontend |
| **💳 ps\_checkout**      | ✅ **Núcleo del módulo** | Crear pago, validar, comunicarse con PayPal      |
| **📝 Blog Avanzado**     | ✅ **Recomendado**       | Gestión de posts, importación, comentarios       |
| **🔗 Conexión ERP**      | ✅ **100% del módulo**   | Conectar API, sincronizar productos/clientes     |
| **🖼️ ps\_imageslider**  | ❌ **Legacy**            | Lógica demasiado simple                          |
| **📧 ps\_emailalerts**   | ❌ **Legacy**            | Lógica en hooks directamente                     |
| **🔔 Popup de Aviso**    | ❌ **No necesario**      | Funcionalidad muy básica                         |

***

### 🗃️ Repositorios

#### 🔹 Patrón de Diseño

Los **Repositorios** abstraen el acceso a datos:

* El código **no interactúa directamente** con la base de datos
* Actúan como **capa intermedia** para operaciones de persistencia
* **Separan** la lógica de negocio de la lógica de datos

#### 🔹 Contexto Symfony/Doctrine

```php
<?php
namespace MiModulo\Repository;

use Doctrine\ORM\EntityRepository;

class ProductRepository extends EntityRepository
{
    public function findActiveProducts(): array
    {
        return $this->createQueryBuilder('p')
            ->where('p.active = :active')
            ->setParameter('active', true)
            ->orderBy('p.position', 'ASC')
            ->getQuery()
            ->getResult();
    }
    
    public function findByCategory(int $categoryId): array
    {
        // Lógica de consulta específica
    }
}
```

#### 🔹 PrestaShop: Legacy vs Moderno

**Legacy (ObjectModel)**

```php
<?php
// Clase que actúa como Active Record
class Product extends ObjectModel
{
    public $name;
    public $price;
    
    // La misma clase maneja persistencia
    public function save() 
    {
        return parent::save(); // Guarda directamente en BD
    }
}

// Uso directo
$product = new Product(123);
$product->name = 'Nuevo nombre';
$product->save(); // Active Record pattern
```

**Moderno (Repository Pattern)**

```php
<?php
// Separación de responsabilidades
class ProductEntity 
{
    private $name;
    private $price;
    // Solo propiedades y getters/setters
}

class ProductRepository 
{
    public function save(ProductEntity $product): void
    {
        // Lógica de persistencia separada
    }
    
    public function findById(int $id): ?ProductEntity
    {
        // Lógica de recuperación separada
    }
}

// Uso con inyección de dependencias
$product = $repository->findById(123);
$product->setName('Nuevo nombre');
$repository->save($product);
```

#### 🔹 Ejemplos por Módulos

| Módulo                   | Acceso a Datos          | Implementación                                  |
| ------------------------ | ----------------------- | ----------------------------------------------- |
| **🔍 ps\_facetedsearch** | **DbQuery + Core**      | Consultas complejas, sin Doctrine               |
| **🖼️ ps\_imageslider**  | **ObjectModel propio**  | Clase `ImageSlider` extends `ObjectModel`       |
| **💳 ps\_checkout**      | **Doctrine + APIs**     | Repositorios para logs, APIs externas           |
| **📧 ps\_emailalerts**   | **Core + Db directo**   | `Product`, `Customer` + `Db::getInstance()`     |
| **📝 Blog Avanzado**     | **✅ Doctrine completo** | Repositorios para `Post`, `Category`, `Comment` |
| **🔗 Conexión ERP**      | **Híbrido**             | ObjectModel Core + lógica propia ERP            |
| **🔔 Popup de Aviso**    | **Configuration**       | Solo `Configuration::get/updateValue`           |

***

### 🏗️ Arquitectura en Acción

#### 🔹 Flujo Típico Moderno

```php
<?php
// 1. CONTROLADOR - Recibe petición
class ProductController extends AbstractController 
{
    public function createProduct(Request $request, ProductService $service): Response
    {
        // 2. Delega la lógica al SERVICIO
        $product = $service->createFromRequest($request);
        
        return $this->json(['id' => $product->getId()]);
    }
}

// 3. SERVICIO - Lógica de negocio
class ProductService 
{
    public function createFromRequest(Request $request): ProductEntity
    {
        $product = new ProductEntity();
        $product->setName($request->get('name'));
        
        // Lógica de negocio
        $this->validateProduct($product);
        $this->applyBusinessRules($product);
        
        // 4. Usa REPOSITORIO para persistir
        return $this->repository->save($product);
    }
}

// 5. REPOSITORIO - Acceso a datos
class ProductRepository 
{
    public function save(ProductEntity $product): ProductEntity
    {
        $this->entityManager->persist($product);
        $this->entityManager->flush();
        
        return $product;
    }
}
```

#### 🔹 Flujo Legacy

```php
<?php
// Todo en el controlador
class AdminProductsController extends AdminController
{
    public function processAdd()
    {
        $product = new Product();
        $product->name = Tools::getValue('name');
        
        // Validación + lógica + persistencia todo junto
        if ($this->validateFields()) {
            $product->save(); // Active Record
            $this->redirect('index.php?controller=AdminProducts');
        }
    }
}
```

***

### 🎯 Ventajas de la Arquitectura Moderna

#### &#x20;**Separación de Responsabilidades**

* **Controladores**: Solo manejan HTTP
* **Servicios**: Solo lógica de negocio
* **Repositorios**: Solo acceso a datos

#### &#x20;**Testabilidad**

```php
// Fácil de testear con mocks
$mockRepository = $this->createMock(ProductRepository::class);
$service = new ProductService($mockRepository);
$result = $service->processProduct($product);
```

#### &#x20;**Reutilización**

```php
// El mismo servicio se puede usar en:
- Controladores web
- Comandos de consola  
- APIs
- Cron jobs
```

#### &#x20;**Mantenibilidad**

* Código más **organizado** y **predecible**
* **Fácil localización** de errores
* **Evolución independiente** de cada capa

***

### 🎓 Conclusión

La arquitectura de **PrestaShop 1.7.8.x** permite:

* **Coexistencia** de código legacy y moderno
* **Migración gradual** hacia patrones actuales
* **Flexibilidad** para elegir el enfoque según la complejidad
* **Preparación** para futuras versiones de PrestaShop

**Regla de oro**: Usa arquitectura moderna para módulos complejos y mantén legacy para funcionalidades simples.
