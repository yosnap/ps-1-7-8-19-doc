# Paso 2: Escribir el Código del Override

Este código intercepta la función que formatea los precios. Llama a la función original para obtener el precio bien formateado (ej. "25,50 €") y luego simplemente le añade nuestro texto.

**Acción:**&#x20;

Pega este código en tu archivo `override/classes/Tools.php`:

```php
<?php

class Tools extends ToolsCore
{
    /**
     * Sobrescribimos el método que muestra los precios.
     * La firma completa del método tiene más parámetros, pero para este ejemplo
     * solo necesitamos los dos primeros.
     *
     * @param float $price El precio numérico
     * @param Currency|int|null $currency El objeto o ID de la moneda
     * @return string El precio formateado con nuestro texto añadido
     */
    public static function displayPrice($price, $currency, $no_utf8 = false, ?Context $context = null)
    {
        // 1. Obtenemos el precio ya formateado por PrestaShop llamando al método original.
        // Esto es una buena práctica para aprovechar toda la lógica de formato de PrestaShop.
        $formattedPrice = parent::displayPrice($price, $currency, $no_utf8, $context);

        // 2. Le añadimos nuestro texto personalizado al final.
        $finalDisplay = $formattedPrice . ' (IVA incl.)';

        // 3. Devolvemos el resultado final.
        return $finalDisplay;
    }
}
```
