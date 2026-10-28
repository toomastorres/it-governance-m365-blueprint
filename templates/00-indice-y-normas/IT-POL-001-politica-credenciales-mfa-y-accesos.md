# IT-POL-001 — Política Corporativa de Credenciales, MFA y Control de Accesos

| Campo | Detalle |
|---|---|
| **Código Documental** | `IT-POL-001` |
| **Versión** | `1.0` |
| **Fecha de Vigencia** | `2026-03-01` |
| **Capítulo** | `00 - Índice y Normas` |
| **Clasificación** | Uso Interno — Cumplimiento Obligatorio |
| **Organización (Ejemplo)** | Atlantic Maritime Agency (AMA) — `atlantic-agency.example.com` |
| **Responsable / Autor** | Tomás Torres — Digital Transformation & IT Governance Lead |
| **Aprobado por** | Dirección General |

---

## 1. Propósito

Proteger las identidades digitales, la información financiera, comercial y operativa, y la infraestructura tecnológica de la organización mediante controles estrictos de autenticación multifactor (MFA), mínimo privilegio, gestión segura de contraseñas y trazabilidad en altas, modificaciones y bajas de personal.

## 2. Alcance

De cumplimiento obligatorio para todos los colaboradores, directivos, pasantes, consultores externos y proveedores con acceso a cuentas `@atlantic-agency.example.com`, equipos corporativos, red interna, VPN o repositorios documentales.

---

## 3. Principios Rectores de Identidad y Acceso

1. **Identidad Única e Intransferible:** Cada persona física debe operar con su propia cuenta nominal (`nombre.apellido@atlantic-agency.example.com`). Queda **estrictamente prohibido** compartir contraseñas o utilizar cuentas genéricas humanas. Cuando un sector requiera atención grupal de correo (`operaciones@`, `pagos@`, `administracion@`), IT aprovisionará un **Buzón Compartido** delegado sobre las cuentas nominales autorizadas.
2. **Mínimo Privilegio (*Least Privilege*):** Los permisos sobre sitios de SharePoint Online, sistemas contables/ERP, carpetas y VPN se otorgan únicamente en función de las responsabilidades actuales del puesto, mediante grupos de seguridad en **Microsoft Entra ID**.
3. **Separación de Privilegios Administrativos:** Los administradores de sistemas deben poseer una cuenta estándar para sus tareas diarias y una cuenta separada con prefijo `adm-` exclusiva para tareas de administración técnica. Ningún usuario estándar posee permisos de Administrador Local sobre su estación de trabajo Windows.

---

## 4. Estándar de Contraseñas y Autenticación Multifactor (MFA)

### 4.1 Requisitos de Contraseña (Alineados a NIST SP 800-63B / Microsoft)

| Tipo de Cuenta | Longitud Mínima | Complejidad y Protección | Expiración |
|---|---|---|---|
| **Usuarios Estándar (M365 / VPN)** | **14 caracteres** (recomendado: frase de paso) | Protección activa de *Entra Password Protection* contra claves filtradas o predecibles. | Sin vencimiento periódico forzado salvo sospecha o evidencia de compromiso. |
| **Cuentas Administrativas (`adm-*`)** | **20 caracteres** aleatorios | Generada y almacenada en `KeePassXC`. Sesión no persistente. | Rotación semestral o inmediata ante cambio en el equipo IT. |
| **Cuenta *Break-Glass* e Infraestructura Crítica** | **32+ caracteres** aleatorios | Resguardo dual (bóveda `Tier 0` + sobre lacrado de Dirección) y monitoreo de uso. | Rotación inmediata posterior a cualquier uso o simulacro. |

### 4.2 Uso Obligatorio de MFA

- **100% de cobertura:** Todo inicio de sesión en Microsoft 365, portales corporativos y VPN requiere **Autenticación Multifactor (MFA)**.
- **Método homologado:** Aplicación **Microsoft Authenticator** configurada con **coincidencia de números (*Number Matching*)** o llaves físicas compatibles FIDO2 / Windows Hello for Business.
- **Anti-Fatiga de MFA:** Ante una notificación inesperada de aprobación de ingreso en el teléfono móvil que el usuario no haya iniciado, el colaborador **debe rechazar la solicitud (`No soy yo`) e informar de inmediato a IT**.

