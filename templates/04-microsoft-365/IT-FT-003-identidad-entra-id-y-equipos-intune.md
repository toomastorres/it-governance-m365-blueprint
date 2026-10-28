# IT-FT-003 — Ficha Técnica: Identidad (Microsoft Entra ID) y Equipos (Microsoft Intune MDM)

| Campo | Detalle |
|---|---|
| **Código Documental** | `IT-FT-003` |
| **Versión** | `1.0` |
| **Fecha de Vigencia** | `2026-03-01` |
| **Capítulo** | `04 - Microsoft 365` |
| **Clasificación** | Confidencial Interno — Exclusivo IT |
| **Organización (Ejemplo)** | Atlantic Maritime Agency (AMA) — `atlantic-agency.example.com` |
| **Responsable / Autor** | Tomás Torres — Digital Transformation & IT Governance Lead |

---

## 1. Arquitectura de Identidad en Microsoft Entra ID

Todos los usuarios y dispositivos corporativos se autentican de forma nativa en la nube (**Cloud-Native Identity** con **Microsoft Entra ID P1**, sin dependencia de controladores Active Directory locales heredados).

### 1.1 Cuentas Privilegiadas y Emergencia

| Cuenta / Patrón | Rol Asignado | Licencia | MFA / Protección | Custodia |
|---|---|---|---|---|
| `adm-ttorres@atlantic-agency.example.com` | `Global Administrator` | Sin licencia de buzón | MFA obligatorio + reautenticación cada 4 h (`CA-003`) | `KeePassXC > Tier0` |
| `adm-soporte01@atlantic-agency.example.com` | `Intune Admin` + `Exchange Admin` | Sin licencia de buzón | MFA obligatorio + reautenticación cada 4 h (`CA-003`) | `KeePassXC > Tier0` |
| `bg-admin-01@atlanticmaritimeama.onmicrosoft.com` | `Global Administrator` (*Break-Glass*) | Sin licencia | Excluida de CA + Llave FIDO2 + Alerta de auditoría inmediata | Sobre lacrado Dirección + `KeePassXC > Tier0` |

### 1.2 Políticas Activas de Acceso Condicional (*Conditional Access*)

*Security Defaults* se encuentra **Deshabilitado** por utilizar políticas personalizadas de Entra ID P1:

1. **`CA-001-Block-Legacy-Auth`:** Bloquea cualquier cliente que intente usar protocolos heredados sin soporte moderno de OAuth2.
2. **`CA-002-Require-MFA-All-Users`:** Exige MFA a todos los usuarios en todas las aplicaciones cloud (incluyendo integración SAML de la VPN corporativa).
3. **`CA-003-Require-Strong-Session-Admins`:** Exige MFA resistente, bloquea persistencia de cookies del navegador y limita la sesión de portales de administración a 4 horas.
4. **`CA-004-Geo-Restriction`:** Permite inicios de sesión solo desde ubicaciones autorizadas (`Argentina` y países de operación marítima declarados), salvo miembros temporales del grupo `SG-Viajeros-Autorizados`.
5. **`CA-005-Require-Compliant-Device`:** Exige que el acceso desde equipos de escritorio Windows se realice únicamente desde estaciones enroladas y marcadas como `Compliant` en Microsoft Intune.

### 1.3 Autoservicio de Restablecimiento de Contraseña (SSPR)

- **Estado:** Habilitado para el grupo `SG-Usuarios-Corporativos`.
- **Métodos requeridos para restablecer:** **2 métodos verificados** (Notificación/Código en Microsoft Authenticator + Teléfono móvil registrado).

---

## 2. Gestión de Dispositivos en Microsoft Intune (MDM)

### 2.1 Enrolamiento Automático

- **MDM User Scope (`Devices > Enroll devices > Windows enrollment > Automatic Enrollment`):** `All` (todo usuario con licencia Business Premium que une un equipo Windows 10/11 Pro a Entra ID queda enrolado automáticamente en Intune).
- **Restricciones de enrolamiento:**
  - Bloqueado el enrolamiento de equipos Windows personales (*BYOD Windows* = `Blocked` salvo pre-registro de número de serie corporativo o Autopilot).
  - Límite de dispositivos por usuario estándar: **3 dispositivos**.

### 2.2 Política de Cumplimiento Windows (`WIN-COMP-001-Standard`)

