# PARTE 2: Desarrollo Avanzado - Módulo de Testimonios PrestaShop 1.7.8.x

### Plan de Formación PrestaShop

***

### 📋 Objetivos de la Clase

Al finalizar esta clase, los participantes serán capaces de:

1. **Desarrollar controladores frontend** completos con validación
2. **Crear templates avanzados** con formularios responsivos
3. **Implementar seguridad CSRF** y validación de datos
4. **Desarrollar sistema de gestión** de testimonios en admin
5. **Añadir CSS y JavaScript** funcional
6. **Realizar testing completo** del módulo

***

### 🎯 Continuación desde la Clase 1

#### Estructura ya creada:

* ✅ Clase principal del módulo
* ✅ Base de datos configurada
* ✅ Hooks básicos implementados
* ✅ Panel de administración inicial

#### Lo que desarrollaremos hoy:

* 🚀 Controladores frontend funcionales
* 🎨 Templates con formularios avanzados
* 🔒 Sistema de seguridad robusto
* ⚙️ Gestión completa de testimonios
* 🎯 Testing y optimización

***

### 💻 PASO 1: Controladores Frontend

#### 1.1 Controlador para Listado - testimonios.php

**Archivo: controllers/front/testimonios.php**

```php
<?php

/**
 * Controlador Frontend - Listado de Testimonios
 * 
 * @extends ModuleFrontController
 */
class GestortestimoniosTestimoniosModuleFrontController extends ModuleFrontController
{
    public function init()
    {
        parent::init();
        
        // Verificar si el módulo está habilitado
        if (!Configuration::get('GESTOR_TESTIMONIOS_ENABLED')) {
            Tools::redirect($this->context->link->getPageLink('index'));
        }
    }

    public function initContent()
    {
        parent::initContent();

        // Obtener página actual para paginación
        $page = (int)Tools::getValue('page', 1);
        $per_page = Configuration::get('GESTOR_TESTIMONIOS_MAX_PER_PAGE');
        
        // Calcular offset para la consulta
        $offset = ($page - 1) * $per_page;

        // Obtener testimonios aprobados
        $testimonios = $this->getTestimonios($per_page, $offset);
        $total_testimonios = $this->getTotalTestimonios();
        $total_pages = ceil($total_testimonios / $per_page);
        
        // Generar enlaces de paginación
        $pagination_links = $this->generatePaginationLinks($page, $total_pages);
        
        // Obtener estadísticas para la página
        $estadisticas = $this->module->getTestimoniosStats();

        // Asignar variables a Smarty
        $this->context->smarty->assign([
            'testimonios' => $testimonios,
            'total_testimonios' => $total_testimonios,
            'estadisticas' => $estadisticas,
            'current_page' => $page,
            'total_pages' => $total_pages,
            'per_page' => $per_page,
            'pagination_links' => $pagination_links,
            'enviar_testimonio_link' => $this->context->link->getModuleLink(
                'gestortestimonios', 
                'enviar'
            )
        ]);

        $this->setTemplate('module:gestortestimonios/views/templates/front/testimonios.tpl');
    }

    /**
     * Obtener testimonios paginados
     */
    private function getTestimonios($limit, $offset)
    {
        $sql = 'SELECT t.*, c.firstname, c.lastname
                FROM `' . _DB_PREFIX_ . 'testimonios` t
                LEFT JOIN `' . _DB_PREFIX_ . 'customer` c ON t.customer_id = c.id_customer
                WHERE t.status = "approved"
                ORDER BY t.created_at DESC
                LIMIT ' . (int)$limit . ' OFFSET ' . (int)$offset;

        return Db::getInstance()->executeS($sql);
    }

    /**
     * Contar total de testimonios aprobados
     */
    private function getTotalTestimonios()
    {
        $sql = 'SELECT COUNT(*) as total
                FROM `' . _DB_PREFIX_ . 'testimonios`
                WHERE status = "approved"';

        $result = Db::getInstance()->getRow($sql);
        return (int)$result['total'];
    }

    /**
     * Generar enlaces de paginación
     */
    private function generatePaginationLinks($current_page, $total_pages)
    {
        $links = [];
        $base_url = $this->context->link->getModuleLink('gestortestimonios', 'testimonios');

        // Enlace a primera página
        if ($current_page > 1) {
            $links['first'] = $base_url . '?page=1';
            $links['previous'] = $base_url . '?page=' . ($current_page - 1);
        }

        // Páginas alrededor de la actual
        for ($i = max(1, $current_page - 2); $i <= min($total_pages, $current_page + 2); $i++) {
            $links['pages'][$i] = [
                'url' => $base_url . '?page=' . $i,
                'current' => ($i == $current_page)
            ];
        }

        // Enlace a última página
        if ($current_page < $total_pages) {
            $links['next'] = $base_url . '?page=' . ($current_page + 1);
            $links['last'] = $base_url . '?page=' . $total_pages;
        }

        return $links;
    }

    public function setMedia()
    {
        parent::setMedia();
        
        $this->context->controller->addCSS($this->module->getPathUri() . 'views/css/gestortestimonios.css');
        $this->context->controller->addJS($this->module->getPathUri() . 'views/js/gestortestimonios.js');
    }

    public function getBreadcrumbLinks()
    {
        $breadcrumb = parent::getBreadcrumbLinks();

        $breadcrumb['links'][] = [
            'title' => $this->trans('Testimonios de Clientes', [], 'Modules.Gestortestimonios.Front'),
            'url' => $this->context->link->getModuleLink('gestortestimonios', 'testimonios')
        ];

        return $breadcrumb;
    }
}
```

#### 1.2 Controlador para Formulario - enviar.php

**Archivo: controllers/front/enviar.php**

