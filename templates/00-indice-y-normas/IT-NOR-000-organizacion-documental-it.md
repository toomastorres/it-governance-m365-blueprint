# IT-NOR-000 — Norma de Organización y Codificación Documental de IT

| Campo | Detalle |
|---|---|
| **Código Documental** | `IT-NOR-000` |
| **Versión** | `1.0` |
| **Fecha de Vigencia** | `2026-03-01` |
| **Capítulo** | `00 - Índice y Normas` |
| **Clasificación** | Uso Interno — Gobierno IT |
| **Organización (Ejemplo)** | Atlantic Maritime Agency (AMA) — `atlantic-agency.example.com` |
| **Responsable / Autor** | Tomás Torres — Digital Transformation & IT Governance Lead |
| **Ciclo de Revisión** | Anual o ante cambios estructurales de arquitectura |

---

## 1. Objetivo

Establecer una estructura única, predecible y auditable para almacenar, clasificar, nombrar y versionar toda la documentación técnica, políticas de seguridad, inventarios e instructivos del área de Tecnología de la Información (IT), eliminando la dispersión documental y el riesgo operativo de dependencia de una sola persona (*Bus Factor = 1*).

## 2. Alcance

Aplica a todos los documentos técnicos, activos de configuración, manuales de usuario, procedimientos de contingencia, contratos tecnológicos e instaladores administrados por el área de IT de **Atlantic Maritime Agency (AMA)** *(reemplazar por el nombre de tu organización)*.

---

## 3. Estructura de Capítulos del Repositorio de IT (`00` a `99`)

El repositorio oficial de Gobierno IT (alojado en el sitio restringido de SharePoint Online `/sites/ama-it` con respaldo versionado) se organiza estrictamente en los siguientes capítulos numerados:

| Capítulo | Carpeta Oficial | Contenido Autorizado | Acceso |
|---|---|---|---|
| **00** | `00 - Índice y Normas` | Norma documental (`IT-NOR`), políticas corporativas (`IT-POL`) e índice maestro vigente. | IT + Dirección |
| **01** | `01 - Inventario` | Registro maestro de notebooks/PCs, monitores, periféricos, números de serie, licencias M365 y garantías. | IT + Administración |
| **02** | `02 - Credenciales y Accesos` | Bóvedas cifradas `KeePassXC` (`.kdbx`) segmentadas por criticidad y matriz de permisos RBAC. **Prohibido texto plano.** | Exclusivo IT Tier 0/1 |
| **03** | `03 - Red, Firewall y VPN` | Topología LAN/VLAN, direccionamiento IP, enlaces ISP redundantes, reglas de Firewall y configuración VPN. | Exclusivo IT |
| **04** | `04 - Microsoft 365` | Fichas técnicas del tenant M365, Entra ID, Exchange Online, Intune MDM, BitLocker y SharePoint Online. | Exclusivo IT |
| **05** | `05 - Servidores y Backups` | Hipervisor VMware ESXi, máquinas virtuales locales, estrategia `Veeam 3-2-1` y actas de prueba de restauración. | Exclusivo IT |
| **06** | `06 - Instructivos y Procedimientos` | Guías paso a paso para usuarios finales (`IT-INS`) y procedimientos técnicos de soporte/contingencia (`IT-PRO`). | `IT-INS`: Todo el personal / `IT-PRO`: IT |
| **07** | `07 - Proveedores y Contratos` | Contratos ISP, partner de licencias CSP, mantenimiento de hardware, SLAs y matriz de escalamiento técnico. | IT + Administración |
| **08** | `08 - Proyectos y Migraciones` | Business cases, planes de corte (*cutover*), minutas de implementación (ej. migración Cloud / ERP). | IT + Dirección |
| **09** | `09 - Instaladores` | Agentes homologados (VPN, RMM, utilidades corporativas) verificados por suma hash SHA-256. | Exclusivo IT |
| **99** | `99 - Archivo` | Versiones históricas depreciadas o sistemas dados de baja conservados por trazabilidad y auditoría. | Exclusivo IT |

---

## 4. Taxonomía de Prefijos Documentales

Todo documento formal de IT debe identificarse mediante el esquema:

```text
IT-<TIPO>-<NNN>-<slug-descriptivo>
```

