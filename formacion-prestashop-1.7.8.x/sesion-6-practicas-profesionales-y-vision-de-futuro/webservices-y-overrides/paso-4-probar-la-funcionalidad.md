# Paso 4: Probar la Funcionalidad

**Acción:**

1. Visita cualquier página de tu tienda donde se muestren precios (página de inicio, una categoría, una ficha de producto).
2. Observa los precios. Donde antes veías `25,50 €`, ahora verás `25,50 € (IVA incl.)`.

#### Conclusión y Advertencia

Este ejemplo demuestra lo poderoso que es un override: con unas pocas líneas de código has modificado una regla de negocio fundamental en toda la tienda. Sin embargo, también resalta su riesgo: un error en este archivo podría afectar a todos los precios de la tienda. Por eso, los overrides deben usarse con conocimiento y solo cuando no hay otra alternativa (como un hook).
