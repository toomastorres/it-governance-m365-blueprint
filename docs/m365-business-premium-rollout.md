# Playbook Técnico: Despliegue y Gobierno de Microsoft 365 Business Premium

> **Organización del caso de estudio:** Atlantic Maritime Agency (AMA) — `atlantic-agency.example.com` (~28 usuarios)  
> **Autor:** Tomás Torres (`@toomastorres`)  
> **Alcance:** Arquitectura de referencia y guía operativa de implementación para transformar una PyME con infraestructura tradicional en un **Modern Workplace seguro, gobernado y auditado** sobre **Microsoft 365 Business Premium**.

---

## Tabla de Contenidos

1. [Por qué Microsoft 365 Business Premium en una PyME](#1-por-qué-microsoft-365-business-premium-en-una-pyme)
2. [Identidad y Control de Acceso en Microsoft Entra ID](#2-identidad-y-control-de-acceso-en-microsoft-entra-id)
3. [Administración y Seguridad de Correo en Exchange Online](#3-administración-y-seguridad-de-correo-en-exchange-online)
4. [Gestión Moderna de Endpoints con Microsoft Intune](#4-gestión-moderna-de-endpoints-con-microsoft-intune)
5. [Gobierno Documental en SharePoint Online y OneDrive](#5-gobierno-documental-en-sharepoint-online-y-onedrive)
6. [Protección contra Amenazas (Defender) e Higiene de Credenciales](#6-protección-contra-amenazas-defender-e-higiene-de-credenciales)
7. [Hoja de Ruta de Implementación (5 Fases)](#7-hoja-de-ruta-de-implementación-5-fases)
8. [Scripts PowerShell de Auditoría y Verificación](#8-scripts-powershell-de-auditoría-y-verificación)

---

## 1. Por qué Microsoft 365 Business Premium en una PyME

En organizaciones de entre 15 y 300 usuarios como **Atlantic Maritime Agency (AMA)**, el error más común es adquirir licencias básicas de correo/ofimática (`Business Basic` o `Business Standard`) y luego intentar cubrir seguridad, MDM, antivirus EDR, cifrado de discos y condicionalidad de accesos con múltiples herramientas aisladas.

**Microsoft 365 Business Premium** consolida en una única licencia por usuario:

| Capa de Arquitectura | Componente Incluido | Problema Real que Resuelve en AMA |
|---|---|---|
| **Identidad (IAM)** | **Microsoft Entra ID P1** | MFA obligatorio, Acceso Condicional granular, bloqueo de protocolos legados y autoservicio de contraseñas (SSPR). |
| **Dispositivos (MDM/MAM)** | **Microsoft Intune Plan 1** | Inventario vivo, políticas de cumplimiento, cifrado **BitLocker** con custodia de claves en la nube y borrado remoto ante robo. |
| **Productividad y Correo** | **Exchange Online + M365 Apps** | Buzones de 50 GB + archivo en línea, buzones compartidos por área sin costo extra y suite Office de escritorio siempre actualizada. |
| **Colaboración y Respaldo** | **SharePoint Online + OneDrive** | Reemplazo de carpetas SMB locales sin gobierno y respaldo transparente de `Escritorio`/`Documentos` con *Known Folder Move*. |
| **Ciberseguridad (EDR + Email)** | **Defender for Business + Defender for O365 P1** | Antivirus de próxima generación, EDR, reglas ASR contra ransomware, *Safe Links*, *Safe Attachments* y anti-spoofing ejecutivo. |

---

## 2. Identidad y Control de Acceso en Microsoft Entra ID

La identidad es el nuevo perímetro de seguridad. Todas las decisiones de acceso parten de **Microsoft Entra ID**.

### 2.1 Separación de Privilegios Administrativos (Tiering)

Ningún administrador utiliza privilegios elevados en su cuenta de correo y uso diario:

- **Cuenta operativa diaria (`tomas.torres@atlantic-agency.example.com`):** Cuenta con licencia *M365 Business Premium* para correo, Teams y trabajo cotidiano, **sin roles de administrador global**.
- **Cuenta administrativa nominada (`adm-ttorres@atlantic-agency.example.com`):** Cuenta dedicada exclusivamente a tareas en portales de administración (`entra.microsoft.com`, `intune.microsoft.com`, `admin.exchange.microsoft.com`), sin buzón de uso comercial ni navegación general.
- **Roles con mínimo privilegio:** Se priorizan roles específicos (`Exchange Administrator`, `Intune Administrator`, `SharePoint Administrator`, `Helpdesk Administrator`) por sobre `Global Administrator` (limitado a máximo 2 cuentas nominadas + 1 cuenta de emergencia).

### 2.2 Cuenta de Emergencia (*Break-Glass Account*)

Para prevenir un bloqueo total del tenant ante errores en políticas de Acceso Condicional o caídas del proveedor celular/autenticador:

- **UPN nativo:** `bg-admin-01@atlanticmaritimeama.onmicrosoft.com` (usa el dominio `.onmicrosoft.com` para no depender de resolución DNS externa ni federación).
- **Credencial:** Contraseña aleatoria de 36+ caracteres generada offline y llave de seguridad física FIDO2, resguardadas en sobre lacrado de Dirección + bóveda `AMA-Tier0-Infraestructura.kdbx`.
- **Exclusión y Monitoreo:** Excluida explícitamente de las políticas de Acceso Condicional, pero configurada con alerta crítica inmediata ante cualquier evento `SignInLogs` exitoso o fallido.

### 2.3 MFA Obligatorio y Políticas de Acceso Condicional (*Conditional Access*)

Con la licencia **Entra ID P1** incluida en Business Premium, se desactivan *Security Defaults* únicamente después de activar en modo *Report-Only* —y luego *On*— el siguiente conjunto base de políticas de **Conditional Access**:

| Código | Nombre de Política | Usuarios / Alcance | Condición | Control de Acceso |
|---|---|---|---|---|
| `CA-001` | `Block-Legacy-Authentication` | Todos los usuarios (excl. *Break-Glass*) | Clientes `Exchange ActiveSync` antiguos, POP3, IMAP4, SMTP básico | **Bloquear acceso** |
| `CA-002` | `Require-MFA-All-Users` | Todos los usuarios (excl. *Break-Glass*) | Todas las aplicaciones en la nube | **Requerir MFA** (Microsoft Authenticator con *Number Matching*) |
| `CA-003` | `Require-Strong-Auth-Admins` | Roles administrativos de Entra ID | Portales de administración Microsoft | **Requerir MFA** + frecuencia de inicio de sesión cada **4 horas** + sesión no persistente |
| `CA-004` | `Block-High-Risk-Countries` | Todos los usuarios (salvo grupo `SG-Viajeros-Autorizados`) | Ubicaciones fuera de países de operación (`AR`, `UY`, `BR`, `IT`) | **Bloquear acceso** |
| `CA-005` | `Require-Compliant-Desktop` | Usuarios internos en apps críticas | Plataformas `Windows` / `macOS` | **Requerir dispositivo marcado como Conforme (*Compliant*) en Intune** |

### 2.4 Gobernanza de Cuentas de Servicio

- **Prohibición de cuentas humanas compartidas:** Ningún empleado inicia sesión con cuentas genéricas como `administracion@` o `operaciones@`. Esas casillas operan como **Buzones Compartidos** delegados.
- **Aplicaciones e impresoras multifunción (Scan-to-Email):** Para equipos internos o scripts de notificación que requieren envío de correo, se utiliza un conector de **SMTP Relay restringido a la IP pública fija primaria de AMA (`203.0.113.10`)** con TLS 1.2 obligatorio, o autenticación moderna OAuth2 / Microsoft Graph API con permisos de aplicación acotados (`Mail.Send` restringido mediante *Application Access Policy*).

---

## 3. Administración y Seguridad de Correo en Exchange Online

### 3.1 Estrategia de Migración y Eliminación de Archivos `.PST`

En el escenario inicial de AMA, múltiples puestos contaban con archivos locales `.pst` de varios gigabytes sin copia de seguridad centralizada y con riesgo constante de corrupción:

1. **Relevamiento previo:** Inventario de casillas activas, alias históricos, listas de correo y archivos `.pst` locales por estación de trabajo.
2. **Migración por lotes:** Sincronización inicial de buzones hacia Exchange Online, seguida de ventana de corte (*Cutover*) de registros `MX` un viernes fuera de horario operativo.
3. **Ingesta de `.PST`:** Importación de historiales locales hacia el **In-Place Archive (Archivo en línea)** de cada usuario en Exchange Online y bloqueo posterior por política de creación de nuevos `.pst` locales.

### 3.2 Buzones Compartidos por Área vs. Listas de Distribución

Para ordenar la operatoria de una agencia marítima (donde los buques, armadores, despachantes y proveedores escriben a casillas funcionales y requieren respuesta trazable en equipo):

| Tipo de Objeto | Ejemplos en AMA (`@atlantic-agency.example.com`) | Licencia Requerida | Uso Recomendado y Configuración |
|---|---|---|---|
| **Buzón Compartido (*Shared Mailbox*)** | `operaciones@`, `administracion@`, `pagos@`, `comercial@`, `it-soporte@` | **Sin licencia** (hasta 50 GB) | Atención grupal con bandeja propia, carpetas de seguimiento y calendario compartido. Se delega `Full Access` + `Send As` vía grupos de seguridad y se activa copia de enviados en ambos buzones. |
| **Grupo de Microsoft 365** | `equipo-finanzas@`, `proyecto-erp@` | Sin licencia extra | Equipos internos que requieren correo grupal + sitio de SharePoint + Planner + canal de Teams integrado. |
| **Lista de Distribución (*DL*)** | `comunicados-internos@`, `alertas-sistemas@` | Sin licencia | Difusión unidireccional de avisos internos o recepción de alertas técnicas hacia múltiples destinatarios sin bandeja propia. |

> **Regla operativa crítica:** Siempre habilitar `MessageCopyForSentAsEnabled` y `MessageCopyForSendOnBehalfEnabled` en todos los buzones compartidos para que cualquier correo respondido por un analista quede visible en `Elementos enviados` del buzón del área y no únicamente en su casilla personal.

### 3.3 Reglas de Transporte (*Mail Flow Rules*) y Anti-Exfiltración

1. **Etiqueta visual de remitente externo:** Regla `ETR-001-External-Sender-Warning` que inserta un aviso superior discreto cuando el remitente es externo (` Outside Organization`) para prevenir ataques de BEC (*Business Email Compromise*) y falsificación de instrucciones de pago.
2. **Bloqueo de auto-reenvío externo:** En *Defender Portal > Email & Collaboration > Anti-spam outbound policy*, el parámetro **Automatic forwarding rules** se fija en `Off - Forwarding is disabled`, impidiendo que un atacante o usuario desvíe copias automáticas de su correo a casillas personales externas.
3. **Bloqueo de adjuntos peligrosos:** Filtro *Common Attachment Types Filter* habilitado en la política Anti-Malware bloqueando `.exe`, `.iso`, `.vhd`, `.vbs`, `.js`, `.hta`, `.scr`, `.bat`, `.cmd`, `.ps1` y `.lnk`.

### 3.4 Autenticación de Dominio: SPF, DKIM y DMARC

Para proteger la reputación de `atlantic-agency.example.com` y evitar que terceros falsifiquen correos en nombre de la agencia:

```dns
; 1. SPF — Autoriza exclusivamente a Exchange Online (HardFail -all)
atlantic-agency.example.com.  IN  TXT  "v=spf1 include:spf.protection.outlook.com -all"

; 2. DKIM — Firma criptográfica RSA 2048-bit gestionada por Exchange Online
selector1._domainkey.atlantic-agency.example.com.  IN  CNAME  selector1-atlantic-agency-example-com._domainkey.atlanticmaritimeama.onmicrosoft.com.
selector2._domainkey.atlantic-agency.example.com.  IN  CNAME  selector2-atlantic-agency-example-com._domainkey.atlanticmaritimeama.onmicrosoft.com.

; 3. DMARC — Política en fase final de rechazo estricto con reportes agregados
_dmarc.atlantic-agency.example.com.  IN  TXT  "v=DMARC1; p=reject; sp=reject; pct=100; adkim=s; aspf=s; rua=mailto:dmarc-reports@atlantic-agency.example.com"
```

---

## 4. Gestión Moderna de Endpoints con Microsoft Intune

### 4.1 Enrolamiento y Estandarización de Equipos

Todos los equipos corporativos (notebooks y PCs de escritorio con **Windows 11 Pro / Business**) se unen directamente a la nube mediante **Microsoft Entra Join + Intune Auto-Enrollment**:

1. **Nomenclatura estándar:** `AMA-NB-XXX` (Notebooks) y `AMA-PC-XXX` (Desktops), garantizando trazabilidad directa contra la planilla maestra del Capítulo `01 - Inventario`.
2. **Eliminación de administrador local para usuarios estándar:** Al enrolar en Entra ID, el perfil *Local user group membership* restringe el grupo local `Administrators` únicamente a `Global Administrators` y al rol `Microsoft Entra Joined Device Local Administrator`, impidiendo instalación de *shadow IT* o ejecución de malware con privilegios elevados.
3. **Windows Autopilot:** Para compras de hardware nuevo, se importa el *Hardware Hash* para que el usuario reciba el equipo sellado, inicie sesión con su correo `@atlantic-agency.example.com` y reciba automáticamente políticas, cifrado y aplicaciones en menos de 30 minutos.

### 4.2 Políticas de Cumplimiento (*Compliance Policies*)

Un equipo Windows solo es marcado como **Conforme (`Compliant`)** —condición necesaria para `CA-005`— si cumple simultáneamente:

- **Windows 10/11 (64-bit)** en versión soportada y con parches de seguridad dentro de ventana.
- **BitLocker Drive Encryption:** Activo y reportando volumen de sistema cifrado.
- **TPM 2.0 y Secure Boot:** Habilitados a nivel UEFI/BIOS.
- **Microsoft Defender Antivirus:** Protección en tiempo real habilitada y firmas actualizadas.
- **Firewall de Windows:** Activo en los tres perfiles (`Domain`, `Private`, `Public`).

### 4.3 Cifrado BitLocker con Custodia Automática en Entra ID (*Key Escrow*)

Uno de los mayores riesgos físicos en operación portuaria y trabajo híbrido es el extravío o robo de notebooks con información financiera, aduanera o comercial. Mediante **Intune > Endpoint Security > Disk Encryption**:

- **Activación silenciosa (`Silent encryption`):** Habilitada para cuentas estándar sin solicitar intervención del usuario.
- **Algoritmo:** `XTS-AES 256-bit` en unidad del sistema operativo y unidades de datos fijas.
- **Custodia garantizada antes de cifrar:** Se habilita *Save BitLocker recovery information to Microsoft Entra ID before enabling BitLocker*. Si el equipo no logra escrowing exitoso de su clave de 48 dígitos en la nube, no inicia el cifrado, evitando bloqueos sin respaldo.
- **Procedimiento operativo documentado:** Ante actualizaciones de BIOS o alertas de TPM, el equipo de IT recupera y rota la clave en minutos siguiendo [`IT-PRO-001`](../templates/06-instructivos-y-procedimientos/IT-PRO-001-recuperacion-clave-bitlocker-intune.md).

### 4.4 Despliegue Automatizado de Aplicaciones y Parches

- **Microsoft 365 Apps for Business:** Desplegado de forma nativa desde Intune (`64-bit`, arquitectura XML sin versiones antiguas MSI, canal *Monthly Enterprise*, idioma `es-ES` / `en-US`).
- **Aplicaciones Win32 (`.intunewin`):** Empaquetado silencioso de agente **VPN Corporativa**, **Adobe Acrobat Reader**, **7-Zip** y utilidades internas homologadas del Capítulo `09 - Instaladores`.
- **Windows Update for Business:**
  - *Anillo 0 — Piloto IT (3 equipos):* Recepción inmediata (Día 0) para validar compatibilidad con herramientas contables/aduaneras.
  - *Anillo 1 — Producción AMA (~25 equipos):* Diferimiento de **7 días** para actualizaciones de calidad mensuales y **30 días** para actualizaciones mayores de características, con reinicio controlado fuera del horario comercial (`09:00–18:00 ART`).

---

## 5. Gobierno Documental en SharePoint Online y OneDrive

### 5.1 De Carpetas Compartidas SMB a Sitios de SharePoint por Área

La estructura heredada basada en un único recurso compartido `\\SRV-FILE01\Datos` presentaba permisos inconsistentes, dependencia obligatoria de VPN para leer archivos simples y riesgo de cifrado masivo por ransomware lateral. Se migró mediante **SharePoint Migration Tool (SPMT)** hacia sitios independientes:

| Sitio de SharePoint Online | URL Ilustrativa | Grupo Edición (Miembros) | Grupo Solo Lectura | Uso y Retención |
|---|---|---|---|---|
| **AMA — Administración y Finanzas** | `/sites/ama-finanzas` | `SG-SPO-Finanzas-RW` | `SG-SPO-Direccion-RO` | Conciliaciones, rendiciones a armadores, tesorería e impuestos. |
| **AMA — Operaciones Marítimas** | `/sites/ama-operaciones` | `SG-SPO-Operaciones-RW` | `SG-SPO-Finanzas-RO` | Legajos de buques/escalas, manifiestos, documentación portuaria. |
| **AMA — Comercial y Atención** | `/sites/ama-comercial` | `SG-SPO-Comercial-RW` | `SG-SPO-Direccion-RO` | Tarifarios, propuestas comerciales y seguimiento de cuentas. |
| **AMA — Gobierno y Operaciones IT** | `/sites/ama-it` | `SG-SPO-IT-Admins` | *(Restringido)* | Repositorio documental codificado (`00` a `99`) — acceso exclusivo IT/Dirección. |

### 5.2 Reglas de Arquitectura en SharePoint Online

1. **Prohibición de permisos directos a personas:** Todo permiso se asigna a grupos de seguridad (`SG-SPO-*`), nunca a usuarios individuales ni rompiendo herencia archivo por archivo.
2. **Control de profundidad de rutas:** Máximo 3–4 niveles de carpetas con nombres breves y metadatos donde aplique, previniendo errores de sincronización por límite de 400 caracteres en URL/ruta local.
3. **Compartición externa bloqueada por defecto en sitios sensibles:** En `/sites/ama-finanzas` y `/sites/ama-it`, la compartición con invitados externos está **deshabilitada**. A nivel tenant, los enlaces anónimos (`Anyone with the link`) están **bloqueados globalmente**.

### 5.3 Respaldo Automático de Puestos con OneDrive Known Folder Move (KFM)

Mediante catálogo de configuración (*Settings Catalog*) de Intune:

- `Silently move Windows known folders to OneDrive` = **Enabled** (Tenant ID configurado).
- `Silently sign in users to the OneDrive sync app with their Windows credentials` = **Enabled**.
- `Use OneDrive Files On-Demand` = **Enabled** (ahorra espacio en discos SSD locales descargando archivos bajo demanda).

Con KFM activo, los archivos que el usuario guarda en su `Escritorio` o `Documentos` están versionados y respaldados continuamente en la nube. Si una notebook falla, el usuario inicia sesión en un equipo de reemplazo y recupera su entorno de trabajo en minutos.

---

## 6. Protección contra Amenazas (Defender) e Higiene de Credenciales

### 6.1 Microsoft Defender for Business y Defender for Office 365

- **Onboarding sin agentes de terceros:** Integración directa entre Intune y el portal `security.microsoft.com` mediante el conector *Microsoft Defender for Endpoint*.
- **Reglas de Reducción de Superficie de Ataque (*ASR Rules*):**
  - Bloquear la creación de procesos secundarios desde aplicaciones de Office.
  - Bloquear macros de Win32 API desde contenido de Office descargado de Internet.
  - Bloquear el robo de credenciales del subsistema `lsass.exe` de Windows.
  - Bloquear la ejecución de archivos ejecutables ofuscados o sin prevalencia/firma confiable.
- **Protección de Correo y Colaboración (*Defender for Office 365 P1*):**
  - **Safe Links:** Reescritura y verificación en tiempo de clic de URLs en correos y mensajes de Microsoft Teams.
  - **Safe Attachments:** Detonación en *sandbox* de archivos adjuntos sospechosos antes de entregarlos en la bandeja de entrada (`Dynamic Delivery`).
  - **Anti-Phishing Ejecutivo:** Protección de nombres de directivos y dominios similares (*typosquatting* contra variaciones de `atlantic-agency.example.com`).

### 6.2 Higiene de Credenciales con KeePassXC (`.kdbx`)

Para secretos de infraestructura que no residen en Entra ID (consola iDRAC/iLO, hipervisor VMware ESXi, consola Next-Gen Firewall, switches gestionables, Veeam Backup, panel de dominio DNS y cuentas *Break-Glass*):

1. **Cero planillas en texto plano:** Queda prohibido almacenar contraseñas en archivos `.xlsx`, `.docx`, notas adhesivas o gestores de navegador personales.
2. **Segmentación en bóvedas `KeePassXC` (`.kdbx` KDF `Argon2id`):**
   - `AMA-Tier0-Infraestructura.kdbx`: Acceso restringido a Dirección e IT Lead (Firewall, ESXi, Veeam, Registrar DNS, Break-Glass).
   - `AMA-Tier1-OperacionesIT.kdbx`: Acceso para soporte técnico operativo (paneles web de impresoras, licencias de terceros, credenciales Wi-Fi secundarias).
3. **Política de rotación:** Rotación inmediata de secretos compartidos ante la baja de cualquier integrante con acceso privilegiado o proveedor externo, registrada en bitácora de auditoría.

---

## 7. Hoja de Ruta de Implementación (5 Fases)

| Fase | Ventana Estimada | Hitos Clave | Entregables Documentales |
|---|---|---|---|
| **Fase 1 — Descubrimiento y Base** | Semana 1 | Relevamiento de usuarios, licencias, DNS, hardware e inventario de carpetas/buzones; alta de tenant y dominio `atlantic-agency.example.com`. | `IT-NOR-000`, Inventario Maestro (`Cap. 01`), `IT-FT-001` |
| **Fase 2 — Identidad y Seguridad Base** | Semana 2 | Creación de cuentas `adm-*` y *Break-Glass*, despliegue de MFA con instructivo a usuarios, políticas `CA-001..004`, bóvedas `KeePassXC`. | `IT-POL-001`, `IT-INS-004` |
| **Fase 3 — Migración Exchange Online** | Semana 3 | Creación de buzones compartidos por área, sincronización de correo, corte `MX` + `SPF/DKIM/DMARC`, reglas anti-phishing y archivo de `.pst`. | `IT-FT-002` |
| **Fase 4 — SharePoint y OneDrive KFM** | Semana 4 | Diseño de sitios y grupos `SG-SPO-*`, migración con SPMT desde servidor SMB local, activación de OneDrive KFM y capacitación por área. | `IT-FT-004` |
| **Fase 5 — Intune MDM, BitLocker y Defender** | Semanas 5–6 | Enrolamiento de notebooks/PCs en Intune, cifrado BitLocker con custodia de claves en Entra ID, onboarding en Defender for Business y pase a `CA-005`. | `IT-FT-003`, `IT-PRO-001`, `IT-INS-001` |

---

## 8. Scripts PowerShell de Auditoría y Verificación

Los siguientes comandos permiten verificar rápidamente que las configuraciones críticas del tenant cumplen con la línea base de gobierno:

```powershell
# ============================================================================
# 1. Exchange Online: Verificar copia de enviados en todos los Shared Mailboxes
# ============================================================================
Import-Module ExchangeOnlineManagement
Connect-ExchangeOnline -UserPrincipalName adm-ttorres@atlantic-agency.example.com

Get-Mailbox -RecipientTypeDetails SharedMailbox |
    Select-Object DisplayName, PrimarySmtpAddress,
                  MessageCopyForSentAsEnabled,
                  MessageCopyForSendOnBehalfEnabled |
    Format-Table -AutoSize

# Habilitar copia automática en cualquier Shared Mailbox que no la tenga activa:
Get-Mailbox -RecipientTypeDetails SharedMailbox |
    Where-Object { -not $_.MessageCopyForSentAsEnabled } |
    Set-Mailbox -MessageCopyForSentAsEnabled $true -MessageCopyForSendOnBehalfEnabled $true
```

```powershell
# ============================================================================
# 2. Microsoft Graph: Auditar estado de custodia de claves BitLocker en Entra ID
# ============================================================================
Import-Module Microsoft.Graph.Identity.SignIns
Import-Module Microsoft.Graph.DeviceManagement

Connect-MgGraph -Scopes "BitlockerKey.ReadBasic.All", "DeviceManagementManagedDevices.Read.All"

$devices = Get-MgDeviceManagementManagedDevice -Filter "operatingSystem eq 'Windows'"
$bitlockerKeys = Get-MgInformationProtectionBitlockerRecoveryKey -All

$report = foreach ($dev in $devices) {
    $hasEscrowKey = $bitlockerKeys.DeviceId -contains $dev.AzureAdDeviceId
    [PSCustomObject]@{
        DeviceName      = $dev.DeviceName
        UserPrincipal   = $dev.UserPrincipalName
        ComplianceState = $dev.ComplianceState
        IsEncrypted     = $dev.IsEncrypted
        KeyInEntraID    = $hasEscrowKey
        LastSyncUTC     = $dev.LastSyncDateTime
    }
}

$report | Sort-Object KeyInEntraID, DeviceName | Format-Table -AutoSize
```

---

> **Referencias cruzadas:** Para las plantillas listas para completar en tu organización, consultá el directorio [`templates/`](../templates/) y el catálogo maestro en [`IT-NOR-000`](../templates/00-indice-y-normas/IT-NOR-000-organizacion-documental-it.md).
