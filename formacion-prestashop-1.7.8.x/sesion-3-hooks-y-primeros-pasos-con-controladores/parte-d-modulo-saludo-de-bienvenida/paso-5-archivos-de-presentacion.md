# Paso 5: Archivos de Presentación

### Estilos CSS

Crea el archivo `views/css/welcomelogin.css`:

```css
#welcome-login-box {
    background-color: #f0f8ff; /* AliceBlue */
    border-left: 5px solid #4169e1; /* RoyalBlue */
    padding: 15px 20px;
    margin: 20px 0;
    border-radius: 5px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

#welcome-login-box p {
    margin: 0;
    font-size: 1.1em;
    color: #333;
}

#welcome-login-box strong {
    color: #4169e1;
}
```

### Plantilla Smarty

```php
{* Solo mostramos la caja si las variables existen y tienen contenido *}
{if isset($customer_name) && !empty($customer_name)}
    <div id="welcome-login-box">
        <p>
            {l s='¡Hola de nuevo,' mod='welcomelogin'} <strong>{$customer_name|escape:'htmlall':'UTF-8'}</strong>!
            <br>
            {l s='Tu último acceso fue el' mod='welcomelogin'} {$last_login|escape:'htmlall':'UTF-8'}.
        </p>
    </div>
{/if}
```
