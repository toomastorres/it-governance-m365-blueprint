# IT-INS-004 — Instructivo de Usuario: Primer Ingreso a Microsoft 365 y Configuración de MFA

| Campo | Detalle |
|---|---|
| **Código Documental** | `IT-INS-004` |
| **Versión** | `1.0` |
| **Fecha de Vigencia** | `2026-03-01` |
| **Capítulo** | `06 - Instructivos y Procedimientos` |
| **Audiencia** | Todo el personal (Ingresos nuevos y recambio de teléfono móvil) |
| **Organización (Ejemplo)** | Atlantic Maritime Agency (AMA) — `atlantic-agency.example.com` |
| **Soporte IT** | `it-soporte@atlantic-agency.example.com` |

---

## 1. Bienvenida a tu Cuenta Corporativa Microsoft 365

Tu cuenta `@atlantic-agency.example.com` te da acceso unificado a **Outlook (correo)**, **Microsoft Teams**, **OneDrive (respaldo automático de tus documentos)** y **SharePoint (carpetas de tu área)**.

Para proteger tu identidad y la información de la agencia, el sistema requiere configurar **por única vez** la verificación en dos pasos (**MFA**) con tu teléfono celular. Todo el proceso demora menos de **5 minutos**.

---

## 2. Antes de Empezar: Prepará tu Celular

En tu teléfono móvil (**Android** o **iPhone**), abrí la tienda oficial (**Google Play Store** o **App Store**) e instalá la aplicación gratuita oficial de Microsoft:

- **Nombre exacto:** `Microsoft Authenticator`
- **Desarrollador:** `Microsoft Corporation` *(ícono celeste con un escudo y una silueta dentro)*

> **Importante:** No descargues aplicaciones con publicidad ni nombres parecidos; verificá siempre que el desarrollador sea **Microsoft Corporation**.

---

## 3. Paso a Paso en tu Computadora

### Paso 1 — Ingresar al portal con tu clave temporal

1. En el navegador de tu computadora, entrá a **`https://m365.cloud.microsoft`** (o `https://aka.ms/mfasetup`).
2. Escribí tu correo asignado (ejemplo: `nombre.apellido@atlantic-agency.example.com`) y presioná **Siguiente**.
3. Ingresá la **contraseña temporal de un solo uso** que te entregó el área de IT.

### Paso 2 — Crear tu contraseña personal definitiva

El sistema te pedirá reemplazar la clave temporal por una contraseña propia que solo vos conozcas:

- **Longitud mínima obligatoria:** **14 caracteres**.
- **Consejo práctico:** Usá una **frase fácil de recordar para vos pero imposible de adivinar** combinando palabras, mayúsculas, números y un símbolo (por ejemplo: `PuertoAtlantico48!Sol`).
- **Nunca reutilices** una contraseña que ya uses en tu correo personal o redes sociales.

### Paso 3 — Vincular Microsoft Authenticator (Código QR)

1. La pantalla mostrará el aviso **"Se necesita más información"** → hacé clic en **Siguiente**.
2. Cuando te indique descargar la app Microsoft Authenticator (que ya instalaste en la sección anterior), hacé clic en **Siguiente** hasta ver un **Código QR** en la pantalla de la PC.
3. En tu **teléfono celular**:
   - Abrí **Microsoft Authenticator** y aceptá el permiso de notificaciones y cámara.
   - Tocá el botón **`+` (Agregar cuenta)** arriba a la derecha.
   - Elegí **`Cuenta profesional o educativa`** → **`Escanear un código QR`**.
   - Apuntá la cámara del celular al código QR que ves en la pantalla de tu computadora.
4. Realizá la **prueba de verificación**:
   - En la pantalla de tu PC aparecerá un **número de 2 dígitos** (por ejemplo, `54`).
   - En tu celular saltará una notificación: escribí `54`, presioná **Sí** y validá con tu huella digital, Face ID o PIN del teléfono.
5. Hacé clic en **Siguiente / Listo** en la computadora. ¡Tu cuenta ya está protegida!

---

## 4. Primeros Pasos en tu Equipo Windows

Una vez completado el ingreso:

1. **OneDrive (Respaldo automático):** Verificá que junto al reloj de Windows aparezca una **nube azul** sin cruces rojas. Todo lo que guardes en tus carpetas **`Escritorio`** y **`Documentos`** queda respaldado automáticamente en tu OneDrive corporativo.
2. **Buzones compartidos de tu área (`operaciones@`, `administracion@`, `pagos@`, etc.):** Al abrir **Outlook**, aguardá entre 5 y 10 minutos; las casillas compartidas a las que tu puesto tiene permiso aparecerán automáticamente en el panel izquierdo debajo de tu buzón personal, sin pedirte ninguna contraseña adicional.
3. **Carpetas de tu sector (SharePoint):** Ingresá desde el menú de aplicaciones (ícono de 9 puntos arriba a la izquierda en el navegador) a **SharePoint** para visualizar los sitios de tu equipo.

---

## 5. ¿Cambiaste o perdiste tu teléfono celular?

- **Si compraste un teléfono nuevo y todavía tenés el anterior:** Avisá a `it-soporte@atlantic-agency.example.com` antes de formatear el teléfono viejo, o ingresá a **`https://mysignins.microsoft.com/security-info`** aprobando con el celular actual para agregar el equipo nuevo.
- **Si perdiste o te robaron el celular:** Informá de inmediato a IT para revocar las sesiones activas en el dispositivo extraviado y habilitarte un pase temporal seguro (*Temporary Access Pass*) para vincular tu nuevo teléfono.

---

## 6. Reglas Básicas de Seguridad para el Día a Día

- **Jamás compartas tu contraseña** con compañeros ni con IT (Soporte IT nunca necesita pedirte tu clave personal).
- **Si recibís una solicitud de aprobación en Authenticator que no generaste vos**, tocá **`No soy yo`** y avisá a IT.
- **Correos con cartel `[EXTERNO]`:** Prestá especial atención antes de abrir enlaces o confirmar cambios de cuentas bancarias/CBU/IBAN; ante cualquier duda, validá por teléfono o Teams con el remitente conocido.

---

## 7. Control de Cambios

| Versión | Fecha | Autor | Descripción del Cambio |
|---|---|---|---|
| `1.0` | `2026-03-01` | Tomás Torres | Creación de la guía de alta inicial en Microsoft 365 y configuración paso a paso de MFA. |
