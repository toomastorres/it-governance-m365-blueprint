# IT-FT-002 — Ficha Técnica: Tenant Microsoft 365 y Administración de Exchange Online

| Campo | Detalle |
|---|---|
| **Código Documental** | `IT-FT-002` |
| **Versión** | `1.0` |
| **Fecha de Vigencia** | `2026-03-01` |
| **Capítulo** | `04 - Microsoft 365` |
| **Clasificación** | Confidencial Interno — Exclusivo IT |
| **Organización (Ejemplo)** | Atlantic Maritime Agency (AMA) — `atlantic-agency.example.com` |
| **Responsable / Autor** | Tomás Torres — Digital Transformation & IT Governance Lead |

---

## 1. Datos Generales del Tenant Microsoft 365

| Parámetro | Valor Configurado (Ilustrativo) |
|---|---|
| **Nombre de la Organización** | Atlantic Maritime Agency S.A. |
| **Dominio Primario Verificado** | `atlantic-agency.example.com` |
| **Dominio Inicial `.onmicrosoft.com`** | `atlanticmaritimeama.onmicrosoft.com` |
| **Tenant ID (Ejemplo Ficticio)** | `00000000-1111-2222-3333-444444444444` |
| **Región de Datos (Data Residency)** | Americas (NAM / LATAM) |
| **Licencia Base Estandarizada** | **Microsoft 365 Business Premium** |
| **Partner Proveedor CSP (Ficticio)** | `CloudBridge Partner LATAM` (ver contrato en Cap. `07`) |
| **Portales de Administración** | `admin.microsoft.com` · `admin.exchange.microsoft.com` · `security.microsoft.com` |

---

## 2. Registros DNS de Correo y Autenticación de Dominio

Zona DNS pública gestionada en `ExampleRegistrar DNS` (credenciales en `KeePassXC > AMA-Tier0-Infraestructura.kdbx`):

| Tipo | Host / Nombre | Valor / Destino | TTL | Propósito |
|---|---|---|---|---|
| `MX` | `@` | `0 atlantic-agency-example-com.mail.protection.outlook.com.` | `3600` | Recepción de correo en Exchange Online / EOP |
| `CNAME` | `autodiscover` | `autodiscover.outlook.com.` | `3600` | Configuración automática de clientes Outlook |
| `TXT` | `@` | `v=spf1 include:spf.protection.outlook.com -all` | `3600` | **SPF** estricto (`HardFail -all`) |
| `CNAME` | `selector1._domainkey` | `selector1-atlantic-agency-example-com._domainkey.atlanticmaritimeama.onmicrosoft.com.` | `3600` | **DKIM** Selector 1 (RSA 2048-bit) |
| `CNAME` | `selector2._domainkey` | `selector2-atlantic-agency-example-com._domainkey.atlanticmaritimeama.onmicrosoft.com.` | `3600` | **DKIM** Selector 2 (rotación de claves) |
| `TXT` | `_dmarc` | `v=DMARC1; p=reject; sp=reject; pct=100; adkim=s; aspf=s; rua=mailto:dmarc-reports@atlantic-agency.example.com` | `3600` | **DMARC** en modo rechazo contra suplantación |

---

## 3. Catálogo de Buzones Compartidos (*Shared Mailboxes*)

Los buzones funcionales de área **no tienen inicio de sesión directo habilitado** ni consumen licencia (mientras se mantengan bajo 50 GB). El acceso se delega exclusivamente a usuarios nominales mediante grupos:

| Dirección SMTP Primaria | Nombre para Mostrar | Grupo con `Full Access` + `Send As` | Copia en Enviados (`SentAs`) | Propósito Operativo |
|---|---|---|---|---|
| `operaciones@atlantic-agency.example.com` | AMA Operaciones Portuarias | `SG-EXO-Operaciones` | `Habilitada` | Coordinación de recaladas, agenciamiento y manifiestos. |
| `administracion@atlantic-agency.example.com` | AMA Administración | `SG-EXO-Administracion` | `Habilitada` | Facturación, proveedores locales y documentación contable. |
| `pagos@atlantic-agency.example.com` | AMA Tesorería y Pagos | `SG-EXO-Tesoreria` | `Habilitada` | Comprobantes de transferencia, retenciones y cobranzas. |
| `comercial@atlantic-agency.example.com` | AMA Comercial | `SG-EXO-Comercial` | `Habilitada` | Cotizaciones de fletes y atención comercial a clientes. |
| `it-soporte@atlantic-agency.example.com` | Soporte IT AMA | `SG-IT-Admins` | `Habilitada` | Mesa de ayuda interna, alertas técnicas y avisos de mantenimiento. |
| `dmarc-reports@atlantic-agency.example.com` | Reportes DMARC AMA | `SG-IT-Admins` | `N/A (Recepción)` | Recepción automatizada de reportes XML de autenticación DNS. |

---

## 4. Listas de Distribución y Grupos de Microsoft 365

| Dirección SMTP | Tipo | Remitentes Autorizados | Uso |
|---|---|---|---|
| `equipo-ama@atlantic-agency.example.com` | Lista de Distribución Dinámica | **Solo Internos** (moderado por Dirección/RR.HH.) | Comunicados institucionales a todo el personal activo. |
| `alertas-infra@atlantic-agency.example.com` | Lista de Distribución | Conectores internos + Monitoreo | Alertas de Veeam Backup, UPS, Firewall y Microsoft Defender. |

---

## 5. Reglas de Transporte (*Mail Flow*) y Políticas de Seguridad

| Regla / Política | Ubicación en Consola | Estado | Detalle Técnico |
|---|---|---|---|
| `ETR-001-Aviso-Correo-Externo` | `Exchange Admin > Mail flow > Rules` | Activa | Inserta encabezado HTML advirtiendo cuando el remitente es externo a la organización (excepto dominios de casa matriz verificados por `DKIM/SPF`). |
| `Bloqueo Auto-Forwarding Externo` | `Defender > Policies > Anti-spam outbound` | Activa (`Off`) | Impide que reglas de bandeja de entrada reenvíen correos automáticamente hacia dominios externos. |
| `Safe Attachments` | `Defender > Policies > Safe Attachments` | `Dynamic Delivery` | Analiza adjuntos desconocidos en entorno aislado antes de habilitar su apertura. |
| `Safe Links` | `Defender > Policies > Safe Links` | Activa (Email + Teams) | Verifica URLs en tiempo de clic y bloquea sitios de phishing sin opción de omitir advertencia. |
| `Anti-Phishing Ejecutivo` | `Defender > Policies > Anti-phishing` | Activa | Protege contra suplantación de nombres de Directores/Gerentes (*User Impersonation*) y dominios visualmente similares. |
| `Retención y Archivo en Línea` | `Purview / Exchange > Retention` | Activa | *In-Place Archive* habilitado en buzones con alto volumen operativo (> 35 GB) y retención de elementos eliminados extendida a **30 días**. |

---

## 6. Conector de Envío Interno (SMTP Relay Controlado)

Para equipos multifunción (*Scan-to-Email*) o servicios internos de alerta que no soportan inicio de sesión interactivo:

- **Tipo de conector:** `Your organization's email server -> Office 365`
- **Validación:** Restringido estrictamente a la IP pública estática primaria de sede central (`203.0.113.10`).
- **Seguridad:** Requiere `STARTTLS` sobre puerto `25` hacia el endpoint MX del tenant.

---

## 7. Control de Cambios

| Versión | Fecha | Autor | Descripción del Cambio |
|---|---|---|---|
| `1.0` | `2026-03-01` | Tomás Torres | Documentación inicial del tenant, DNS (SPF/DKIM/DMARC), buzones compartidos y políticas Defender for Office 365. |