```php
<?php

/**
 * Controlador Frontend - Envío de Testimonios
 * 
 * @extends ModuleFrontController
 */
class GestortestimoniosEnviarModuleFrontController extends ModuleFrontController
{
    public function init()
    {
        parent::init();
        
        // Verificar si el módulo está habilitado
        if (!Configuration::get('GESTOR_TESTIMONIOS_ENABLED')) {
            Tools::redirect($this->context->link->getPageLink('index'));
        }
    }

    public function initContent()
    {
        parent::initContent();

        $success = false;
        $error = '';
        $form_data = [];

        // Procesar formulario si se ha enviado
        if (Tools::isSubmit('submitTestimonio')) {
            $result = $this->processTestimonioForm();
            $success = $result['success'];
            $error = $result['error'];
            $form_data = $result['form_data'];
        }

        // Obtener información del cliente si está logueado
        $customer_info = [];
        if ($this->context->customer->isLogged()) {
            $customer_info = [
                'id' => $this->context->customer->id,
                'firstname' => $this->context->customer->firstname,
                'lastname' => $this->context->customer->lastname,
                'email' => $this->context->customer->email
            ];
        }

        // Generar token CSRF
        $csrf_token = $this->module->generateCSRFToken();

        // Asignar variables a Smarty
        $this->context->smarty->assign([
            'success' => $success,
            'error' => $error,
            'form_data' => $form_data,
            'customer_info' => $customer_info,
            'csrf_token' => $csrf_token,
            'min_rating' => Configuration::get('GESTOR_TESTIMONIOS_MIN_RATING'),
            'max_rating' => Configuration::get('GESTOR_TESTIMONIOS_MAX_RATING'),
            'testimonios_link' => $this->context->link->getModuleLink(
                'gestortestimonios', 
                'testimonios'
            )
        ]);

        $this->setTemplate('module:gestortestimonios/views/templates/front/enviar.tpl');
    }

    /**
     * Procesar formulario de testimonio
     */
    private function processTestimonioForm()
    {
        $success = false;
        $error = '';
        $form_data = [];

        // Obtener datos del formulario
        $client_name = trim(Tools::getValue('client_name'));
        $company = trim(Tools::getValue('company'));
        $email = trim(Tools::getValue('email'));
        $testimonial_text = trim(Tools::getValue('testimonial_text'));
        $rating = (int)Tools::getValue('rating');
        $privacy_accept = Tools::getValue('privacy_accept');
        $csrf_token = Tools::getValue('csrf_token');

        // Guardar datos para repoblar formulario en caso de error
        $form_data = [
            'client_name' => $client_name,
            'company' => $company,
            'email' => $email,
            'testimonial_text' => $testimonial_text,
            'rating' => $rating
        ];

        // Validar token CSRF
        if (!$this->module->validateCSRFToken($csrf_token)) {
            $error = $this->trans('Token de seguridad inválido. Por favor, recarga la página e inténtalo de nuevo.', [], 'Modules.Gestortestimonios.Front');
            return ['success' => false, 'error' => $error, 'form_data' => $form_data];
        }

        // Validaciones de campos obligatorios
        if (empty($client_name)) {
            $error = $this->trans('El nombre es obligatorio.', [], 'Modules.Gestortestimonios.Front');
        } elseif (strlen($client_name) > 255) {
            $error = $this->trans('El nombre no puede superar los 255 caracteres.', [], 'Modules.Gestortestimonios.Front');
        } elseif (empty($testimonial_text)) {
            $error = $this->trans('El testimonio es obligatorio.', [], 'Modules.Gestortestimonios.Front');
        } elseif (strlen($testimonial_text) < 10) {
            $error = $this->trans('El testimonio debe tener al menos 10 caracteres.', [], 'Modules.Gestortestimonios.Front');
        } elseif (strlen($testimonial_text) > 2000) {
            $error = $this->trans('El testimonio no puede superar los 2000 caracteres.', [], 'Modules.Gestortestimonios.Front');
        } elseif (!empty($email) && !Validate::isEmail($email)) {
            $error = $this->trans('El formato del email no es válido.', [], 'Modules.Gestortestimonios.Front');
        } elseif (!empty($company) && strlen($company) > 255) {
            $error = $this->trans('El nombre de la empresa no puede superar los 255 caracteres.', [], 'Modules.Gestortestimonios.Front');
        } elseif ($rating > 0 && ($rating < Configuration::get('GESTOR_TESTIMONIOS_MIN_RATING') || $rating > Configuration::get('GESTOR_TESTIMONIOS_MAX_RATING'))) {
            $error = $this->trans('La valoración debe estar entre %min% y %max% estrellas.', 
                ['%min%' => Configuration::get('GESTOR_TESTIMONIOS_MIN_RATING'), '%max%' => Configuration::get('GESTOR_TESTIMONIOS_MAX_RATING')], 
                'Modules.Gestortestimonios.Front');
        } elseif (!$privacy_accept) {
            $error = $this->trans('Debes aceptar los términos de privacidad para continuar.', [], 'Modules.Gestortestimonios.Front');
        }

        // Si hay errores, devolver
        if (!empty($error)) {
            return ['success' => false, 'error' => $error, 'form_data' => $form_data];
        }

        // Preparar datos para guardar
        $data = [
            'client_name' => $client_name,
            'company' => !empty($company) ? $company : null,
            'email' => !empty($email) ? $email : null,
            'testimonial_text' => $testimonial_text,
            'rating' => $rating > 0 ? $rating : null,
            'customer_id' => $this->context->customer->isLogged() ? $this->context->customer->id : null
        ];

        // Guardar testimonio
        if ($this->module->saveTestimonio($data)) {
            $success = true;
            $form_data = []; // Limpiar formulario en caso de éxito
        } else {
            $error = $this->trans('Error al guardar el testimonio. Por favor, inténtalo de nuevo.', [], 'Modules.Gestortestimonios.Front');
        }

        return ['success' => $success, 'error' => $error, 'form_data' => $form_data];
    }

    /**
     * Manejar peticiones AJAX
     */
    public function displayAjax()
    {
        $action = Tools::getValue('action');
        
        if ($action === 'validateEmail') {
            $email = Tools::getValue('email');
            $isValid = !empty($email) && Validate::isEmail($email);
            
            die(json_encode(['valid' => $isValid]));
        }
        
        die(json_encode(['error' => 'Acción no válida']));
    }

    public function setMedia()
    {
        parent::setMedia();
        
        $this->context->controller->addCSS($this->module->getPathUri() . 'views/css/gestortestimonios.css');
        $this->context->controller->addJS($this->module->getPathUri() . 'views/js/gestortestimonios.js');
    }

    public function getBreadcrumbLinks()
    {
        $breadcrumb = parent::getBreadcrumbLinks();

        $breadcrumb['links'][] = [
            'title' => $this->trans('Testimonios de Clientes', [], 'Modules.Gestortestimonios.Front'),
            'url' => $this->context->link->getModuleLink('gestortestimonios', 'testimonios')
        ];

        $breadcrumb['links'][] = [
            'title' => $this->trans('Enviar Testimonio', [], 'Modules.Gestortestimonios.Front'),
            'url' => $this->context->link->getModuleLink('gestortestimonios', 'enviar')
        ];

        return $breadcrumb;
    }
}
```

***

### 🎨 PASO 2: Templates Frontend Avanzados

#### 2.1 Template Listado - testimonios.tpl

**Archivo: views/templates/front/testimonios.tpl**

```smarty
{extends file='page.tpl'}

{block name='page_title'}
    {l s='Testimonios de Nuestros Clientes' mod='gestortestimonios'}
{/block}

{block name='page_content'}
<div class="testimonios-container">
    
    {* Estadísticas *}
    <div class="testimonios-stats mb-4">
        <div class="row">
            <div class="col-md-3 col-6">
                <div class="stat-card">
                    <i class="material-icons">rate_review</i>
                    <div class="stat-number">{$estadisticas.approved}</div>
                    <div class="stat-label">{l s='Testimonios' mod='gestortestimonios'}</div>
                </div>
            </div>
            <div class="col-md-3 col-6">
                <div class="stat-card">
                    <i class="material-icons">star</i>
                    <div class="stat-number">{$estadisticas.avg_rating|string_format:"%.1f"}</div>
                    <div class="stat-label">{l s='Valoración Media' mod='gestortestimonios'}</div>
                </div>
            </div>
            <div class="col-md-6">
                <div class="testimonios-actions">
                    <a href="{$enviar_testimonio_link}" class="btn btn-primary">
                        <i class="material-icons">add</i>
                        {l s='Enviar tu Testimonio' mod='gestortestimonios'}
                    </a>
                </div>
            </div>
        </div>
    </div>

    {* Lista de Testimonios *}
    {if $testimonios && count($testimonios) > 0}
        <div class="testimonios-grid">
            {foreach from=$testimonios item=testimonio name=testimonios}
                <article class="testimonio-card" itemscope itemtype="https://schema.org/Review">
                    <div class="testimonio-header">
                        <div class="cliente-info">
                            <h3 class="cliente-nombre" itemprop="author" itemscope itemtype="https://schema.org/Person">
                                <span itemprop="name">
                                    {if $testimonio.firstname && $testimonio.lastname}
                                        {$testimonio.firstname|escape:'html'} {$testimonio.lastname|escape:'html'}
                                    {else}
                                        {$testimonio.client_name|escape:'html'}
                                    {/if}
                                </span>
                            </h3>
                            {if $testimonio.company}
                                <div class="cliente-empresa">
                                    <i class="material-icons">business</i>
                                    {$testimonio.company|escape:'html'}
                                </div>
                            {/if}
                        </div>

                        {if $testimonio.rating}
                            <div class="testimonio-rating" itemprop="reviewRating" itemscope itemtype="https://schema.org/Rating">
                                <meta itemprop="ratingValue" content="{$testimonio.rating}">
                                <meta itemprop="bestRating" content="5">
                                <div class="stars">
                                    {for $i=1 to 5}
                                        <i class="material-icons star {if $i <= $testimonio.rating}filled{/if}">star</i>
                                    {/for}
                                </div>
                            </div>
                        {/if}
                    </div>

                    <div class="testimonio-content">
                        <blockquote itemprop="reviewBody">
                            "{$testimonio.testimonial_text|escape:'html'|nl2br}"
                        </blockquote>
                    </div>

                    <div class="testimonio-footer">
                        <time class="testimonio-fecha" itemprop="datePublished" datetime="{$testimonio.created_at|date_format:'c'}">
                            {$testimonio.created_at|date_format:'%d/%m/%Y'}
                        </time>
                    </div>
                </article>
            {/foreach}
        </div>

        {* Paginación *}
        {if $total_pages > 1}
            <nav class="testimonios-pagination" aria-label="{l s='Navegación de testimonios' mod='gestortestimonios'}">
                <ul class="pagination justify-content-center">
                    {if isset($pagination_links.first)}
                        <li class="page-item">
                            <a class="page-link" href="{$pagination_links.first}">
                                <i class="material-icons">first_page</i>
                            </a>
                        </li>
                    {/if}
                    
                    {if isset($pagination_links.previous)}
                        <li class="page-item">
                            <a class="page-link" href="{$pagination_links.previous}">
                                <i class="material-icons">chevron_left</i>
                            </a>
                        </li>
                    {/if}

                    {if isset($pagination_links.pages)}
                        {foreach from=$pagination_links.pages key=page_num item=page_data}
                            <li class="page-item {if $page_data.current}active{/if}">
                                <a class="page-link" href="{$page_data.url}">{$page_num}</a>
                            </li>
                        {/foreach}
                    {/if}

                    {if isset($pagination_links.next)}
                        <li class="page-item">
                            <a class="page-link" href="{$pagination_links.next}">
                                <i class="material-icons">chevron_right</i>
                            </a>
                        </li>
                    {/if}
                    
                    {if isset($pagination_links.last)}
                        <li class="page-item">
                            <a class="page-link" href="{$pagination_links.last}">
                                <i class="material-icons">last_page</i>
                            </a>
                        </li>
                    {/if}
                </ul>
            </nav>
        {/if}

    {else}
        <div class="testimonios-empty">
            <div class="empty-state">
                <i class="material-icons">sentiment_satisfied</i>
                <h3>{l s='Aún no hay testimonios' mod='gestortestimonios'}</h3>
                <p>{l s='Sé el primero en compartir tu experiencia con nosotros.' mod='gestortestimonios'}</p>
                <a href="{$enviar_testimonio_link}" class="btn btn-primary">
                    {l s='Enviar tu Testimonio' mod='gestortestimonios'}
                </a>
            </div>
        </div>
    {/if}

</div>
{/block}
```

