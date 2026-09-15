# Protocolo de Integración — Mercado (Tema 09) & Identidad / Usuarios (Tema 01)
## Seguridad Perimetral, Validación de JWT y Propagación de Identidad por Cabeceras

---

## 1. Resumen Ejecutivo del Flujo

El módulo de **Mercado (Tema 09)** delega 100% la autenticación de usuarios y la seguridad perimetral en el **Tema 01 (Identidad y API Gateway)**.

1. **Principio de Confianza Cero Perimetral:** Mercado nunca recibe credenciales (usuario/contraseña) ni valida firmas criptográficas de JWT directamente. Esta labor es responsabilidad exclusiva del API Gateway en el borde de la red.
2. **Propagación Segura de Identidad (Header Injection):** Tras autenticar exitosamente la firma del token, el Gateway inyecta cabeceras HTTP de confianza aguas abajo (`X-User-Id`, `X-Roles`, `X-User-Email`).
3. **Autorización Basada en Roles (RBAC en Mercado):** Mercado lee estas cabeceras y aplica sus reglas de negocio:
   - Rol `ROLE_STUDENT`: Habilitado para consultar catálogo, comprar consumibles, pujar en subastas y gestionar su mochila/slots.
   - Rol `ROLE_PROFESSOR`: Habilitado para curaduría de ofertas por cohorte, ajuste de cupos y lanzamiento de subastas pedagógicas.
   - Rol `ROLE_ADMIN`: Habilitado para auditoría global y parametrización.

---

## 2. Diagrama de Secuencia de Autenticación y Despacho

```mermaid
sequenceDiagram
    autonumber
    actor Cliente as Cliente Web / Front
    participant GW as API Gateway / Auth (Tema 01)
    participant Mercado as Mercado (Tema 09)

    %% Autenticación Perimetral
    Cliente->>GW: Request HTTP (ej: POST /api/v1/market/orders)<br/>Header: Authorization: Bearer <JWT>
    
    critical Verificación Criptográfica en Gateway
        GW->>GW: Valida firma de JWT (Public Key / Secret)
        GW->>GW: Valida vigencia temporal (exp > now)
        GW->>GW: Extrae claims: sub, roles, email
    option Token Inválido o Expirado
        GW-->>Cliente: 401 Unauthorized { error: "INVALID_OR_EXPIRED_TOKEN" }
    end

    %% Inyección y Reenvío
    GW->>Mercado: Request HTTP reescrito con cabeceras seguras<br/>X-User-Id: usr-4821<br/>X-Roles: ROLE_STUDENT<br/>X-User-Email: alumno@frc.utn.edu.ar
    
    %% Validación de Roles en Mercado
    alt Rol Autorizado para la Operación
        Mercado->>Mercado: Procesa lógica de negocio (asocia a usr-4821)
        Mercado-->>GW: 200 OK / 202 Accepted { responsePayload }
        GW-->>Cliente: 200 OK / 202 Accepted { responsePayload }
    else Rol Insuficiente (ej: Alumno queriendo cambiar cupos)
        Mercado-->>GW: 403 Forbidden { error: "ROLE_NOT_PERMITTED" }
        GW-->>Cliente: 403 Forbidden { error: "ROLE_NOT_PERMITTED" }
    end
```

---

## 3. Especificación Técnica de Cabeceras y Contratos

### 3.1 Lo que Enviamos (Cliente → API Gateway Tema 01)

* **Formato de Cabecera:**
  ```http
  Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
  ```
* **Claims estándar esperados dentro del JWT:**
  ```json
  {
    "sub": "usr-4821",
    "email": "estudiante2026@frc.utn.edu.ar",
    "roles": ["ROLE_STUDENT"],
    "fullName": "Tomás Giménez",
    "iat": 1757433600,
    "exp": 1757462400,
    "iss": "aulaquest-auth-service"
  }
  ```

---

### 3.2 Lo que Gateway Reenvía a Mercado (Qué Recibimos de Tema 01)

El Gateway remueve tokens redundantes y envía hacia los microservicios internos las cabeceras normalizadas:

| Cabecera HTTP | Tipo | Ejemplo de Valor | Propósito en Mercado |
|---|---|---|---|
| `X-User-Id` | `String (UUID/ID)` | `usr-4821` | Identificador inmutable del alumno para asignar la compra o consultar la mochila. |
| `X-Roles` | `String (CSV)` | `ROLE_STUDENT` | Control de acceso para discriminar Alumnos de Docentes. |
| `X-User-Email` | `String` | `estudiante@frc.utn.edu.ar` | Traza de auditoría en órdenes de compra. |

---

### 3.3 Qué Hacemos en Mercado con lo Recibido

1. **Extracción y Sanitización en Middleware/Interceptor Spring Boot:**
   ```java
   String userId = request.getHeader("X-User-Id");
   String roles = request.getHeader("X-Roles");
   
   if (userId == null || userId.isBlank()) {
       throw new SecurityException("Identidad de usuario no provista por el Gateway.");
   }
   ```
2. **Auditoría Transaccional:**
   Todo registro en base de datos (`orders`, `student_inventory`, `auctions`) asocia el `student_id = userId`, garantizando que ningún alumno pueda manipular identificadores en el payload para comprar a nombre de otro.
3. **Control de Acceso Fino (RBAC):**
   - Endpoints de compra y equipamiento (`/api/v1/market/orders`, `/inventory/{id}/equip`) exigen `ROLE_STUDENT`.
   - Endpoints de curaduría de cohorte (`/api/v1/market/cohorts/{id}/offers`) exigen `ROLE_PROFESSOR` o `ROLE_ADMIN`.

---

### 3.4 Sincronización de Perfiles (Opcional Asincrónica por Bus)

