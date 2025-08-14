# 🚖 RideWave API – Diseño de Endpoints (App de transporte tipo Uber)

**Autor:** Hely Santiago Diaz Almeida

---

## 📌 1. Preguntas guía

1. **¿Qué es un “endpoint” en el contexto de una API?**  
    Es una Url del api que se usa para conectarse con el api y poder realizar peticiones etc mediante HTTP.

2. **¿Diferencia entre un endpoint público y uno privado?**  
   - Público: no requiere autenticación. 
   - Privado: requiere autenticación (token) y puede restringirse a roles.

3. **Información confidencial de un usuario:**  
   Email, contraseña, teléfono, documentos, datos de pago, historial de ubicaciones precisas.

4. **Importancia de definir bien métodos HTTP:**  
   Da claridad, seguridad, idempotencia y facilita mantenimiento.

5. **Información que requiere autenticación en este sistema:**  
   Datos de usuario, creación y gestión de viajes, pagos, calificaciones y funciones administrativas.

6. **Seguridad de ubicación:**  
   Uso de HTTPS, almacenamiento mínimo, envío cifrado, actualización controlada y permisos explícitos.

7. **Respuesta si no hay conductores disponibles:**  
   Estado 200 con `available_drivers: 0` y mensaje explicativo, más sugerencia de reintento.

8. **Identificación de recursos principales:**  
   `users`, `drivers`, `vehicles`, `rides`, `payments`, `ratings`, `locations`, `admin`, `notifications`.

9. **Ventajas de versionar la API:**  
   Compatibilidad hacia atrás, despliegue controlado y fácil migración.

10. **Importancia de documentar errores:**  
    Facilita integración, depuración y respuesta consistente ante fallos.

---

## 👥 2. Roles y permisos

- **Pasajero**
  - Crear cuenta / iniciar sesión.
  - Solicitar y cancelar viajes.
  - Realizar pagos.
  - Calificar conductores.
  - Consultar historial.

- **Conductor**
  - Crear cuenta y subir documentos.
  - Marcar disponibilidad.
  - Aceptar, iniciar y completar viajes.
  - Actualizar ubicación.
  - Calificar pasajeros.

- **Administrador**
  - Gestionar usuarios.
  - Revisar reportes.
  - Administrar tarifas y zonas.
  - Consultar métricas.
  - Ejecutar reembolsos y suspensiones.

---

## 📂 3. Recursos principales

- `auth` – autenticación y sesión
- `users` – datos de usuario
- `drivers` – estado y documentos
- `vehicles` – información de vehículos
- `rides` – solicitudes y seguimiento
- `locations` – ubicación en tiempo real
- `payments` – cobros y recibos
- `ratings` – calificaciones
- `notifications` – avisos a usuarios
- `admin` – gestión administrativa
- `support` – tickets y disputas
- `promotions` – promociones y descuentos

---

## 🔗 4. Tabla de Endpoints