#### 2.2 Template Formulario - enviar.tpl

{% hint style="info" %}
Este template incluye validación avanzada, contador de caracteres y sistema de rating interactivo.
{% endhint %}

**Archivo: views/templates/front/enviar.tpl**

```smarty
{extends file='page.tpl'}

{block name='page_title'}
    {l s='Enviar Testimonio' mod='gestortestimonios'}
{/block}

{block name='page_content'}
<div class="enviar-testimonio-container">

    {* Mensajes de estado *}
    {if $success}
        <div class="alert alert-success" role="alert">
            <i class="material-icons">check_circle</i>
            <strong>{l s='¡Gracias por tu testimonio!' mod='gestortestimonios'}</strong>
            {if Configuration::get('GESTOR_TESTIMONIOS_AUTO_APPROVE')}
                {l s='Tu testimonio ha sido publicado y aparecerá en nuestra página.' mod='gestortestimonios'}
            {else}
                {l s='Tu testimonio será revisado y publicado en breve.' mod='gestortestimonios'}
            {/if}
            <div class="mt-3">
                <a href="{$testimonios_link}" class="btn btn-outline-primary">
                    {l s='Ver Testimonios' mod='gestortestimonios'}
                </a>
            </div>
        </div>
    {/if}

    {if $error}
        <div class="alert alert-danger" role="alert">
            <i class="material-icons">error</i>
            <strong>{l s='Error:' mod='gestortestimonios'}</strong> {$error}
        </div>
    {/if}

    {if !$success}
        <div class="testimonio-form-wrapper">
            <div class="form-intro">
                <h2>{l s='Comparte tu Experiencia' mod='gestortestimonios'}</h2>
                <p class="lead">{l s='Tu opinión es muy valiosa para nosotros y para otros clientes. Compártenos tu experiencia.' mod='gestortestimonios'}</p>
            </div>

            <form id="testimonioForm" method="post" action="" class="testimonio-form">
                <input type="hidden" name="csrf_token" value="{$csrf_token}">
                
                {if $customer_info.id}
                    <input type="hidden" name="customer_id" value="{$customer_info.id}">
                {/if}

                <div class="row">
                    <div class="col-md-6">
                        <div class="form-group">
                            <label for="client_name" class="form-label required">
                                <i class="material-icons">person</i>
                                {l s='Tu Nombre' mod='gestortestimonios'} <span class="required-asterisk">*</span>
                            </label>
                            <input type="text" 
                                   id="client_name" 
                                   name="client_name" 
                                   class="form-control"
                                   value="{if $form_data.client_name}{$form_data.client_name|escape:'html'}{elseif $customer_info.firstname}{$customer_info.firstname|escape:'html'} {$customer_info.lastname|escape:'html'}{/if}"
                                   maxlength="255"
                                   required>
                            <div class="form-text">{l s='Nombre que aparecerá con tu testimonio' mod='gestortestimonios'}</div>
                        </div>
                    </div>

                    <div class="col-md-6">
                        <div class="form-group">
                            <label for="email" class="form-label">
                                <i class="material-icons">email</i>
                                {l s='Email' mod='gestortestimonios'}
                            </label>
                            <input type="email" 
                                   id="email" 
                                   name="email" 
                                   class="form-control"
                                   value="{if $form_data.email}{$form_data.email|escape:'html'}{elseif $customer_info.email}{$customer_info.email|escape:'html'}{/if}"
                                   maxlength="255">
                            <div class="form-text">{l s='Opcional - No se mostrará públicamente' mod='gestortestimonios'}</div>
                            <div id="email-validation" class="invalid-feedback"></div>
                        </div>
                    </div>
                </div>

                <div class="form-group">
                    <label for="company" class="form-label">
                        <i class="material-icons">business</i>
                        {l s='Empresa u Organización' mod='gestortestimonios'}
                    </label>
                    <input type="text" 
                           id="company" 
                           name="company" 
                           class="form-control"
                           value="{if $form_data.company}{$form_data.company|escape:'html'}{/if}"
                           maxlength="255">
                    <div class="form-text">{l s='Opcional - Aparecerá junto a tu nombre' mod='gestortestimonios'}</div>
                </div>

                <div class="form-group">
                    <label for="testimonial_text" class="form-label required">
                        <i class="material-icons">rate_review</i>
                        {l s='Tu Testimonio' mod='gestortestimonios'} <span class="required-asterisk">*</span>
                    </label>
                    <textarea id="testimonial_text" 
                              name="testimonial_text" 
                              class="form-control"
                              rows="6"
                              maxlength="2000"
                              required
                              placeholder="{l s='Cuéntanos sobre tu experiencia con nosotros...' mod='gestortestimonios'}">{if $form_data.testimonial_text}{$form_data.testimonial_text|escape:'html'}{/if}</textarea>
                    <div class="form-text">
                        <span class="char-counter">
                            <span id="char-count">0</span>/2000 {l s='caracteres' mod='gestortestimonios'}
                        </span>
                        <span class="min-chars">{l s='Mínimo 10 caracteres' mod='gestortestimonios'}</span>
                    </div>
                </div>

                <div class="form-group">
                    <label class="form-label">
                        <i class="material-icons">star_rate</i>
                        {l s='Valoración' mod='gestortestimonios'}
                    </label>
                    <div class="rating-input">
                        <div class="stars-wrapper">
                            {for $i=$min_rating to $max_rating}
                                <input type="radio" 
                                       id="rating_{$i}" 
                                       name="rating" 
                                       value="{$i}"
                                       {if $form_data.rating == $i}checked{/if}>
                                <label for="rating_{$i}" class="star-label">
                                    <i class="material-icons star">star</i>
                                </label>
                            {/for}
                        </div>
                        <div class="rating-text">{l s='Opcional - Selecciona tu valoración' mod='gestortestimonios'}</div>
                    </div>
                </div>

                <div class="form-group">
                    <div class="form-check">
                        <input type="checkbox" 
                               id="privacy_accept" 
                               name="privacy_accept" 
                               class="form-check-input"
                               value="1"
                               required>
                        <label for="privacy_accept" class="form-check-label required">
                            {l s='Acepto que mi testimonio pueda ser publicado en el sitio web' mod='gestortestimonios'}
                            <span class="required-asterisk">*</span>
                        </label>
                    </div>
                    <div class="form-text">
                        {l s='Tu información personal no será compartida. Solo se mostrará tu nombre y empresa.' mod='gestortestimonios'}
                    </div>
                </div>

                <div class="form-actions">
                    <button type="submit" name="submitTestimonio" class="btn btn-primary btn-lg">
                        <i class="material-icons">send</i>
                        {l s='Enviar Testimonio' mod='gestortestimonios'}
                    </button>
                    <a href="{$testimonios_link}" class="btn btn-outline-secondary">
                        <i class="material-icons">arrow_back</i>
                        {l s='Ver Testimonios' mod='gestortestimonios'}
                    </a>
                </div>
            </form>
        </div>
    {/if}

</div>

{* JavaScript para funcionalidad del formulario *}
<script>
document.addEventListener('DOMContentLoaded', function() {
    // Contador de caracteres
    const textarea = document.getElementById('testimonial_text');
    const charCount = document.getElementById('char-count');
    
    if (textarea && charCount) {
        function updateCharCount() {
            charCount.textContent = textarea.value.length;
        }
        
        textarea.addEventListener('input', updateCharCount);
        updateCharCount(); // Inicializar contador
    }
    
    // Sistema de rating interactivo
    const ratingInputs = document.querySelectorAll('input[name="rating"]');
    const starLabels = document.querySelectorAll('.star-label');
    
    starLabels.forEach((label, index) => {
        label.addEventListener('mouseenter', function() {
            highlightStars(index + 1);
        });
        
        label.addEventListener('mouseleave', function() {
            const checkedRating = document.querySelector('input[name="rating"]:checked');
            if (checkedRating) {
                highlightStars(parseInt(checkedRating.value));
            } else {
                highlightStars(0);
            }
        });
        
        label.addEventListener('click', function() {
            highlightStars(index + 1);
        });
    });
    
    function highlightStars(rating) {
        starLabels.forEach((label, index) => {
            const star = label.querySelector('.star');
            if (index < rating) {
                star.classList.add('active');
            } else {
                star.classList.remove('active');
            }
        });
    }
    
    // Inicializar estrellas si hay rating seleccionado
    const checkedRating = document.querySelector('input[name="rating"]:checked');
    if (checkedRating) {
        highlightStars(parseInt(checkedRating.value));
    }
    
    // Validación de email en tiempo real
    const emailInput = document.getElementById('email');
    const emailValidation = document.getElementById('email-validation');
    
    if (emailInput) {
        emailInput.addEventListener('blur', function() {
            if (this.value.trim() !== '') {
                validateEmail(this.value);
            } else {
                emailValidation.textContent = '';
                this.classList.remove('is-invalid', 'is-valid');
            }
        });
    }
    
    function validateEmail(email) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (emailRegex.test(email)) {
            emailInput.classList.remove('is-invalid');
            emailInput.classList.add('is-valid');
            emailValidation.textContent = '';
        } else {
            emailInput.classList.remove('is-valid');
            emailInput.classList.add('is-invalid');
            emailValidation.textContent = '{l s="Formato de email inválido" mod="gestortestimonios"}';
        }
    }
});
</script>
{/block}
```

