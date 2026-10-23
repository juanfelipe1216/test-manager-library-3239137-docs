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
- 48 pruebas PHPUnit aprobadas, 210 aserciones: dominio, aplicación,
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