| # | Método | Ruta | Descripción | Parámetros (path / query / body) | Auth |
|---:|:---:|:---|:---|:---|:---|
| 1 | POST | `/v1/auth/register` | Registrar usuario (pasajero o conductor) | body: `{name, email, phone, password, role: "passenger"|"driver"}` | Público |
| 2 | POST | `/v1/auth/login` | Login (genera access + refresh token) | body: `{email, password}` | Público |
| 3 | POST | `/v1/auth/refresh` | Refrescar token | body: `{refresh_token}` | Público |
| 4 | POST | `/v1/auth/logout` | Cerrar sesión (revocar token) | headers: Authorization | Requiere token |
| 5 | GET | `/v1/users/me` | Obtener perfil del usuario autenticado | headers | Requiere token |
| 6 | PATCH | `/v1/users/me` | Actualizar perfil | body: campos opcionales | Requiere token |
| 7 | POST | `/v1/drivers/:driverId/documents` | Subir documentos verificatorios | path: driverId; body: multipart files | Token (driver) |
| 8 | GET | `/v1/drivers/:driverId/status` | Estado de conductor | path: driverId | Token (driver/admin/owner) |
| 9 | PATCH | `/v1/drivers/:driverId/availability` | Marcar disponibilidad | path: driverId; body: `{available: true/false}` | Token (driver) |
| 10 | POST | `/v1/locations/update` | Actualizar ubicación | body: `{lat, lng, heading, speed}` | Token (driver) |
| 11 | GET | `/v1/drivers/nearby` | Conductores cercanos | query: `lat,lng,radius_km,vehicle_type` | Token (passenger) |
| 12 | POST | `/v1/rides/estimate` | Estimar tarifa y ETA | body: `{from, to, passengers}` | Token (passenger) |
| 13 | POST | `/v1/rides` | Crear solicitud de viaje | body: `{from, to, payment_method_id, rider_id, options}` | Token (passenger) |
| 14 | GET | `/v1/rides/:rideId` | Detalles de viaje | path: rideId | Token (involved user) |
| 15 | POST | `/v1/rides/:rideId/cancel` | Cancelar viaje | path: rideId; body: `{reason}` | Token (passenger/driver) |
| 16 | POST | `/v1/rides/:rideId/accept` | Aceptar viaje | path: rideId | Token (driver) |
| 17 | POST | `/v1/rides/:rideId/start` | Iniciar viaje | path: rideId; body: `{timestamp}` | Token (driver) |
| 18 | POST | `/v1/rides/:rideId/complete` | Finalizar viaje | path: rideId; body: `{distance_m, duration_s}` | Token (driver) |
| 19 | GET | `/v1/rides/history` | Historial de viajes | query: `page, per_page, role` | Token (user) |
| 20 | POST | `/v1/payments/charge` | Procesar pago | body: `{rideId, method_id}` | Token (system/passenger) |
| 21 | GET | `/v1/payments/:paymentId/receipt` | Ver recibo | path: paymentId | Token (involved user) |
| 22 | POST | `/v1/ratings` | Crear calificación | body: `{rideId, from_user_id, to_user_id, score, comment}` | Token (involved user) |
| 23 | GET | `/v1/notifications` | Notificaciones del usuario | query: `since, unread` | Token (user) |
| 24 | GET | `/v1/admin/users` | Listar usuarios | query: `role,status,page` | Token (admin) |
| 25 | PATCH | `/v1/admin/users/:userId/suspend` | Suspender usuario | path: userId; body: `{suspend:true/false, reason}` | Token (admin) |
| 26 | GET | `/v1/admin/metrics` | Métricas generales | query: `from,to,group_by` | Token (admin) |
| 27 | POST | `/v1/webhooks/notify` | Webhooks externos | body: `{event, payload}` | HMAC / Token |
| 28 | GET | `/v1/vehicles` | Vehículos del conductor | query: `driverId` | Token (driver/admin) |
| 29 | POST | `/v1/support/tickets` | Crear ticket de soporte | body: `{rideId, userId, subject, description, attachments}` | Token (user) |
| 30 | GET | `/v1/promotions/active` | Promociones activas | query: `city,vehicle_type` | Público / token opcional |

---

## 🔄 5. Flujos de uso

### **Flujo A – Solicitud, aceptación y finalización de un viaje**
1. Pasajero solicita estimación: `POST /v1/rides/estimate`.
2. Crea viaje: `POST /v1/rides`.
3. Conductor recibe solicitud y acepta: `POST /v1/rides/:rideId/accept`.
4. Conductor inicia viaje: `POST /v1/rides/:rideId/start`.
5. Conductor finaliza viaje: `POST /v1/rides/:rideId/complete`.
6. Se procesa pago: `POST /v1/payments/charge`.
7. Ambos usuarios califican: `POST /v1/ratings`.

### **Flujo B – Cancelación y devolución parcial**
1. Pasajero cancela viaje: `POST /v1/rides/:rideId/cancel`.
2. Backend evalúa penalización.
3. Si corresponde, se procesa reembolso parcial.

### **Flujo C – Calificación y disputa**
1. Usuario califica: `POST /v1/ratings`.
2. Si rating < 3, crea ticket de soporte: `POST /v1/support/tickets`.
3. Admin revisa y decide acción.

---

## 🛠 6. Decisiones de diseño

- Versionado `/v1/` desde el inicio.
- Autenticación JWT + Refresh tokens.
- RBAC para proteger endpoints por rol.
- Actualización de ubicación segura y eficiente.
- Pagos integrados con pasarela externa.
- Documentación clara de errores y respuestas.

---

## ⚠ 7. Manejo de errores

Formato estándar:
```json

{
  "error": "Descripción del error",
  "code": "ERR_CODE"
}
```
## Situaciones de Error HTTP
- 400 – ERR_INVALID_PAYLOAD: Datos inválidos.
- 401 – ERR_UNAUTHORIZED: Token inválido.
- 403 – ERR_FORBIDDEN: Rol insuficiente.
- 404 – ERR_NOT_FOUND: Recurso no encontrado.
- 409 – ERR_RIDE_ALREADY_ACCEPTED: Viaje ya aceptado.