| Prefijo | Tipo de Documento | Propósito Principal | Audiencia Objetivo | Ejemplo |
|---|---|---|---|---|
| **`IT-NOR`** | **Norma Técnica** | Define estándares obligatorios de organización, nomenclatura o arquitectura interna de IT. | Equipo IT / Auditores | `IT-NOR-000` |
| **`IT-POL`** | **Política Corporativa** | Establece reglas de cumplimiento obligatorio en seguridad de la información, accesos y uso aceptable. | Toda la organización | `IT-POL-001` |
| **`IT-FT`** | **Ficha Técnica (*As-Built*)** | Documenta la arquitectura real configurada de una plataforma, red o servicio crítico para mantenimiento y DRP. | Administradores IT / Proveedores | `IT-FT-001` a `IT-FT-004` |
| **`IT-INS`** | **Instructivo de Usuario** | Guía visual y simple orientada al autogobierno del usuario final sin jerga innecesaria. | Usuarios finales | `IT-INS-001`, `IT-INS-004` |
| **`IT-PRO`** | **Procedimiento Operativo (*SOP / Runbook*)** | Secuencia exacta y verificable para ejecutar tareas críticas de soporte, altas/bajas o recuperación ante incidentes. | Soporte Técnico / SysAdmins | `IT-PRO-001` |

---

## 5. Reglas de Nomenclatura de Activos Relacionados

Para mantener consistencia entre la documentación, **Microsoft Entra ID**, **Microsoft Intune** y la red local:

- **Estaciones de trabajo:** `<SIGLA>-NB-<NNN>` para notebooks (ej. `AMA-NB-014`) y `<SIGLA>-PC-<NNN>` para equipos de escritorio (ej. `AMA-PC-005`).
- **Infraestructura de red y servidores:** `<ROL>-<SIGLA>-<SEDE>-<NN>` (ej. `FW-AMA-BUE-01`, `SW-AMA-CORE-01`, `ESXI-AMA-01`, `VEEAM-AMA-01`).
- **Cuentas administrativas:** `adm-<inicialapellido>@atlantic-agency.example.com` (separadas siempre de la cuenta diaria del usuario).
- **Grupos de seguridad en Entra ID:** `SG-<PLATAFORMA>-<AREA>-<PERMISO>` (ej. `SG-SPO-Finanzas-RW`, `SG-VPN-Usuarios-Remotos`).

---

## 6. Ciclo de Vida, Versionado y Reglas de Seguridad

1. **Versionado Semántico Documental:**
   - Cambio menor (corrección de redacción, actualización de captura o contacto): incrementa decimal (`1.0` → `1.1`).
   - Cambio estructural (cambio de arquitectura, nueva política de Acceso Condicional o rediseño de red): incrementa entero (`1.1` → `2.0`).
2. **Prohibición de Secretos en Documentos:** Ninguna ficha técnica (`IT-FT`), procedimiento (`IT-PRO`) o planilla puede incluir contraseñas, claves privadas, cadenas de conexión o claves precompartidas IPsec. En su lugar, debe referenciar la ruta lógica dentro de la bóveda `KeePassXC` correspondiente (ej. `KeePassXC > Tier0 > Firewall > FW-AMA-BUE-01`).
3. **Archivado Controlado:** Cuando un sistema o versión queda obsoleto, el archivo no se elimina sin registro; se antecede la fecha `[ARCHIVADO-AAAA-MM]` y se traslada a `99 - Archivo`.

---

## 7. Catálogo Maestro Inicial (Índice Vigente)

| Código | Título del Documento | Capítulo | Estado |
|---|---|---|---|
| `IT-NOR-000` | Organización y Codificación Documental de IT | `00` | Vigente |
| `IT-POL-001` | Política de Credenciales, MFA y Control de Accesos | `00` | Vigente |
| `S/C (Guía)` | Guía y Modelo de Inventario de Equipos y Licencias | `01` | Vigente |
| `IT-FT-001` | Arquitectura de Red, Firewall Perimetral y VPN | `03` | Vigente |
| `IT-FT-002` | Microsoft 365 y Administración de Exchange Online | `04` | Vigente |
| `IT-FT-003` | Identidad (Entra ID) y Gestión de Equipos (Intune MDM) | `04` | Vigente |
| `IT-FT-004` | Gobierno de SharePoint Online, OneDrive y Carpetas | `04` | Vigente |
| `IT-INS-001` | Instalación y Conexión a la VPN Corporativa | `06` | Vigente |
| `IT-INS-004` | Primer Ingreso a Microsoft 365 y Configuración de MFA | `06` | Vigente |
| `IT-PRO-001` | Recuperación y Rotación de Clave BitLocker en Intune | `06` | Vigente |

---

## 8. Control de Cambios

| Versión | Fecha | Autor | Descripción del Cambio |
|---|---|---|---|
| `1.0` | `2026-03-01` | Tomás Torres | Emisión inicial del marco documental y taxonomía codificada de IT. |
