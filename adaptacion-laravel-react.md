# Adaptación de Manager Library a Laravel y React

## Objetivo

Implementar el sistema de biblioteca descrito en la especificación,
con React en el frontend y PHP con Laravel en el backend.

Se conservan las historias de usuario, reglas de negocio, modelo de
datos y contrato HTTP. Se adaptan las herramientas de Python/.NET
a sus equivalentes en PHP.

## Organización del backend

- Domain: entidades, objetos de valor y reglas en PHP puro.
- Application: casos de uso e interfaces de los puertos.
- Adapters: HTTP, persistencia, seguridad y almacenamiento.
- Un proveedor de servicios de Laravel conecta los puertos
  con sus implementaciones.

El dominio no depende de Laravel ni de Eloquent.
Los controladores reciben peticiones y llaman a los casos de uso.

## Persistencia

Se utiliza PostgreSQL 16.

Las migraciones del backend crean el esquema library, las seis
tablas, restricciones, índices y las cinco categorías fijas.

Infraestructura solamente proporciona el motor vacío.
No se agregan columnas created_at, updated_at ni valores DEFAULT.

## Reglas que deben conservarse

- Máximo tres libros pendientes por socio.
- Plazo de préstamo de catorce días.
- Un socio con vencidos no puede solicitar otro préstamo.
- Préstamo y descuento de ejemplares dentro de una transacción.
- Concurrencia optimista para evitar prestar el último ejemplar dos veces.
- Devolución por línea, sin incrementar existencias dos veces.
- Baja lógica de libros.
- Estado del préstamo calculado.
- Título, ISBN y categoría congelados en cada línea.

## Seguridad y contrato

JWT firmado con vigencia de ocho horas.
Administrador inicial creado desde variables de entorno.
Contraseñas protegidas mediante hash y nunca expuestas.
Archivo .env excluido de Git; solamente se publica .env.example.

Se respeta /api/v1, UUID, campos snake_case, fechas UTC y el
formato uniforme de errores definido en api-contract.md.

## Verificación prevista

Pruebas de dominio, aplicación con fakes, integración con PostgreSQL,
contrato HTTP, arquitectura y concurrencia.

Una tarea se marcará terminada únicamente cuando tenga evidencia
ejecutable. El estado real, las pruebas y las limitaciones se registran en estado-entrega.md.