***

### 🔒 PASO 3: Sistema de Seguridad y Validación

#### 3.1 Añadir métodos de seguridad al módulo principal

**Añadir a gestortestimonios.php:**

```php
/**
 * Generar token CSRF usando sistema de PrestaShop
 */
public function generateCSRFToken() {
    return Tools::getToken(false);
}

/**
 * Validar token CSRF usando sistema de PrestaShop
 */
public function validateCSRFToken($token) {
    return Tools::getToken(false) === $token;
}

/**
 * Guardar nuevo testimonio con validación completa
 */
public function saveTestimonio($data) {
    // Validación de datos
    if (empty($data['client_name']) || empty($data['testimonial_text'])) {
        return false;
    }

    // Sanear datos de entrada
    $client_name = pSQL($data['client_name']);
    $company = !empty($data['company']) ? pSQL($data['company']) : null;
    $testimonial_text = pSQL($data['testimonial_text']);
    $rating = !empty($data['rating']) ? (int)$data['rating'] : null;
    $email = !empty($data['email']) ? pSQL($data['email']) : null;
    $customer_id = !empty($data['customer_id']) ? (int)$data['customer_id'] : null;

    // Estado inicial según configuración
    $status = Configuration::get('GESTOR_TESTIMONIOS_AUTO_APPROVE') ? 'approved' : 'pending';

    $sql = 'INSERT INTO `' . _DB_PREFIX_ . 'testimonios`
            (client_name, company, testimonial_text, rating, status, email, customer_id, created_at)
            VALUES ("' . $client_name . '", ' .
        ($company ? '"' . $company . '"' : 'NULL') . ', "' .
        $testimonial_text . '", ' .
        ($rating ? $rating : 'NULL') . ', "' .
        $status . '", ' .
        ($email ? '"' . $email . '"' : 'NULL') . ', ' .
        ($customer_id ? $customer_id : 'NULL') . ', NOW())';

    $result = Db::getInstance()->execute($sql);

    if (!$result) {
        $error = Db::getInstance()->getMsgError();
        PrestaShopLogger::addLog(
            'GestorTestimonios: Error saving testimonial - ' . $error,
            3,
            null,
            'GestorTestimonios'
        );
    }

    return $result;
}
```

***

### ⚙️ PASO 4: Gestión Avanzada de Testimonios en Admin

#### 4.1 Ampliar métodos de gestión en gestortestimonios.php

```php
/**
 * Página de configuración del módulo - Versión completa
 */
public function getContent() {
    $output = '';

    // Procesar acciones de testimonios
    if (Tools::isSubmit('approveTestimonio')) {
        $id = Tools::getValue('id_testimonio');
        $output .= $this->approveTestimonio($id);
    }

    if (Tools::isSubmit('rejectTestimonio')) {
        $id = Tools::getValue('id_testimonio');
        $output .= $this->rejectTestimonio($id);
    }

    if (Tools::isSubmit('deleteTestimonio')) {
        $id = Tools::getValue('id_testimonio');
        $output .= $this->deleteTestimonio($id);
    }

    // Procesar formulario de configuración
    if (Tools::isSubmit('submit' . $this->name)) {
        $output .= $this->processConfiguration();
    }

    return $output . $this->displayConfigurationForm();
}

/**
 * Aprobar testimonio
 */
public function approveTestimonio($id) {
    $id = (int)$id;
    if ($id <= 0) {
        return $this->displayError($this->l('ID de testimonio inválido.'));
    }

    $sql = 'UPDATE `' . _DB_PREFIX_ . 'testimonios`
            SET status = "approved"
            WHERE id = ' . $id;

    if (Db::getInstance()->execute($sql)) {
        return $this->displayConfirmation($this->l('Testimonio aprobado exitosamente.'));
    } else {
        return $this->displayError($this->l('Error al aprobar el testimonio.'));
    }
}

/**
 * Rechazar testimonio
 */
public function rejectTestimonio($id) {
    $id = (int)$id;
    if ($id <= 0) {
        return $this->displayError($this->l('ID de testimonio inválido.'));
    }

    $sql = 'UPDATE `' . _DB_PREFIX_ . 'testimonios`
            SET status = "rejected"
            WHERE id = ' . $id;

    if (Db::getInstance()->execute($sql)) {
        return $this->displayConfirmation($this->l('Testimonio rechazado exitosamente.'));
    } else {
        return $this->displayError($this->l('Error al rechazar el testimonio.'));
    }
}

/**
 * Eliminar testimonio
 */
public function deleteTestimonio($id) {
    $id = (int)$id;
    if ($id <= 0) {
        return $this->displayError($this->l('ID de testimonio inválido.'));
    }

    $sql = 'DELETE FROM `' . _DB_PREFIX_ . 'testimonios` WHERE id = ' . $id;

    if (Db::getInstance()->execute($sql)) {
        return $this->displayConfirmation($this->l('Testimonio eliminado exitosamente.'));
    } else {
        return $this->displayError($this->l('Error al eliminar el testimonio.'));
    }
}

/**
 * Obtener testimonios pendientes de aprobación
 */
public function getPendingTestimonios() {
    $sql = 'SELECT t.*, c.firstname, c.lastname
            FROM `' . _DB_PREFIX_ . 'testimonios` t
            LEFT JOIN `' . _DB_PREFIX_ . 'customer` c ON t.customer_id = c.id_customer
            WHERE t.status = "pending"
            ORDER BY t.created_at DESC';

    return Db::getInstance()->executeS($sql);
}

/**
 * Obtener testimonios recientes
 */
public function getRecentTestimonios($limit = 10) {
    $sql = 'SELECT t.*, c.firstname, c.lastname
            FROM `' . _DB_PREFIX_ . 'testimonios` t
            LEFT JOIN `' . _DB_PREFIX_ . 'customer` c ON t.customer_id = c.id_customer
            ORDER BY t.created_at DESC
            LIMIT ' . (int)$limit;

    return Db::getInstance()->executeS($sql);
}

/**
 * Obtener estadísticas de testimonios
 */
public function getTestimoniosStats() {
    $sql = 'SELECT
                COUNT(*) as total,
                SUM(CASE WHEN status = "approved" THEN 1 ELSE 0 END) as approved,
                SUM(CASE WHEN status = "pending" THEN 1 ELSE 0 END) as pending,
                SUM(CASE WHEN status = "rejected" THEN 1 ELSE 0 END) as rejected,
                AVG(rating) as avg_rating
            FROM `' . _DB_PREFIX_ . 'testimonios`';

    $result = Db::getInstance()->getRow($sql);

    return [
        'total' => (int)$result['total'],
        'approved' => (int)$result['approved'],
        'pending' => (int)$result['pending'],
        'rejected' => (int)$result['rejected'],
        'avg_rating' => round((float)$result['avg_rating'], 2)
    ];
}
```