---

## 5. Resguardo de Secretos y Uso de Bóvedas Cifradas

1. **Prohibición de Texto Plano:** Queda terminantemente prohibido guardar contraseñas corporativas, claves Wi-Fi de administración, accesos bancarios/fiscales o secretos técnicos en planillas Excel, documentos Word, correos electrónicos, chats o notas adhesivas físicas.
2. **Bóveda Oficial de IT (`KeePassXC`):** Todos los secretos de infraestructura (Firewall, hipervisor ESXi, Veeam Backup, switches, DNS, proveedores) se custodian en archivos `.kdbx` cifrados con `ChaCha20` y derivación `Argon2id`:
   - `AMA-Tier0-Infraestructura.kdbx`: Crítico — solo IT Lead y Dirección.
   - `AMA-Tier1-OperacionesIT.kdbx`: Operativo — soporte técnico autorizado.

---

## 6. Ciclo de Vida de Accesos (Altas, Cambios y Bajas — *JML*)

### 6.1 Alta de Personal (*Joiner*)

- Recursos Humanos / Administración notifica a IT con un mínimo de **48 horas hábiles** indicando nombre completo, área, puesto, líder directo, buzones compartidos requeridos y necesidad de acceso VPN.
- IT crea la identidad en Entra ID, asigna licencia **Microsoft 365 Business Premium**, incorpora al usuario en los grupos `SG-*` correspondientes, prepara equipo enrolado en **Intune** con **BitLocker** activo y entrega los instructivos `IT-INS-004` y `IT-INS-001`.

### 6.2 Cambio de Rol (*Mover*)

- Ante un cambio de sector, se **revocan primero** los grupos de acceso del área anterior antes de otorgar los nuevos permisos, evitando acumulación histórica de privilegios (*Privilege Creep*).

### 6.3 Desvinculación o Baja (*Leaver*) — Ejecución en < 15 Minutos

Al momento efectivo comunicado por Dirección / RR.HH., IT ejecuta la lista de corte inmediato:

1. **Bloquear inicio de sesión** en Microsoft Entra ID y ejecutar **Revoke Sessions** (cierre forzado de tokens activos en PC, web y celular).
2. **Revocar acceso VPN** removiendo al usuario del grupo `SG-VPN-Usuarios-Remotos` / deshabilitando perfil en Firewall.
3. **Convertir buzón de usuario en Buzón Compartido (*Shared Mailbox*)** para preservar el historial comercial/operativo y delegar lectura o respuesta automática al responsable del área, liberando posteriormente la licencia M365.
4. **Resguardar OneDrive** transfiriendo propiedad temporal de archivos laborales al gerente del área por el plazo definido (30–90 días).
5. **Recuperar y auditar el equipo físico** (o ejecutar *Selective Wipe / Retire* desde Intune en dispositivos móviles).
6. Si la persona desvinculada tenía acceso a secretos técnicos o portales externos compartidos, **rotar de inmediato** las credenciales involucradas en `KeePassXC`.

---

## 7. Bloqueo de Equipos y Trabajo Remoto

- Todo colaborador debe bloquear su sesión (`Win + L`) al ausentarse de su puesto; Intune fuerza bloqueo automático de pantalla tras **5 minutos de inactividad**.
- El acceso remoto a recursos internos de red local se realiza exclusivamente mediante la **VPN oficial** (`IT-INS-001`) desde equipos corporativos administrados y conformes en Intune.

---

## 8. Control de Cambios

| Versión | Fecha | Autor | Descripción del Cambio |
|---|---|---|---|
| `1.0` | `2026-03-01` | Tomás Torres | Aprobación inicial de la política de credenciales, MFA, bóvedas KeePassXC y protocolo JML. |