Para evitar que Mercado deba llamar por HTTP a Identidad cada vez que renderiza el catálogo de una subasta con el nombre del mejor postor:
* **Topic Broker:** `users.lifecycle.events`
* **Evento consumido:** `USER_PROFILE_UPDATED`
* **Acción en Mercado:** Actualiza su tabla de lectura rápida `cached_users (user_id, display_name, avatar_url)`.

---

## 4. Inspección Interactiva de Cada Paso del Flujo (Drill-Down)

A continuación se detalla qué realiza exactamente cada paso de la interacción entre **Mercado (Tema 09)** y **Identidad / API Gateway (Tema 01)**. Haz clic sobre cualquier paso para desplegar su especificación completa:

<details>
<summary><b>Paso 1: Petición HTTP con Token Criptográfico (Cliente → API Gateway)</b></summary>

#### ¿Qué se hace en este paso?
* **Emisor:** Cliente Web / Navegador del Alumno.
* **Receptor:** API Gateway perimetral (Tema 01).
* **Protocolo:** HTTPS REST Seguro (`Authorization: Bearer <jwt>`).
* **Lógica:**
  1. El estudiante realiza una acción autenticada (ej: abrir catálogo, comprar ítem o consultar mochila).
  2. El cliente inyecta en el encabezado HTTP estándar el token JWT firmado previamente al iniciar sesión.
  3. La llamada impacta en el único punto de entrada público expuesto hacia Internet.
* **Cabeceras HTTP Enviadas:**
  ```http
  POST /api/v1/market/orders HTTP/1.1
  Host: api.aulaquest.edu.ar
  Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
  Content-Type: application/json
  ```
* **Manejo de Errores:** Si no se envía el header Authorization, el Gateway rechaza inmediatamente con `401 Unauthorized` sin consultar a ningún microservicio interno.
</details>

<details>
<summary><b>Paso 2: Validación Criptográfica Perimetral en API Gateway (Gateway RS256)</b></summary>

#### ¿Qué se hace en este paso?
* **Emisor:** API Gateway (Tema 01).
* **Receptor:** Validación perimetral en memoria del Gateway.
* **Protocolo:** Verificación criptográfica con Clave Pública (RS256).
* **Lógica:**
  1. Valida matemáticamente la firma del JWT contra la clave pública del servicio de Identidad.
  2. Comprueba la ventana de validez temporal (`exp > now`).
  3. Extrae los claims de identidad: `sub: "usr-4821"`, `roles: ["ROLE_STUDENT"]`, `email: "estudiante@frc.utn.edu.ar"`.
  4. Descarta el token opaco y sanitiza el contexto de la llamada.
* **Contrato de Verificación en Gateway:**
  ```json
  {
    "gatewayCheck": {
      "signatureValid": true,
      "expired": false,
      "extractedUser": "usr-4821",
      "roles": ["ROLE_STUDENT"]
    }
  }
  ```
* **Manejo de Errores:** Si la firma es inválida o el token venció, responde `HTTP 401 Unauthorized { "error": "INVALID_OR_EXPIRED_TOKEN" }` protegiendo toda la red interna de microservicios.
</details>

<details>
<summary><b>Paso 3: Inyección de Cabeceras de Confianza Aguas Abajo (Gateway → Mercado)</b></summary>

#### ¿Qué se hace en este paso?
* **Emisor:** API Gateway (Tema 01).
* **Receptor:** Mercado & Inventario (Tema 09) en Red Privada Docker.
* **Protocolo:** HTTP Interno con Header Injection.
* **Lógica:**
  1. El Gateway reescribe los encabezados del paquete HTTP eliminando el Bearer token (ahorra ancho de banda).
  2. Inyecta encabezados de confianza normalizados:
     - `X-User-Id: usr-4821`
     - `X-Roles: ROLE_STUDENT`
     - `X-User-Email: estudiante@frc.utn.edu.ar`
     - `X-Forwarded-For: 192.168.1.104`
  3. Enruta la petición hacia la IP interna del microservicio de Mercado (`http://mercado-service:8089`).
* **Cabeceras Inyectadas:**
  ```http
  POST /api/v1/market/orders HTTP/1.1
  Host: mercado-service:8089
  X-User-Id: usr-4821
  X-Roles: ROLE_STUDENT
  X-User-Email: estudiante@frc.utn.edu.ar
  ```
</details>

<details>
<summary><b>Paso 4: Control de Acceso RBAC y Ejecución Local en Mercado (Spring Security)</b></summary>

#### ¿Qué se hace en este paso?
* **Emisor:** Mercado & Inventario (Tema 09).
* **Receptor:** Controladores y Casos de Uso del Dominio de Mercado.
* **Protocolo:** Interceptor Spring Boot & `SecurityContextHolder`.
* **Lógica:**
  1. El filtro intercepta la llamada y verifica la presencia de `X-User-Id`.
  2. Evalúa las reglas RBAC locales:
     - `POST /market/orders` o `/inventory/equip`: Exige `ROLE_STUDENT`.
     - `/market/cohorts/{id}/offers`: Exige `ROLE_PROFESSOR` o `ROLE_ADMIN`.
  3. Asocia la transacción exclusivamente a `usr-4821`, impidiendo que ningún usuario suplante la identidad de otro.
* **Base de Datos (Mercado):**
  ```sql
  SELECT * FROM student_inventory WHERE student_id = 'usr-4821';
  ```
* **Manejo de Errores:** Si el rol es insuficiente (ej: un estudiante queriendo modificar cupos docentes), Mercado rechaza con `HTTP 403 Forbidden { "error": "ROLE_NOT_PERMITTED" }`.
</details>