#### 4.2 Template de administración completo

**Actualizar views/templates/admin/configure.tpl:**

```smarty
<div class="panel">
    <div class="panel-heading">
        <i class="icon-star"></i>
        {l s='Gestor de Testimonios - Panel de Control' mod='gestortestimonios'}
    </div>
    
    <div class="panel-body">
        {* Estadísticas *}
        <div class="row">
            <div class="col-md-3">
                <div class="kpi-container">
                    <div class="kpi kpi-success">
                        <span class="kpi-title">{l s='Total' mod='gestortestimonios'}</span>
                        <span class="kpi-value">{$estadisticas.total}</span>
                    </div>
                </div>
            </div>
            <div class="col-md-3">
                <div class="kpi-container">
                    <div class="kpi kpi-primary">
                        <span class="kpi-title">{l s='Aprobados' mod='gestortestimonios'}</span>
                        <span class="kpi-value">{$estadisticas.approved}</span>
                    </div>
                </div>
            </div>
            <div class="col-md-3">
                <div class="kpi-container">
                    <div class="kpi kpi-warning">
                        <span class="kpi-title">{l s='Pendientes' mod='gestortestimonios'}</span>
                        <span class="kpi-value">{$estadisticas.pending}</span>
                    </div>
                </div>
            </div>
            <div class="col-md-3">
                <div class="kpi-container">
                    <div class="kpi kpi-info">
                        <span class="kpi-title">{l s='Valoración Media' mod='gestortestimonios'}</span>
                        <span class="kpi-value">{$estadisticas.avg_rating}</span>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

{* Testimonios Pendientes *}
{if $testimonios_pendientes && count($testimonios_pendientes) > 0}
<div class="panel">
    <div class="panel-heading">
        <i class="icon-clock-o"></i>
        {l s='Testimonios Pendientes de Aprobación' mod='gestortestimonios'}
        <span class="badge badge-warning">{count($testimonios_pendientes)}</span>
    </div>
    
    <div class="table-responsive">
        <table class="table">
            <thead>
                <tr>
                    <th>{l s='Cliente' mod='gestortestimonios'}</th>
                    <th>{l s='Empresa' mod='gestortestimonios'}</th>
                    <th>{l s='Testimonio' mod='gestortestimonios'}</th>
                    <th>{l s='Rating' mod='gestortestimonios'}</th>
                    <th>{l s='Fecha' mod='gestortestimonios'}</th>
                    <th>{l s='Acciones' mod='gestortestimonios'}</th>
                </tr>
            </thead>
            <tbody>
                {foreach from=$testimonios_pendientes item=testimonio}
                <tr>
                    <td>
                        <strong>
                            {if $testimonio.firstname && $testimonio.lastname}
                                {$testimonio.firstname|escape:'html'} {$testimonio.lastname|escape:'html'}
                            {else}
                                {$testimonio.client_name|escape:'html'}
                            {/if}
                        </strong>
                        {if $testimonio.email}
                            <br><small class="text-muted">{$testimonio.email|escape:'html'}</small>
                        {/if}
                    </td>
                    <td>
                        {if $testimonio.company}
                            {$testimonio.company|escape:'html'}
                        {else}
                            <span class="text-muted">-</span>
                        {/if}
                    </td>
                    <td>
                        <div class="testimonio-preview">
                            {$testimonio.testimonial_text|escape:'html'|truncate:100:'...':true}
                        </div>
                    </td>
                    <td>
                        {if $testimonio.rating}
                            <div class="rating-display">
                                {for $i=1 to 5}
                                    <i class="icon-star{if $i <= $testimonio.rating} active{/if}"></i>
                                {/for}
                            </div>
                        {else}
                            <span class="text-muted">-</span>
                        {/if}
                    </td>
                    <td>
                        {$testimonio.created_at|date_format:'%d/%m/%Y %H:%M'}
                    </td>
                    <td>
                        <div class="btn-group">
                            <button type="submit" name="approveTestimonio" value="1" 
                                    class="btn btn-success btn-xs"
                                    onclick="document.getElementById('id_testimonio').value = '{$testimonio.id}'; this.form.submit();">
                                <i class="icon-check"></i> {l s='Aprobar' mod='gestortestimonios'}
                            </button>
                            <button type="submit" name="rejectTestimonio" value="1" 
                                    class="btn btn-warning btn-xs"
                                    onclick="document.getElementById('id_testimonio').value = '{$testimonio.id}'; this.form.submit();">
                                <i class="icon-times"></i> {l s='Rechazar' mod='gestortestimonios'}
                            </button>
                            <button type="submit" name="deleteTestimonio" value="1" 
                                    class="btn btn-danger btn-xs"
                                    onclick="if(confirm('{l s='¿Estás seguro de eliminar este testimonio?' mod='gestortestimonios'}')) { document.getElementById('id_testimonio').value = '{$testimonio.id}'; this.form.submit(); }">
                                <i class="icon-trash"></i> {l s='Eliminar' mod='gestortestimonios'}
                            </button>
                        </div>
                    </td>
                </tr>
                {/foreach}
            </tbody>
        </table>
    </div>
</div>
{/if}

{* Configuración *}
<form id="configuration_form" class="defaultForm form-horizontal" 
      action="{$smarty.server.REQUEST_URI}" method="post" enctype="multipart/form-data">
    
    <input type="hidden" id="id_testimonio" name="id_testimonio" value="">
    
    <div class="panel">
        <div class="panel-heading">
            <i class="icon-cogs"></i>
            {l s='Configuración del Módulo' mod='gestortestimonios'}
        </div>
        
        <div class="form-wrapper">
            <div class="form-group">
                <label class="control-label col-lg-3">
                    {l s='Habilitar módulo' mod='gestortestimonios'}
                </label>
                <div class="col-lg-9">
                    <span class="switch prestashop-switch fixed-width-lg">
                        <input type="radio" name="GESTOR_TESTIMONIOS_ENABLED" id="enabled_on" value="1" 
                               {if $gestor_enabled}checked="checked"{/if} />
                        <label for="enabled_on">{l s='Sí' mod='gestortestimonios'}</label>
                        <input type="radio" name="GESTOR_TESTIMONIOS_ENABLED" id="enabled_off" value="0" 
                               {if !$gestor_enabled}checked="checked"{/if} />
                        <label for="enabled_off">{l s='No' mod='gestortestimonios'}</label>
                        <a class="slide-button btn"></a>
                    </span>
                    <p class="help-block">{l s='Habilita o deshabilita completamente el módulo de testimonios.' mod='gestortestimonios'}</p>
                </div>
            </div>

            <div class="form-group">
                <label class="control-label col-lg-3">
                    {l s='Auto-aprobar testimonios' mod='gestortestimonios'}
                </label>
                <div class="col-lg-9">
                    <span class="switch prestashop-switch fixed-width-lg">
                        <input type="radio" name="GESTOR_TESTIMONIOS_AUTO_APPROVE" id="auto_approve_on" value="1" 
                               {if $auto_approve}checked="checked"{/if} />
                        <label for="auto_approve_on">{l s='Sí' mod='gestortestimonios'}</label>
                        <input type="radio" name="GESTOR_TESTIMONIOS_AUTO_APPROVE" id="auto_approve_off" value="0" 
                               {if !$auto_approve}checked="checked"{/if} />
                        <label for="auto_approve_off">{l s='No' mod='gestortestimonios'}</label>
                        <a class="slide-button btn"></a>
                    </span>
                    <p class="help-block">{l s='Si está activado, los testimonios se publican automáticamente sin revisión.' mod='gestortestimonios'}</p>
                </div>
            </div>

            <div class="form-group">
                <label class="control-label col-lg-3">
                    {l s='Testimonios por página' mod='gestortestimonios'}
                </label>
                <div class="col-lg-9">
                    <input type="number" name="GESTOR_TESTIMONIOS_MAX_PER_PAGE" 
                           value="{$max_per_page}" min="1" max="50" class="form-control fixed-width-md" />
                    <p class="help-block">{l s='Número de testimonios que se mostrarán por página en el frontend.' mod='gestortestimonios'}</p>
                </div>
            </div>

            <div class="form-group">
                <label class="control-label col-lg-3">
                    {l s='Rango de valoraciones' mod='gestortestimonios'}
                </label>
                <div class="col-lg-9">
                    <div class="row">
                        <div class="col-md-6">
                            <div class="input-group">
                                <span class="input-group-addon">{l s='Mínimo' mod='gestortestimonios'}</span>
                                <input type="number" name="GESTOR_TESTIMONIOS_MIN_RATING" 
                                       value="{$min_rating}" min="1" max="5" class="form-control" />
                            </div>
                        </div>
                        <div class="col-md-6">
                            <div class="input-group">
                                <span class="input-group-addon">{l s='Máximo' mod='gestortestimonios'}</span>
                                <input type="number" name="GESTOR_TESTIMONIOS_MAX_RATING" 
                                       value="{$max_rating}" min="1" max="5" class="form-control" />
                            </div>
                        </div>
                    </div>
                    <p class="help-block">{l s='Define el rango de estrellas permitido para las valoraciones.' mod='gestortestimonios'}</p>
                </div>
            </div>
        </div>

        <div class="panel-footer">
            <button type="submit" value="1" id="configuration_form_submit_btn" 
                    name="{$submit_action}" class="btn btn-default pull-right">
                <i class="process-icon-save"></i> {l s='Guardar' mod='gestortestimonios'}
            </button>
        </div>
    </div>
</form>
```

