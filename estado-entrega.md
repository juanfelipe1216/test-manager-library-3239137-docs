# Estado de implementación y evidencia

## Stack y organización
Backend Laravel 12 / PHP 8.4; frontend React 19 con Vite; PostgreSQL 16.
Página pública estática con Nginx. Los repositorios son carpetas hermanas.
Domain: entidades y objetos de valor sin Laravel.
Application: puertos y casos de uso sin adaptadores.
Adapters: PostgreSQL, reloj, Argon2id, JWT y portadas.
Http: validación y adaptación al contrato.
AppServiceProvider es el único punto de composición.

## Comprobaciones realizadas
- 58 pruebas PHPUnit aprobadas, 260 aserciones: dominio, aplicación,
  arquitectura, salud HTTP y restricciones reales de PostgreSQL.
  El total incluye dos pruebas de ejemplo del esqueleto Laravel.
- Recorrido HTTP en api/backend/tests/smoke.php: login, autorización,
  libro, socio, préstamo, devolución, duplicados, baja lógica y
  estabilidad del título del reporte tras renombrar.
- Dos peticiones simultáneas al último ejemplar: exactamente 201 y 409.
- Dos devoluciones simultáneas: 200 y 409; disponibles finales uno.
- Seis tablas en library, cinco categorías, ninguna columna DEFAULT.
- ck_book_total_positive y ck_user_role implementados en una segunda
  migración. Inserciones directas inválidas rechazadas por PostgreSQL.
- Portadas: PNG real, descarga, eliminación, tipo falso y exceso de 2 MB comprobados por API.
- npm run build de React aprobado.
- Sembrador ejecutado: 40 libros, 10 socios, 15 préstamos y tres líneas vencidas del lote. El reloj temporal se retiró al terminar.
- Compose inicia PostgreSQL, API, React y página pública.

## Recorrido para revisión
1. Preparar .env siguiendo el documento ARRANQUE.md de infra.
2. Desde infra: docker compose up -d --build.
3. Abrir http://localhost:5173 e ingresar con el admin del entorno.
4. Crear libro y socio; prestar; devolver una línea.
5. Consultar vencidos y reporte con el rango de fechas correspondiente.
6. La presentación abre en http://localhost:8080 sin consumir la API.

## Evidencia reproducible
Desde infra:
- docker compose exec api php artisan test --compact
- docker compose exec api php tests/smoke.php

Desde tool con credenciales en .env:
- node concurrency.mjs
- npm run seed

Smoke y concurrencia crean registros nuevos; no borran el historial.

## Límites que no se dan por cumplidos
- RNF-1 no se ha medido con 5 000 libros ni demostrado menos de 300 ms.
- Smoke comprueba un recorrido representativo, no todos los endpoints
  y códigos de manera exhaustiva.
- Solo LoginService y las guardas de LendBooksService tienen dobles de
  puertos comprobados; no todos los casos de uso tienen tests con fakes.
- Las pruebas de arquitectura detectan dependencias del núcleo, pero
  no verifican automáticamente todas las instanciaciones fuera de
  AppServiceProvider.
- Los selectores del préstamo cargan hasta 50 libros y socios.
- Almacenamiento de portadas local y servidores de desarrollo para
  el entorno académico; no se entrega despliegue de producción.
- No se afirma cobertura completa ni T-01 a T-23 terminadas.

## Personalización posterior
Se añadió el catálogo de 32 ediciones colombianas reales, con fuentes
en tool/CATALOGO-FUENTES.md e importación por real-catalogue.mjs.
Los libros ficticios se dan de baja y sus préstamos quedan conservados.
Los socios y las cantidades continúan marcados como demostración.

El administrador puede actualizarse desde la consola local:
docker compose exec api php artisan library:credentials juancortes
La contraseña se introduce de forma oculta y se almacena como hash en
PostgreSQL y como configuración local en el .env ignorado.
No se habilita esta operación mediante HTTP.

## Corrección de filtros del catálogo
Cada libro conserva su categoría mediante category_id. El catálogo activo
tiene 26 ediciones de Novela y 6 de Historia; las otras tres categorías
siguen disponibles al crear o editar libros, pero aparecen deshabilitadas
con contador cero en el filtro de consulta.

El texto se busca por título o autor, sin distinguir mayúsculas ni tildes,
y se combina con la categoría. Los cambios reinician la paginación y
cancelan solicitudes anteriores para evitar resultados fuera de orden.
Cuatro pruebas de integración comprueban texto, categoría, paginación y
conteos de libros activos. En navegador se verificó Historia con seis
resultados, y Historia más «garcia» con dos. Compilación React aprobada.

## Extensión solicitada: eliminar socios sin historial
Por petición posterior del usuario se amplía CA-05.3, que originalmente
indicaba que los socios no se eliminan. Solo el administrador puede usar
DELETE /api/v1/members/{id}; un socio sin préstamos se elimina con 204.
Si tiene cualquier préstamo registrado, abierto o devuelto, responde 409
MEMBER_HAS_LOANS y conserva tanto el socio como todo su historial.
El borrado comparte el bloqueo transaccional por socio con los préstamos.
No se modifica el esquema ni se usa borrado en cascada.
La interfaz pide confirmación, informa del resultado y ajusta la página
cuando se elimina la última fila de un listado paginado.
Seis pruebas de integración adicionales verifican borrado, repetición,
permisos, UUID, socio inexistente e historial abierto y devuelto.
La compilación de React y la suite completa de 58 pruebas pasaron.