| Control Evaluado | Valor Requerido | Acción ante Incumplimiento |
|---|---|---|
| **Cifrado de almacenamiento (BitLocker)** | `Require` | Día 0: Notificación · Día 3: Marcar `Noncompliant` (bloquea `CA-005`) |
| **TPM (Trusted Platform Module) 2.0 y Secure Boot** | `Require` | Día 0: Marcar `Noncompliant` |
| **Microsoft Defender Antivirus & Real-Time Protection** | `Require` | Día 1: Marcar `Noncompliant` |
| **Firewall de Windows activo** | `Require` | Día 1: Marcar `Noncompliant` |
| **Versión mínima de Windows 11** | `10.0.22631` (23H2 o superior) | Día 14: Marcar `Noncompliant` |

---

## 3. Perfiles de Configuración y Seguridad de Endpoints

| Nombre del Perfil en Intune | Tipo de Perfil | Configuraciones Aplicadas | Asignación |
|---|---|---|---|
| `WIN-SEC-BitLocker-Silent-AES256` | `Endpoint Security > Disk Encryption` | Cifrado silencioso `XTS-AES 256-bit`, unidad SO + fijas, **escrow obligatorio de clave de 48 dígitos en Entra ID antes de cifrar**. | Todos los equipos Windows |
| `WIN-CFG-OneDrive-KFM` | `Settings Catalog` | Inicio de sesión automático en OneDrive, redirección silenciosa de `Escritorio`, `Documentos` e `Imágenes`, *Files On-Demand* activo. | Todos los usuarios |
| `WIN-SEC-LocalAdmins-Cleanup` | `Endpoint Security > Account Protection` | Remueve al usuario estándar del grupo local `Administrators`; mantiene solo roles de administración Entra ID. | Todos los equipos Windows |
| `WIN-SEC-Defender-ASR-Baseline` | `Endpoint Security > Antivirus / ASR` | Protección en la nube alta, Protección contra alteraciones (*Tamper Protection*) activa, reglas ASR contra macros maliciosas y robo LSASS. | Todos los equipos Windows |
| `WIN-CFG-WiFi-Corporativo` | `Templates > Wi-Fi` | Configura automáticamente el SSID `AMA-Corporativo` en notebooks. | Todos los equipos Windows |
| `WIN-UPD-Ring0-IT` / `WIN-UPD-Ring1-Prod` | `Update rings for Windows 10 and later` | Piloto IT: 0 días diferimiento · Producción: 7 días calidad / 30 días características; horas activas `09:00–18:00`. | Grupos Piloto / Prod |

---

## 4. Aplicaciones Desplegadas Automáticamente desde Intune

| Aplicación | Formato en Intune | Modo de Asignación | Notas |
|---|---|---|---|
| **Microsoft 365 Apps for Business** | Suite integrada M365 (`64-bit`) | `Required` (Todos los equipos) | Word, Excel, PowerPoint, Outlook, Teams, OneDrive (Canal *Monthly Enterprise*). |
| **Agente VPN Corporativo** | Win32 App (`.intunewin`) | `Required` (`SG-VPN-Usuarios-Remotos`) | Preconfigurado con el perfil `vpn.atlantic-agency.example.com:10443`. |
| **Adobe Acrobat Reader DC** | Microsoft Store App (New) | `Required` (Todos los equipos) | Actualización automática gestionada. |
| **7-Zip (64-bit MSI)** | Line-of-Business / Win32 | `Required` (Todos los equipos) | Compresión estándar libre de adware. |

---

## 5. Procedimientos Operativos Asociados

- **Bloqueo de pantalla azul BitLocker:** Ejecutar procedimiento [`IT-PRO-001`](../06-instructivos-y-procedimientos/IT-PRO-001-recuperacion-clave-bitlocker-intune.md) (incluye rotación obligatoria post-uso de la clave).
- **Extravío o robo de notebook:** Ejecutar acción remota **`Wipe`** desde `intune.microsoft.com > Devices > Windows > [Equipo]`, bloquear sesión del usuario y asentar incidente.

---

## 6. Control de Cambios

| Versión | Fecha | Autor | Descripción del Cambio |
|---|---|---|---|
| `1.0` | `2026-03-01` | Tomás Torres | Estandarización de Entra ID CA, enrolamiento Intune, BitLocker silencioso con Key Escrow y catálogo de aplicaciones. |