***

### 🎨 PASO 5: CSS y JavaScript Funcional

#### 5.1 Estilos CSS

**Archivo: views/css/gestortestimonios.css**

```css
/**
 * CSS SIMPLIFICADO Y AISLADO para Gestor de Testimonios
 * Versión sin conflictos con PrestaShop
 */

/* === CONTENEDOR PRINCIPAL AISLADO === */
.gestortestimonios.testimonio-section {
    background: #f8fafc;
    padding: 2rem 0;
    margin: 0;
    width: 100%;
    clear: both;
    position: relative;
}

.gestortestimonios .testimonio-container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 15px;
}

/* === HEADER DEL FORMULARIO === */
.gestortestimonios .testimonio-header {
    text-align: center;
    margin: 0 auto 3rem;
    padding: 2rem;
    background: linear-gradient(135deg, #f8f9ff 0%, #e8f0ff 100%);
    border-radius: 12px;
    max-width: 800px;
    box-shadow: 0 2px 15px rgba(0,0,0,0.08);
}

.gestortestimonios .testimonio-header h1 {
    font-size: 2.5rem;
    font-weight: 700;
    color: #667eea;
    margin: 0 0 1rem 0;
    line-height: 1.2;
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.gestortestimonios .testimonio-header p {
    font-size: 1.1rem;
    color: #6c757d;
    margin: 0;
    line-height: 1.6;
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

/* === TARJETA PRINCIPAL === */
.gestortestimonios .testimonio-card {
    background: white;
    border-radius: 12px;
    padding: 2.5rem;
    margin: 0 auto 2rem;
    max-width: 800px;
    box-shadow: 0 2px 15px rgba(0,0,0,0.08);
    border: 1px solid #e1e8ed;
    position: relative;
}

/* === ALERTAS === */
.gestortestimonios .alert {
    padding: 1rem 1.5rem;
    border-radius: 8px;
    margin-bottom: 1.5rem;
    border: none;
}

.gestortestimonios .alert-success {
    background: rgba(16, 220, 96, 0.1);
    color: #10dc60;
    border-left: 4px solid #10dc60;
}

.gestortestimonios .alert-danger {
    background: rgba(240, 65, 65, 0.1);
    color: #f04141;
    border-left: 4px solid #f04141;
}

.gestortestimonios .alert-heading {
    font-size: 1.1rem;
    font-weight: 600;
    margin-bottom: 0.5rem;
    display: flex;
    align-items: center;
}

.gestortestimonios .alert .material-icons {
    margin-right: 0.5rem;
    font-size: 1.2rem;
}

/* === FORMULARIO === */
.gestortestimonios .testimonio-form {
    margin: 0;
}

.gestortestimonios .form-section {
    background: white;
    border-radius: 8px;
    padding: 2rem;
    margin-bottom: 2rem;
    border: 1px solid #e1e8ed;
}

.gestortestimonios .section-title {
    font-size: 1.3rem;
    font-weight: 600;
    color: #2c3e50;
    margin-bottom: 1.5rem;
    display: flex;
    align-items: center;
    gap: 0.75rem;
}

.gestortestimonios .section-title .material-icons {
    color: #667eea;
    font-size: 1.5rem;
}

.gestortestimonios .form-group {
    margin-bottom: 1.5rem;
}

.gestortestimonios .form-label {
    display: block;
    font-weight: 600;
    color: #2c3e50;
    margin-bottom: 0.5rem;
    font-size: 0.95rem;
}

.gestortestimonios .form-label.required::after {
    content: '*';
    color: #f04141;
    margin-left: 0.25rem;
}

.gestortestimonios .form-control {
    width: 100%;
    padding: 0.75rem 1rem;
    border: 2px solid #e1e8ed;
    border-radius: 8px;
    font-size: 1rem;
    line-height: 1.5;
    color: #2c3e50;
    background-color: white;
    transition: all 0.3s ease;
    font-family: inherit;
    box-sizing: border-box;
}

.gestortestimonios .form-control:focus {
    outline: none;
    border-color: #667eea;
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.gestortestimonios .form-control:hover {
    border-color: #667eea;
}

.gestortestimonios .form-control::placeholder {
    color: #7f8c8d;
    opacity: 0.7;
}

.gestortestimonios textarea.form-control {
    min-height: 120px;
    resize: vertical;
}

.gestortestimonios .form-text {
    font-size: 0.875rem;
    color: #7f8c8d;
    margin-top: 0.25rem;
}

/* === CONTADOR DE CARACTERES === */
.gestortestimonios .char-counter {
    text-align: right;
    font-size: 0.875rem;
    color: #7f8c8d;
    margin-top: 0.5rem;
}

.gestortestimonios .char-counter.warning {
    color: #ffce00;
}

.gestortestimonios .char-counter.danger {
    color: #f04141;
}

/* === SISTEMA DE RATING === */
.gestortestimonios .rating-input {
    text-align: center;
    padding: 1rem;
    background: #f8fafc;
    border-radius: 8px;
}

.gestortestimonios .rating-label {
    font-weight: 600;
    color: #2c3e50;
    margin-bottom: 1rem;
    display: block;
}

.gestortestimonios .rating-stars {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 0.5rem;
    margin-bottom: 1rem;
}

.gestortestimonios .rating-stars input[type="radio"] {
    display: none;
}

.gestortestimonios .star-label {
    font-size: 2rem;
    color: #ddd;
    cursor: pointer;
    transition: all 0.3s ease;
    position: relative;
    display: inline-block;
}

.gestortestimonios .star-label:hover {
    color: #ffc107;
    transform: scale(1.1);
}

.gestortestimonios .rating-stars input[type="radio"]:checked + .star-label,
.gestortestimonios .rating-stars input[type="radio"]:checked ~ .star-label {
    color: #ffc107;
}

.gestortestimonios .rating-description {
    font-size: 0.95rem;
    color: #7f8c8d;
    font-style: italic;
    min-height: 1.5rem;
}

/* === CHECKBOX DE PRIVACIDAD === */
.gestortestimonios .privacy-group {
    background: rgba(102, 126, 234, 0.05);
    border: 2px solid rgba(102, 126, 234, 0.1);
    border-radius: 8px;
    padding: 1.5rem;
    margin: 2rem 0;
}

.gestortestimonios .privacy-checkbox {
    display: flex;
    align-items: flex-start;
    gap: 1rem;
}

.gestortestimonios .privacy-checkbox input[type="checkbox"] {
    width: 20px;
    height: 20px;
    margin: 0;
    cursor: pointer;
}

.gestortestimonios .privacy-checkbox label {
    flex: 1;
    color: #2c3e50;
    line-height: 1.5;
    cursor: pointer;
    font-size: 0.95rem;
}

/* === BOTONES === */
.gestortestimonios .btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
    padding: 0.75rem 1.5rem;
    font-size: 1rem;
    font-weight: 600;
    line-height: 1.5;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    text-decoration: none;
    transition: all 0.3s ease;
    min-height: 48px;
    font-family: inherit;
}

.gestortestimonios .btn-primary {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
}

.gestortestimonios .btn-primary:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(102, 126, 234, 0.4);
    color: white;
    text-decoration: none;
}

.gestortestimonios .btn-secondary {
    background: #f8f9fa;
    color: #2c3e50;
    border: 2px solid #e1e8ed;
}

.gestortestimonios .btn-secondary:hover {
    background: #e1e8ed;
    transform: translateY(-1px);
    color: #2c3e50;
    text-decoration: none;
}

.gestortestimonios .btn .material-icons {
    font-size: 1.2rem;
}

/* === INFORMACIÓN ADICIONAL === */
.gestortestimonios .additional-info {
    margin-top: 3rem;
}

.gestortestimonios .info-card {
    background: white;
    border-radius: 8px;
    padding: 2rem;
    text-align: center;
    border: 1px solid #e1e8ed;
    transition: all 0.3s ease;
    height: 100%;
    margin-bottom: 2rem;
}

.gestortestimonios .info-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 8px 25px rgba(0,0,0,0.1);
}

.gestortestimonios .info-icon {
    width: 60px;
    height: 60px;
    margin: 0 auto 1rem;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
}

.gestortestimonios .info-icon .material-icons {
    color: white;
    font-size: 1.8rem;
}

.gestortestimonios .info-card h4 {
    font-size: 1.2rem;
    font-weight: 600;
    color: #2c3e50;
    margin-bottom: 1rem;
}

.gestortestimonios .info-card p {
    color: #7f8c8d;
    line-height: 1.6;
    margin: 0;
}

/* === RESPONSIVE === */
@media (max-width: 768px) {
    .gestortestimonios.testimonio-section {
        padding: 1rem 0;
    }
    
    .gestortestimonios .testimonio-container {
        padding: 0 10px;
    }
    
    .gestortestimonios .testimonio-header {
        margin: 1rem auto 2rem;
        padding: 1.5rem;
    }
    
    .gestortestimonios .testimonio-header h1 {
        font-size: 2rem;
    }
    
    .gestortestimonios .testimonio-card {
        padding: 1.5rem;
    }
    
    .gestortestimonios .form-section {
        padding: 1.5rem;
    }
    
    .gestortestimonios .rating-stars {
        gap: 0.25rem;
    }
    
    .gestortestimonios .star-label {
        font-size: 1.5rem;
    }
    
    .gestortestimonios .privacy-checkbox {
        flex-direction: column;
        align-items: flex-start;
        gap: 0.5rem;
    }
}

/* === PREVENIR CONFLICTOS CON PRESTASHOP === */
.gestortestimonios * {
    box-sizing: border-box;
}

/* Asegurar que no se rompa el layout de PrestaShop */
body #header,
body .header-nav,
body #footer {
    position: relative !important;
    z-index: 1000 !important;
}
```

