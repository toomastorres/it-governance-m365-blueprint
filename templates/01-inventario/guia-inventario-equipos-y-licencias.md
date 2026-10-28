# Guía y Modelo Maestro: Inventario de Equipos, Activos de Red y Licencias

| Campo | Detalle |
|---|---|
| **Referencia** | `Capítulo 01 - Inventario` (Complemento de `IT-NOR-000`) |
| **Versión** | `1.0` |
| **Fecha de Vigencia** | `2026-03-01` |
| **Clasificación** | Uso Interno — IT y Administración |
| **Organización (Ejemplo)** | Atlantic Maritime Agency (AMA) — `atlantic-agency.example.com` |
| **Responsable / Autor** | Tomás Torres — Digital Transformation & IT Governance Lead |

---

## 1. Propósito

Estandarizar los campos obligatorios, estados de ciclo de vida y criterios de conciliación entre el **inventario físico administrativo-contable** y la **telemetría viva en Microsoft Intune / Entra ID**, asegurando visibilidad completa sobre hardware asignado, garantías, cifrado BitLocker y consumo mensual de licencias.

> **Nota de seguridad para repositorios Git:** El inventario real de producción (con números de serie verdaderos, facturas de compra y nombres de empleados) reside únicamente en el sitio privado de SharePoint `/sites/ama-it/01 - Inventario` y **jamás** se versiona en repositorios públicos (`.xlsx` y `.csv` bloqueados por `.gitignore`). Utilizá las estructuras siguientes para crear las hojas de tu libro maestro.

---

## 2. Hoja 1 — Parque de Estaciones de Trabajo (`Equipos_Usuarios`)

Cada notebook o PC de escritorio debe poseer una etiqueta física coincidente con su `Hostname` configurado en Windows e Intune (`AMA-NB-XXX` o `AMA-PC-XXX`).

### Estructura de Columnas y Datos Ilustrativos (Ficticios)

| ID Activo / Hostname | Tipo | Marca / Modelo | Nro. de Serie (Ilustrativo) | CPU / RAM / Disco | SO | Usuario Asignado (UPN) | Área | Estado Intune | BitLocker Escrow | Fecha Compra | Fin Garantía | Estado Activo |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `AMA-NB-001` | Notebook | Dell Latitude 5440 | `SN-9X44K1-DEMO` | Core i5-1335U / 16 GB / 512 GB NVMe | Win 11 Pro 24H2 | `juan.perez@atlantic-agency.example.com` | Finanzas | `Compliant` | Sí (Entra ID) | `2025-04-10` | `2028-04-10` | En uso |
| `AMA-NB-002` | Notebook | Lenovo ThinkPad E14 Gen 5 | `SN-PF38L9-DEMO` | Core i7-1355U / 16 GB / 512 GB NVMe | Win 11 Pro 24H2 | `maria.gomez@atlantic-agency.example.com` | Operaciones | `Compliant` | Sí (Entra ID) | `2025-06-15` | `2028-06-15` | En uso |
| `AMA-PC-004` | Mini PC | HP Pro Mini 400 G9 | `SN-MXL209-DEMO` | Core i5-12500T / 16 GB / 256 GB NVMe | Win 11 Pro 24H2 | `recepcion@atlantic-agency.example.com` | Recepción | `Compliant` | Sí (Entra ID) | `2024-11-20` | `2027-11-20` | En uso |
| `AMA-NB-019` | Notebook | Dell Latitude 5430 | `SN-7B11Q0-DEMO` | Core i5-1235U / 16 GB / 512 GB NVMe | Win 11 Pro 24H2 | *(Stock de contingencia IT)* | IT | `Compliant` | Sí (Entra ID) | `2024-05-03` | `2027-05-03` | Stock frío listo |

### Reglas de Gobierno de Hardware

1. **Stock de Contingencia (*Cold Spare*):** Mantener al menos **1 a 2 notebooks preparadas** en estado `Stock frío listo` (enroladas en Intune, parcheadas y con M365 Apps instalado) para reemplazo inmediato en menos de 20 minutos ante rotura o siniestro de un equipo operativo.
2. **Acta de Entrega y Devolución:** Toda entrega o cambio de notebook se respalda con comprobante firmado (físico o digital) registrando cargador original, periféricos adicionales y estado estético.
3. **Política de Renovación (*Refresh Cycle*):** Vida útil objetivo de **36 a 48 meses** para notebooks operativas y **60 meses** para puestos fijos de escritorio; prohibido incorporar equipos con ediciones `Windows Home` (no soportan unión nativa a Entra ID ni gobierno completo de BitLocker/Intune).

