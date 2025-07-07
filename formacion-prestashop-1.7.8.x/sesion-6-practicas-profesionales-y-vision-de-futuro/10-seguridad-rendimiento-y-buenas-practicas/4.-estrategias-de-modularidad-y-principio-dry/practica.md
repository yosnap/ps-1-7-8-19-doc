# Práctica

Vamos a refactorizar el código de `advancedtricks.php` para separar la lógica de procesamiento.

1. Crea un nuevo método `postProcess()`: Este método se encargará exclusivamente de la lógica de "guardar".
2. Mueve el código: Corta todo el bloque `if (Tools::isSubmit(...))` del método `getContent()` y pégalo dentro de `postProcess()`.
3. Limpia `getContent()`: Ahora `getContent()` será mucho más simple. Su única tarea será llamar a `postProcess()` y luego a `renderForm()`.

Código Refactorizado (`advancedtricks.php`):

PHP

```php
// ... (constructor, install, uninstall) ...

public function getContent()
{
    // Llama primero al método que procesa el formulario.
    // Este devolverá el mensaje de éxito/error o una cadena vacía.
    $output = $this->postProcess();

    // Luego, concatena el resultado con el formulario.
    return $output . $this->renderForm();
}

/**
 * Nuevo método dedicado a procesar el envío del formulario.
 */
private function postProcess()
{
    $output = '';
    if (Tools::isSubmit('submit' . $this->name)) {
        $texto_usuario = Tools::getValue(self::ADVANCEDTRICKS_TEXT_KEY);

        if (!Validate::isCleanHtml($texto_usuario)) {
            $output = $this->displayError($this->l('El texto contiene código no permitido.'));
        } else {
            Configuration::updateValue(self::ADVANCEDTRICKS_TEXT_KEY, $texto_usuario, true);
            PrestaShopLogger::addLog(
                sprintf('El módulo AdvancedTricks fue configurado por el empleado ID: %d', $this->context->employee->id),
                1
            );
            $output = $this->displayConfirmation($this->l('Ajuste guardado correctamente.'));
        }
    }
    return $output;
}


// ... (el método renderForm() se queda igual) ...
```

**Conclusión del ejercicio:** Aunque el resultado final es el mismo, el código ahora es más limpio. El método `getContent()` es más fácil de leer y la lógica de procesamiento está encapsulada en su propio método, `postProcess()`. Esto es un ejemplo práctico y sencillo de modularidad y del principio DRY.