#### 5.2 JavaScript funcional

**Archivo: views/js/gestortestimonios.js**

```javascript
/**
 * JavaScript para el módulo Gestor de Testimonios
 * Archivo: views/js/gestortestimonios.js
 */

document.addEventListener('DOMContentLoaded', function() {
    'use strict';

    console.log('Gestor de Testimonios JS cargado');

    // === CONFIGURACIÓN GLOBAL ===
    const CONFIG = {
        charLimit: 2000,
        charWarning: 1800,
        emailValidationDelay: 500,
        animationDuration: 300
    };

    // === ELEMENTOS DEL DOM ===
    const elements = {
        form: document.getElementById('testimonioForm'),
        testimonialText: document.getElementById('testimonial_text'),
        charCounter: document.querySelector('.char-counter'),
        emailInput: document.getElementById('email'),
        ratingInputs: document.querySelectorAll('input[name="rating"]'),
        ratingDescription: document.querySelector('.rating-description'),
        submitButton: document.querySelector('button[name="submitTestimonio"]'),
        privacyCheckbox: document.getElementById('privacy_accept')
    };

    // === MENSAJES DE RATING ===
    const ratingMessages = {
        1: 'Muy insatisfecho - Nos ayudará a mejorar',
        2: 'Insatisfecho - Valoramos tu opinión',
        3: 'Neutral - Gracias por tu honestidad',
        4: 'Satisfecho - ¡Nos alegra saberlo!',
        5: 'Muy satisfecho - ¡Excelente experiencia!'
    };

    // === FUNCIONES PRINCIPALES ===

    // Inicializar rating con estrellas
    function initStarRating() {
        if (!elements.ratingInputs.length) return;

        elements.ratingInputs.forEach(function(input, index) {
            input.addEventListener('change', function() {
                const rating = parseInt(this.value);
                updateRatingDisplay(rating);
                updateRatingDescription(rating);
            });
        });
    }

    // Actualizar visualización del rating
    function updateRatingDisplay(rating) {
        const stars = document.querySelectorAll('.rating-stars label');
        stars.forEach(function(star, index) {
            if (index < rating) {
                star.classList.add('selected');
            } else {
                star.classList.remove('selected');
            }
        });
    }

    // Actualizar descripción del rating
    function updateRatingDescription(rating) {
        if (elements.ratingDescription && ratingMessages[rating]) {
            elements.ratingDescription.textContent = ratingMessages[rating];
            elements.ratingDescription.style.display = 'block';
        }
    }

    // Inicializar contador de caracteres
    function initCharacterCounter() {
        if (!elements.testimonialText || !elements.charCounter) return;

        elements.testimonialText.addEventListener('input', function() {
            const currentLength = this.value.length;
            const remaining = CONFIG.charLimit - currentLength;

            elements.charCounter.textContent = remaining + ' caracteres restantes';

            if (remaining < 0) {
                elements.charCounter.classList.add('over-limit');
                elements.charCounter.classList.remove('warning');
            } else if (currentLength > CONFIG.charWarning) {
                elements.charCounter.classList.add('warning');
                elements.charCounter.classList.remove('over-limit');
            } else {
                elements.charCounter.classList.remove('warning', 'over-limit');
            }
        });

        // Trigger inicial
        elements.testimonialText.dispatchEvent(new Event('input'));
    }

    // Inicializar formulario de testimonio
    function initTestimonioForm() {
        if (!elements.form) return;

        elements.form.addEventListener('submit', function(e) {
            if (!validateForm()) {
                e.preventDefault();
                return false;
            }

            // Mostrar indicador de carga
            if (elements.submitButton) {
                elements.submitButton.disabled = true;
                elements.submitButton.innerHTML = '<i class="fa fa-spinner fa-spin"></i> Enviando...';
            }
        });

        // Mejorar experiencia visual del formulario
        setupVisualEnhancements();
    }

    function setupVisualEnhancements() {
        var self = this;

        // Añadir contador de caracteres para textarea
        var mensaje = document.getElementById('mensaje');
        if (mensaje) {
            var counter = document.createElement('div');
            counter.className = 'character-counter';
            counter.setAttribute('data-max', '500');
            mensaje.parentNode.appendChild(counter);

            var updateCounter = function() {
                var length = mensaje.value.length;
                var max = 500;
                counter.textContent = length + '/' + max + ' caracteres';

                if (length > max * 0.8) {
                    counter.className = length > max ? 'character-counter error' : 'character-counter warning';
                } else {
                    counter.className = 'character-counter';
                }
            };

            mensaje.addEventListener('input', updateCounter);
            updateCounter();
        }

        // Añadir efectos de focus mejorados
        var inputs = document.querySelectorAll('.testimonios-form input, .testimonios-form textarea');
        inputs.forEach(function(input) {
            input.addEventListener('focus', function() {
                this.parentNode.classList.add('focused');
            });

            input.addEventListener('blur', function() {
                this.parentNode.classList.remove('focused');
            });
        });

        // Mejorar botón de envío con loading state
        var submitBtn = document.querySelector('.btn-primary[type="submit"]');
        if (submitBtn) {
            var form = submitBtn.closest('form');
            if (form) {
                form.addEventListener('submit', function() {
                    submitBtn.classList.add('loading');
                    submitBtn.textContent = 'Enviando...';
                });
            }
        }
    }

    // Validar formulario
    function validateForm() {
        let isValid = true;
        const errors = [];

        // Validar nombre del cliente
        const clientName = document.getElementById('client_name');
        if (clientName && !clientName.value.trim()) {
            errors.push('El nombre del cliente es obligatorio');
            isValid = false;
        }

        // Validar testimonio
        if (elements.testimonialText && !elements.testimonialText.value.trim()) {
            errors.push('El testimonio es obligatorio');
            isValid = false;
        }

        if (elements.testimonialText && elements.testimonialText.value.length > CONFIG.charLimit) {
            errors.push('El testimonio excede el límite de caracteres');
            isValid = false;
        }

        // Validar privacidad
        if (elements.privacyCheckbox && !elements.privacyCheckbox.checked) {
            errors.push('Debes aceptar los términos de privacidad');
            isValid = false;
        }

        // Validar email si se proporciona
        if (elements.emailInput && elements.emailInput.value.trim()) {
            if (!isValidEmail(elements.emailInput.value)) {
                errors.push('El formato del email no es válido');
                isValid = false;
            }
        }

        // Mostrar errores
        if (!isValid) {
            showFormErrors(errors);
        }

        return isValid;
    }

    // Validar email
    function isValidEmail(email) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return emailRegex.test(email);
    }

    // Mostrar errores del formulario
    function showFormErrors(errors) {
        // Remover errores anteriores
        const existingErrors = document.querySelectorAll('.form-error-message');
        existingErrors.forEach(function(error) {
            error.remove();
        });

        // Crear contenedor de errores
        const errorContainer = document.createElement('div');
        errorContainer.className = 'alert alert-danger form-error-message';
        errorContainer.innerHTML = '<ul><li>' + errors.join('</li><li>') + '</li></ul>';

        // Insertar al inicio del formulario
        elements.form.insertBefore(errorContainer, elements.form.firstChild);

        // Scroll al error
        errorContainer.scrollIntoView({ behavior: 'smooth', block: 'center' });
    }

    // === INICIALIZACIÓN ===

    // Inicializar todas las funcionalidades
    if (elements.form) {
        initTestimonioForm();
    }

    initStarRating();
    initCharacterCounter();

    console.log('Todas las funcionalidades del módulo inicializadas');
});

```



