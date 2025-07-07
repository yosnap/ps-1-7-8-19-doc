# Paso 4: Implementación de Hooks

### Hook de Acción: Detectar Login

```php
     /**
     * Se ejecuta cuando un usuario inicia sesión
     * @param array $params
     */
    public function hookActionAuthentication($params) {
        $id_customer = (int)$this->context->customer->id;

        if ($id_customer > 0) {
            // Verificar si ya existe un registro para este cliente
            $existingRecord = Db::getInstance()->getValue(
                'SELECT id_customer FROM `' . _DB_PREFIX_ . 'customer_login_log`
                WHERE id_customer = ' . $id_customer
            );

            if ($existingRecord) {
                // Actualizar la fecha del último login
                Db::getInstance()->update('customer_login_log', [
                    'last_login' => date('Y-m-d H:i:s')
                ], 'id_customer = ' . $id_customer);
            } else {
                // Insertar nuevo registro
                Db::getInstance()->insert('customer_login_log', [
                    'id_customer' => $id_customer,
                    'last_login' => date('Y-m-d H:i:s')
                ]);
            }
        }
    }
```

### Hook de Display: Mostrar Saludo

<pre class="language-php"><code class="lang-php"><strong>    /**
</strong>     * Muestra el saludo en la página de inicio
     * @param array $params
     */
    public function hookDisplayHome($params) {
        // Solo actuar si el cliente está logueado
        if (!$this->context->customer->isLogged()) {
            return;
        }

        $id_customer = (int)$this->context->customer->id;
        $customer_name = $this->context->customer->firstname;

        // Obtener la fecha del último login
        $sql = "SELECT `last_login` FROM `" . _DB_PREFIX_ . "customer_login_log`
                WHERE `id_customer` = " . $id_customer;
        $last_login_date = Db::getInstance()->getValue($sql);

        $this->context->smarty->assign([
            'customer_name' => $customer_name,
            'last_login' => $last_login_date ? Tools::displayDate($last_login_date, null, true) : $this->l('Esta es tu primera visita'),
            'is_first_visit' => !$last_login_date
        ]);

        return $this->display(__FILE__, 'views/templates/hook/home.tpl');
    }
</code></pre>

### Hook de Header: Cargar CSS

```php
    /**
     * Carga los estilos CSS
     * @param array $params
     */
    public function hookDisplayHeader($params) {
        if ($this->context->controller->php_self == 'index') {
            $this->context->controller->registerStylesheet(
                'welcomelogin-css',
                $this->_path . 'views/css/welcomelogin.css',
                ['media' => 'all', 'priority' => 150]
            );
        }
    }
```
