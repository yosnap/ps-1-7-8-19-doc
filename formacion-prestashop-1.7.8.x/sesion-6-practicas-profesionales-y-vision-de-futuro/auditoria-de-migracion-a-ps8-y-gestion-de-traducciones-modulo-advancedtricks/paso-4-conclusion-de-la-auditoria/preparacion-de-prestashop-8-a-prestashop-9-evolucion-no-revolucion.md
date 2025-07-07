# Preparación de PrestaShop 8 a PrestaShop 9: Evolución, no Revolución

### ¿Y qué pasa con la compatibilidad para prestashop 9?

Pra la nueva versión 9 de Prestashop, en general, no hace falta una preparación adicional y específica si ya has preparado bien el módulo para la 8.

La razón es que el gran salto arquitectónico fue de PrestaShop 1.7 a PrestaShop 8. La transición de la versión 8 a la 9 es una evolución sobre la misma base moderna (Symfony, PHP 8.1+, etc.), no una revolución.

Un módulo que sigue las buenas prácticas para PrestaShop 8 ya está prácticamente listo para la versión 9. La "auditoría" para pasar de la 8 a la 9 sería mucho más rápida y se centraría en estos puntos:

1. `ps_versions_compliancy` en `config.xml`: El primer paso sería simplemente actualizar la versión máxima para que incluya `9.x`.
2. Versión de PHP: Comprobar si PrestaShop 9 exige una versión mínima de PHP más alta (ej. PHP 8.2) y asegurarse de que el código no usa ninguna función que haya quedado obsoleta en esa nueva versión de PHP.
3. Nuevos Avisos "Deprecated": El trabajo principal consistiría en instalar el módulo en un entorno de PrestaShop 9 con el modo debug activado y revisar si aparecen nuevos avisos de funciones obsoletas que en la versión 8 todavía eran válidas. El proceso de refactorización sería el mismo que ya hemos discutido.
4. Dependencias de Symfony: Si el módulo interactúa muy profundamente con componentes de Symfony, habría que verificar si la actualización de la versión de Symfony entre PrestaShop 8 y 9 afecta a alguna de esas interacciones, pero esto es para casos muy avanzados.

En resumen, si sigues las buenas prácticas para PrestaShop 8 (evitar código obsoleto, usar la arquitectura moderna cuando sea posible, etc.), la transición a la versión 9 será un pequeño ajuste, no una migración completa como la que se requiere desde la 1.7.