#### 6.1 Lista de verificación para testing

**Funcionalidades a probar:**

{% tabs %}
{% tab title="Backend/Admin" %}
* ✅ Instalación del módulo sin errores
* ✅ Configuración guarda correctamente
* ✅ Estadísticas se muestran correctamente
* ✅ Aprobación de testimonios funciona
* ✅ Rechazo de testimonios funciona
* ✅ Eliminación con confirmación
* ✅ Validación de rangos de configuración
{% endtab %}

{% tab title="Frontend" %}
* ✅ Página de listado carga correctamente
* ✅ Paginación funciona
* ✅ Formulario de envío se valida
* ✅ CSRF token funciona
* ✅ Rating interactivo responde
* ✅ Contador de caracteres actualiza
* ✅ Validación de email en tiempo real
* ✅ Responsive design en móviles
{% endtab %}

{% tab title="Seguridad" %}
* ✅ Datos se sanean con pSQL()
* ✅ Validación server-side funciona
* ✅ CSRF protection activo
* ✅ No hay inyecciones SQL posibles
* ✅ Archivos index.php protegen directorios
{% endtab %}
{% endtabs %}

#### 6.2 Pruebas de funcionalidad

{% hint style="info" %}
**Proceso de testing paso a paso:**
{% endhint %}

```bash
bash# 
1. Verificar instalación# - Instalar módulo desde admin# - Comprobar tabla creada en BD# - Verificar configuraciones en ps_configuration# 
2. Probar configuración# - Cambiar valores en admin# - Verificar que se guardan# - Probar validaciones (valores inválidos)# 
3. Enviar testimonio# - Completar formulario en frontend# - Verificar validaciones# - Comprobar que se guarda en BD# 
4. Gestionar testimonio# - Aprobar desde admin# - Verificar aparece en frontend# - Probar rechazo y eliminación
```

***

### 📝 Ejercicios Prácticos Finales

{% tabs %}
{% tab title="Ejercicio 1" %}
#### Búsqueda de Testimonios

Añade funcionalidad de búsqueda en el listado de testimonios:

1. Campo de búsqueda en template
2. Modificar controlador para filtrar
3. Actualizar paginación para mantener búsqueda
{% endtab %}

{% tab title="Búsqueda de Testimonios" %}
Implementa exportación CSV en el admin:

1. Botón de export en configure.tpl
2. Método en clase principal para generar CSV
3. Headers apropiados para descarga
{% endtab %}

{% tab title="Export de Testimonios" %}
Implementa exportación CSV en el admin:

1. Botón de export en configure.tpl
2. Método en clase principal para generar CSV
3. Headers apropiados para descarga
{% endtab %}

{% tab title="Notificaciones por Email " %}
Añade notificación por email cuando llegue nuevo testimonio:

1. Configuración de email en admin
2. Envío automático en saveTestimonio()
3. Template de email en módulo
{% endtab %}
{% endtabs %}

***

### 🏆 Resumen Final

#### ✅ Lo que hemos logrado en esta clase:

1. **Controladores frontend completos** con validación robusta
2. **Templates avanzados** con formularios responsivos y interactivos
3. **Sistema de seguridad** con CSRF y validación de datos
4. **Gestión completa** de testimonios desde admin con estadísticas
5. **CSS y JavaScript** funcional para UX profesional
6. **Testing completo** de todas las funcionalidades

#### 🚀 Funcionalidades implementadas:

{% hint style="success" %}
**Sistema completo de testimonios** con todos los campos
{% endhint %}

{% hint style="info" %}
**Panel de administración** con estadísticas y gestión
{% endhint %}

{% hint style="danger" %}
**Frontend responsivo** con formularios validados
{% endhint %}

{% hint style="warning" %}
**Seguridad robusta** con CSRF y sanitización
{% endhint %}

* ✅ **Rating interactivo** con estrellas
* ✅ **Paginación avanzada** con navegación completa
* ✅ **Validación en tiempo real** con JavaScript
* ✅ **Integración con clientes** de PrestaShop
* ✅ **SEO optimizado** con Schema.org
* ✅ **Sistema de hooks** para integración

#### 🎯 El módulo está listo para producción con:

* Arquitectura escalable y mantenible
* Código limpio siguiendo estándares PrestaShop
* Seguridad implementada correctamente
* UX/UI profesional y responsive
* Testing completo de funcionalidades
* Documentación completa del desarrollo

***

### 📚 Recursos para Continuar

* Añadir caché para consultas frecuentes
* Implementar más filtros y ordenación
* Conectar con otros módulos de PrestaShop
* Añadir métricas y estadísticas avanzadas
* Expandir soporte de traducciones
