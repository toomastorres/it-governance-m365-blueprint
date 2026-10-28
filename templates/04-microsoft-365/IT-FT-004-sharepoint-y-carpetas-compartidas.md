# IT-FT-004 — Ficha Técnica: Gobierno de SharePoint Online, OneDrive y Carpetas Compartidas

| Campo | Detalle |
|---|---|
| **Código Documental** | `IT-FT-004` |
| **Versión** | `1.0` |
| **Fecha de Vigencia** | `2026-03-01` |
| **Capítulo** | `04 - Microsoft 365` |
| **Clasificación** | Confidencial Interno — Exclusivo IT |
| **Organización (Ejemplo)** | Atlantic Maritime Agency (AMA) — `atlantic-agency.example.com` |
| **Responsable / Autor** | Tomás Torres — Digital Transformation & IT Governance Lead |

---

## 1. Modelo de Almacenamiento Corporativo en la Nube

Tras el retiro del antiguo servidor de archivos SMB local (`\\SRV-FILE01\Datos`), toda la documentación corporativa se gobierna bajo una regla clara de dos capas:

| Plataforma | Propósito | Cuota Base | Quién Accede | Respaldo / Versionado |
|---|---|---|---|---|
| **OneDrive for Business** | Trabajo individual en curso + respaldo automático (`KFM`) de `Escritorio`, `Documentos` e `Imágenes` de cada puesto. | `1 TB` por usuario | Exclusivo del usuario titular (transferible al líder en caso de baja). | 500 versiones + Papelera de 93 días |
| **SharePoint Online** | Repositorio oficial departamental y transversal (reemplazo moderno de las unidades de red `Z:\` y `P:\`). | `1 TB` base + `10 GB` por licencia | Equipos de trabajo según matriz RBAC por grupos `SG-SPO-*`. | 500 versiones + Papelera de dos etapas (93 días) |

---

## 2. Mapa de Sitios y Matriz de Permisos RBAC

> **Regla de Oro de Gobierno:** Jamás se otorgan permisos individuales por nombre de persona ni se rompe la herencia de permisos en subcarpetas sueltas. Si una carpeta requiere permisos distintos, debe ser una **Biblioteca de Documentos separada** o un **Sitio independiente**.

| Sitio de SharePoint | URL Relativa | Biblioteca Principal | Grupo Edición (`Contribute` sin eliminar biblioteca) | Grupo Solo Lectura (`Read`) | Compartición Externa |
|---|---|---|---|---|---|
| **AMA — Dirección** | `/sites/ama-direccion` | `Directorio y Actas` | `SG-SPO-Direccion-RW` | — | **Bloqueada** |
| **AMA — Administración y Finanzas** | `/sites/ama-finanzas` | `Contabilidad`, `Tesorería`, `Rendiciones Armador`, `Impuestos` | `SG-SPO-Finanzas-RW` | `SG-SPO-Direccion-RO`, `SG-SPO-Auditores-RO` | **Bloqueada** |
| **AMA — Operaciones Marítimas** | `/sites/ama-operaciones` | `Buques y Escalas`, `Manifiestos`, `Aduana y Puerto` | `SG-SPO-Operaciones-RW` | `SG-SPO-Finanzas-RO`, `SG-SPO-Comercial-RO` | Invitados autenticados existentes |
| **AMA — Comercial** | `/sites/ama-comercial` | `Clientes`, `Cotizaciones`, `Tarifarios` | `SG-SPO-Comercial-RW` | `SG-SPO-Operaciones-RO`, `SG-SPO-Direccion-RO` | Invitados autenticados existentes |
| **AMA — Recursos Institucionales** | `/sites/ama-institucional` | `Plantillas Oficiales`, `Políticas e Instructivos Usuario` | `SG-IT-Admins`, `SG-RRHH-RW` | `SG-Usuarios-Corporativos` (Toda la empresa) | **Bloqueada** |
| **AMA — Gobierno IT** | `/sites/ama-it` | `Repositorio Documental IT (00 a 99)` | `SG-IT-Admins` | `SG-SPO-Direccion-RO` | **Bloqueada** |

---

## 3. Estándar de Sincronización en Equipos Windows

Para brindar una experiencia ágil en el **Explorador de Archivos de Windows** sin saturar discos SSD locales ni generar conflictos de sincronización:

1. **Preferencia por `Agregar acceso directo a OneDrive` o `Sincronizar por biblioteca específica`:** Nunca sincronizar un sitio raíz completo si el usuario solo opera sobre una biblioteca puntual.
2. **Archivos a petición (*Files On-Demand*) obligatorios:** Los archivos permanecen en estado nube (ícono de nube azul) hasta que el usuario los abre; tras **30 días sin uso**, Storage Sense libera espacio local automáticamente volviéndolos a estado `Online-only`.
3. **Límite de profundidad y longitud de ruta:**
   - Profundidad máxima recomendada: **3 a 4 niveles de carpetas** (ej. `Rendiciones Armador / 2026 / Buque-Atlantic-Star / ...`).
   - Evitar nombres de carpetas repetitivos porque Windows + OneDrive imponen un tope práctico de ~255/400 caracteres sumando ruta completa.
4. **Bloqueo de tipos de archivos conflictivos en sincronización:** En `SharePoint Admin Center > Settings > Sync`, se bloquea la subida de archivos `.pst`, `.ost`, `.tmp` y bases de datos locales `.mdb`/`.ldb` que provocan bloqueos continuos de sincronización.

---

## 4. Políticas Globales de Compartición y Prevención de Fuga de Datos

Configuración aplicada en `admin.microsoft.com > SharePoint > Policies > Sharing`:

| Parámetro Global | Configuración Aplicada | Justificación de Seguridad |
|---|---|---|
| **Nivel máximo del Tenant** | `New and existing guests` (requiere código de verificación / inicio de sesión del destinatario) | Elimina por completo los enlaces públicos anónimos (`Anyone with the link`) que podrían reenviarse sin control. |
| **Tipo de enlace predeterminado** | `Specific people (only the people the user specifies)` | Evita que por error un enlace interno sea abierto por sectores no autorizados. |
| **Expiración de acceso de invitados** | `60 días` | Obliga a revalidar periódicamente el acceso compartido con estudios contables o despachantes externos. |
| **Descarga en equipos no gestionados** | Vista web solo lectura en equipos no conformes | Reduce exfiltración masiva de bibliotecas completas hacia PCs hogareñas. |

---

## 5. Recuperación ante Borrado Masivo o Ransomware

Si un puesto infectado antes de ser aislado llegara a eliminar o sobrescribir archivos sincronizados en una biblioteca de SharePoint o OneDrive:

1. Aislar inmediatamente el equipo afectado desde **Microsoft Defender Portal (`Isolate device`)** y cerrar sesiones del usuario en Entra ID.
2. Ingresar a la biblioteca afectada con rol administrador → **Configuración (⚙️) > Restaurar esta biblioteca (`Restore this library`)**.
3. Seleccionar el punto en el tiempo exacto previo al incidente (disponible hasta **30 días hacia atrás** con granularidad por operación) para revertir masivamente todas las modificaciones o eliminaciones en minutos.

---

## 6. Control de Cambios

| Versión | Fecha | Autor | Descripción del Cambio |
|---|---|---|---|
| `1.0` | `2026-03-01` | Tomás Torres | Diseño de arquitectura de sitios, migración desde servidor SMB legado, matriz RBAC y políticas de uso compartido. |
