# Práctica

Vamos a registrar un evento cada vez que un administrador guarde la configuración de nuestro módulo.

1.  Modifica el método `getContent()` en `advancedtricks.php`. Dentro del `if`, después de que la configuración se guarde con éxito, añade la llamada al Logger.

    PHP

    ```php
    public function getContent()
    {
        $output = '';
        if (Tools::isSubmit('submit' . $this->name)) {
            // ... (código de validación que ya escribimos)
            if (!Validate::isCleanHtml($texto_usuario)) {
                // ...
            } else {
                Configuration::updateValue(self::ADVANCEDTRICKS_TEXT_KEY, $texto_usuario, true);
                $output .= $this->displayConfirmation($this->l('Ajuste guardado correctamente.'));

                // --- INICIO DEL CÓDIGO DEL EJERCICIO ---
                // Añadimos una entrada al log del sistema
                PrestaShopLogger::addLog(
                    sprintf('El módulo AdvancedTricks fue configurado por el empleado ID: %d', $this->context->employee->id),
                    1 // 1=Informativo, 2=Advertencia, 3=Error
                );
                // --- FIN DEL CÓDIGO DEL EJERCICIO ---
            }
        }
        return $output . $this->renderForm();
    }
    ```
2. Prueba: Guardamos el formulario. Luego, para ver el resultado, vamos a Parámetros Avanzados > Registros (Logs). Veremos la nueva entrada que acabamos de crear.
