# ADR-005 — Adaptación a Laravel, versión y reloj
Estado: implementada para el entorno académico.

Se conservan seis tablas, tipos y nombres del contrato. Las migraciones
Laravel de api son las únicas propietarias del DDL.

Concurrencia optimista: xmin::text de PostgreSQL es el testigo del libro.
La escritura exige que coincida con la versión leída. No agrega columnas
al modelo. Si la actualización no afecta exactamente una fila, responde
409 y revierte la transacción. El CHECK de disponibles es la última barrera.
Se comprobó con dos solicitudes HTTP simultáneas.

Un bloqueo transaccional asesor por socio evita exceder tres pendientes
con préstamos simultáneos de libros diferentes. No bloquea pesimistamente
los libros. La devolución condiciona returned_at IS NULL y el incremento
de existencias a una misma transacción.

El reloj es un puerto. Solo en APP_ENV=local admite un archivo temporal
ignorado para sembrar vencidos sin agregar fechas al contrato HTTP.
El sembrador crea los registros por API y restablece el reloj al terminar.
No usar en producción ni durante operaciones o pruebas concurrentes.

JWT: HS256, sub, role, iat y exp; ocho horas. Hash Argon2id.
Claves y credenciales recibidas exclusivamente por entorno.
