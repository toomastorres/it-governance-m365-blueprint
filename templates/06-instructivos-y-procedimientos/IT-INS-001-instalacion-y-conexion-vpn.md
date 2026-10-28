# IT-INS-001 — Instructivo de Usuario: Instalación y Conexión a la VPN Corporativa

| Campo | Detalle |
|---|---|
| **Código Documental** | `IT-INS-001` |
| **Versión** | `1.0` |
| **Fecha de Vigencia** | `2026-03-01` |
| **Capítulo** | `06 - Instructivos y Procedimientos` |
| **Audiencia** | Colaboradores autorizados para trabajo remoto (`SG-VPN-Usuarios-Remotos`) |
| **Organización (Ejemplo)** | Atlantic Maritime Agency (AMA) — `atlantic-agency.example.com` |
| **Soporte IT** | `it-soporte@atlantic-agency.example.com` |

---

## 1. ¿Cuándo necesitás conectarte a la VPN?

Con nuestra migración a **Microsoft 365**, la mayoría de tus herramientas diarias funcionan directamente por Internet de forma segura sin abrir la VPN:

| Herramienta / Servicio | ¿Requiere encender la VPN? |
|---|---|
| **Correo Outlook, Microsoft Teams, OneDrive y SharePoint** | **NO** — Funcionan directo desde cualquier conexión a Internet. |
| **Sistemas internos alojados en el servidor local de oficina** | **SÍ** — Debés conectarte primero a la VPN. |
| **Impresoras o recursos exclusivos de la red interna de sede central** | **SÍ** — Solo cuando operes de forma remota autorizada. |

---

## 2. Requisitos Previos

1. Estar operando desde tu **notebook corporativa asignada por AMA** (enrolada en Microsoft Intune).
2. Tener configurada la app **Microsoft Authenticator** en tu teléfono celular (ver instructivo [`IT-INS-004`](IT-INS-004-primer-ingreso-m365-y-mfa.md)).
3. Contar con alta en el grupo de acceso remoto (solicitada por tu responsable de área a IT).

---

## 3. Verificación o Configuración Inicial del Cliente VPN

En las notebooks administradas por AMA, el **Cliente VPN Corporativo** y su perfil ya se instalan de forma automática mediante Microsoft Intune.

1. Buscá el ícono de **escudo / candado del Cliente VPN** en la bandeja inferior derecha de Windows (junto al reloj).
2. Si abrís la aplicación por primera vez y necesitás verificar los parámetros de conexión, confirmá que figuren exactamente así:

| Campo | Valor Oficial |
|---|---|
| **Nombre de la conexión** | `AMA - VPN Corporativa` |
| **Servidor / Gateway remoto** | `vpn.atlantic-agency.example.com` |
| **Puerto personalizado** | `10443` |
| **Método de inicio de sesión** | `Single Sign-On (SSO / Navegador Externo — Microsoft 365)` |

> **Contingencia de enlace secundario:** Únicamente si Soporte IT te informa un mantenimiento programado sobre el enlace principal, utilizá el perfil secundario `vpn-bkp.atlantic-agency.example.com` (puerto `10443`).

---

## 4. Paso a Paso para Conectarte

1. Asegurate de tener conexión estable a Internet (Wi-Fi hogareño o datos móviles personales; evitá redes Wi-Fi abiertas sin contraseña).
2. Abrí el **Cliente VPN** y seleccioná el perfil **`AMA - VPN Corporativa`**.
3. Hacé clic en **Conectar (`Connect / SAML Login`)**.
4. Se abrirá una ventana de inicio de sesión de **Microsoft**:
   - Ingresá tu correo corporativo (`nombre.apellido@atlantic-agency.example.com`) y tu contraseña habitual.
   - En pantalla verás un **número de 2 dígitos**.
5. Abrí la notificación de **Microsoft Authenticator** en tu celular, escribí ese mismo número de 2 dígitos y tocá **Sí (`Approve`)** confirmando con tu huella o rostro.
6. Una vez validado, el estado del cliente pasará a **Conectado (`Connected`)** en pocos segundos y ya podrás abrir los sistemas internos.

---

## 5. Cierre de Sesión al Finalizar

- Cuando termines de usar los sistemas internos o cierres tu jornada laboral, abrí el Cliente VPN y presioná **Desconectar (`Disconnect`)**.
- Recordá bloquear tu pantalla con **`Windows + L`** cada vez que te retires de tu puesto.

---

## 6. Solución Rápida de Problemas Frecuentes

| Síntoma | Causa Probable y Solución |
|---|---|
| **Queda detenido en `40% / 45%` o da error de servidor no alcanzable** | Verificá que tengas Internet activo. Algunas redes de hoteles o aeropuertos bloquean puertos no estándar: probá compartir datos desde tu celular (*Hotspot*) para confirmar. |
| **Muestra `Acceso denegado por política de cumplimiento`** | Tu notebook lleva varios días apagada y tiene actualizaciones pendientes de Windows o Antivirus. Conectate a Internet normal, abrí **Portal de Empresa > Dispositivos > Sincronizar**, aguardá 5 minutos y reintentá. |
| **No llega la notificación a Microsoft Authenticator** | Abrí manualmente la aplicación **Microsoft Authenticator** en tu celular y deslizá hacia abajo para refrescar, o verificá que el teléfono tenga conexión a datos/Wi-Fi. |
| **Aparece una solicitud de MFA que VOS NO INICIASTE** | **¡NUNCA la apruebes!** Presioná **`No soy yo` (`Deny`)** y avisá inmediatamente a `it-soporte@atlantic-agency.example.com`. |

---

## 7. Control de Cambios

| Versión | Fecha | Autor | Descripción del Cambio |
|---|---|---|---|
| `1.0` | `2026-03-01` | Tomás Torres | Emisión inicial del instructivo de conexión VPN con SSO Microsoft Entra ID + MFA. |