---

## 3. Hoja 2 — Infraestructura Crítica y Red (`Activos_Infraestructura`)

| ID Activo | Rol | Equipo / Modelo (Ilustrativo) | Ubicación | IP Gestión (RFC 1918) | Nro. Serie | Proveedor Soporte | Fin Soporte / Licencia | Referencia Ficha Técnica |
|---|---|---|---|---|---|---|---|---|
| `FW-AMA-BUE-01` | Next-Gen Firewall + VPN | Appliance UTM Empresarial | Rack Principal U04 | `10.20.10.1` | `FW-DEMO-0091` | Partner Seguridad Red | `2027-08-31` | `IT-FT-001` |
| `SW-AMA-CORE-01` | Switch Core L2+ 48p PoE+ | Switch Gestionable Gigabit | Rack Principal U06 | `10.20.10.2` | `SW-DEMO-1142` | Integrador Red | Garantía Vitalicia | `IT-FT-001` |
| `AP-AMA-PISO1-01` | Access Point Wi-Fi 6 | AP Corporativo Dual-Band | Oficina Norte | `10.20.10.11` | `AP-DEMO-3301` | Integrador Red | En vigencia | `IT-FT-001` |
| `ESXI-AMA-01` | Hipervisor Local | Servidor Rack Dual Xeon / RAID10 | Rack Principal U10 | `10.20.30.10` | `SRV-DEMO-8820` | Soporte Hardware OEM | `2027-12-15` | Cap. `05` |
| `NAS-AMA-BKP-01` | Repositorio Primario Veeam | NAS 4-Bay RAID 6 Dedicado | Sala Secundaria Aislada | `10.20.30.20` | `NAS-DEMO-5510` | Integrador Storage | `2028-02-01` | Cap. `05` |
| `UPS-AMA-RACK-01` | UPS Online Doble Conversión | UPS 3000VA Rackeable + SNMP | Rack Principal U01 | `10.20.10.25` | `UPS-DEMO-7741` | Mantenimiento Eléctrico | Cambio baterías `2027-06` | Cap. `05` |

---

## 4. Hoja 3 — Control de Licenciamiento Cloud (`Licencias_M365_y_SaaS`)

Permite conciliar mensualmente la factura del partner **CSP** contra usuarios reales activos, evitando pagar licencias huérfanas de ex-empleados:

| SKU / Producto | Modalidad Compromiso | Cantidad Contratada | Asignadas | Disponibles (Buffer) | Criterio de Asignación | Revisión |
|---|---|---|---|---|---|---|
| **Microsoft 365 Business Premium** | Anual / Facturación Mensual CSP | `28` | `27` | `1` | Estándar para 100% del personal con PC/Notebook corporativa. | Mensual |
| **Power BI Pro** | Anual CSP | `5` | `5` | `0` | Dirección, Finanzas y Analistas de Gestión. | Mensual |
| **Exchange Online Archiving / Buzones Compartidos** | Incluido sin cargo hasta 50 GB | `6` buzones | `6` | — | Casillas de área (`operaciones@`, `pagos@`, `administracion@`, etc.) + ex-empleados retenidos. | Trimestral |

---

## 5. Rutina Mensual de Conciliación de Inventario (Checklist de 15 Minutos)

1. Exportar desde **Microsoft Intune (`Devices > All devices > Export`)** el listado actualizado de equipos Windows con su fecha de `Last check-in`.
2. Identificar cualquier dispositivo sin contacto superior a **30 días** (equipo guardado, en reparación o dado de baja no depurado en Entra ID).
3. Verificar en **Microsoft 365 Admin Center (`Billing > Licenses`)** que ninguna licencia *Business Premium* permanezca asignada a cuentas con inicio de sesión bloqueado tras convertirse en *Shared Mailbox*.
4. Revisar vencimientos de dominios DNS, certificados SSL y licencias de Firewall dentro de los próximos **60 días**.
