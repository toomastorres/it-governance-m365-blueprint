# IT-PRO-001 — Procedimiento Operativo: Recuperación y Rotación de Clave BitLocker en Intune

| Campo | Detalle |
|---|---|
| **Código Documental** | `IT-PRO-001` |
| **Versión** | `1.0` |
| **Fecha de Vigencia** | `2026-03-01` |
| **Capítulo** | `06 - Instructivos y Procedimientos` |
| **Audiencia** | Exclusivo Administradores IT / Mesa de Ayuda (`Intune Admin` / `Helpdesk Admin`) |
| **Organización (Ejemplo)** | Atlantic Maritime Agency (AMA) — `atlantic-agency.example.com` |
| **Responsable / Autor** | Tomás Torres — Digital Transformation & IT Governance Lead |
| **SLA Objetivo** | `< 15 minutos` en horario operativo |

---

## 1. Propósito y Escenario de Activación

Establecer el procedimiento seguro y auditado para desbloquear una estación de trabajo corporativa Windows que presenta la pantalla azul de **Recuperación de BitLocker (*Enter the recovery key for this drive*)**, garantizando:

1. La **verificación fehaciente de identidad** del solicitante antes de revelar los 48 dígitos.
2. El diagnóstico de la causa raíz que disparó la alerta del chip TPM 2.0.
3. La **rotación obligatoria** de la clave de recuperación en **Microsoft Intune** una vez restablecido el equipo, para que una clave dictada o escrita nunca pueda reutilizarse en el futuro.

### Causas Habituales Legítimas vs. Sospechosas

- **Legítimas frecuentes:** Actualización automática de firmware UEFI/BIOS vía Windows Update, encendido del equipo con un pendrive booteable o dock USB-C nuevo conectado, descarga completa de batería CMOS o múltiples intentos fallidos de PIN pre-arranque.
- **Sospechosas (escalar de inmediato):** Extracción física del disco NVMe hacia otra computadora, modificación no autorizada de opciones *Secure Boot* o solicitud realizada por un canal no verificado.

---

## 2. Prerrequisitos del Administrador

- Iniciar sesión en el navegador con la **cuenta administrativa dedicada** (`adm-<usuario>@atlantic-agency.example.com`) y MFA activo — **nunca** con cuenta de uso cotidiano.
- Contar con rol **`Intune Administrator`**, **`Cloud Device Administrator`** o **`Helpdesk Administrator`** en Microsoft Entra ID.

---

## 3. Paso a Paso de Ejecución

### Paso 1 — Validar Identidad del Usuario y Obtener el `Key ID`

1. Verificar la identidad del colaborador mediante **llamada/videollamada por Microsoft Teams** o presencialmente en oficina (prohibido enviar claves de 48 dígitos por correo abierto o mensajería no verificada sin confirmar voz/rostro).
2. Solicitar al usuario dos datos que aparecen en su pantalla azul de BitLocker:
   - **Nombre del equipo** (etiqueta física, ej. `AMA-NB-012`).
   - **Primeros 8 caracteres del Identificador de Clave (`Recovery Key ID`)**, por ejemplo: `A1B2C3D4` *(correspondiente al inicio del GUID `A1B2C3D4-XXXX-XXXX-XXXX-XXXXXXXXXXXX`)*.

### Paso 2 — Localizar la Clave de 48 Dígitos en Microsoft Intune

1. Ingresar al centro de administración de **Microsoft Intune**: `https://intune.microsoft.com`.
2. Navegar a **Devices (`Dispositivos`) > All devices (`Todos los dispositivos`)**.
3. Buscar y abrir el equipo reportado (ej. `AMA-NB-012`) y validar que el `Primary user` coincida con el colaborador solicitante.
4. En el menú lateral del dispositivo, seleccionar **Monitor > Recovery keys (`Claves de recuperación`)**.
5. Localizar la fila cuyo **`Key ID`** comience exactamente con los **8 caracteres** informados por el usuario en el Paso 1.
6. Hacer clic en **`Show recovery key` (`Mostrar clave de recuperación`)**.
   > *Nota de auditoría:* Esta acción genera automáticamente un registro indeleble en los **Audit Logs de Microsoft Entra ID** (`Action: Read BitLocker key`) indicando qué administrador visualizó la clave y en qué minuto exacto.

### Paso 3 — Ingreso de Clave y Verificación de Arranque Limpio

1. Dictar los **8 bloques de 6 dígitos numéricos** (48 dígitos en total) al usuario o ingresarlos directamente si la asistencia es presencial, y presionar **`Enter`**.
2. Una vez iniciado Windows:
   - Solicitar desconectar periféricos USB de almacenamiento externos no esenciales.
   - Ejecutar un **reinicio normal (`Inicio > Reiniciar`)** y confirmar que el equipo vuelve a iniciar Windows **sin volver a pedir la clave de BitLocker** (confirmando que los registros PCR del chip TPM volvieron a sellarse correctamente).

### Paso 4 — Si en el Reinicio Vuelve a Pedir la Clave (Remediación de Suspend/Resume)

Si tras una actualización de BIOS el sellado del TPM quedó desincronizado y vuelve a solicitar la clave en cada encendido:

1. Conectarse por asistencia remota al equipo e iniciar una terminal **PowerShell como Administrador** (autenticando con credencial `adm-*` / LAPS).
2. Suspender la protección por 1 ciclo de reinicio para re-sellar el perfil PCR del TPM:

```powershell
# Verificar estado actual de protectores en C:
manage-bde -status C:

# Suspender protección durante 1 reinicio y reiniciar para actualizar el sellado TPM
Suspend-BitLocker -MountPoint "C:" -RebootCount 1
Restart-Computer
```

3. Tras el reinicio, ejecutar `manage-bde -status C:` y confirmar que **`Protection Status: Protection On`** haya vuelto a activarse automáticamente.

### Paso 5 — Rotación Obligatoria de la Clave Revelada (Paso Crítico)

Como la clave de 48 dígitos fue verbalizada o visualizada en pantalla, **debe invalidarse y reemplazarse por una nueva clave** custodiada en Entra ID:

1. Con el equipo encendido y conectado a Internet, regresar en `intune.microsoft.com` a la ficha del dispositivo **`AMA-NB-012`**.
2. En la barra superior de acciones remotas (`...`), seleccionar **`BitLocker key rotation` (`Rotación de clave de BitLocker`)** y confirmar.
3. Forzar sincronización en el equipo (`Company Portal > Settings > Sync`) o aguardar ~10 minutos y comprobar en **Monitor > Recovery keys** que aparezca un **nuevo `Key ID`** con fecha actual en Entra ID.

---

## 4. Registro del Incidente

Registrar brevemente el evento en el canal/registro de soporte IT consignando:
- Fecha y hora · Equipo (`AMA-NB-XXX`) · Usuario · Motivo detectado (ej. `BIOS Update 1.18.0`) · Confirmación de rotación completada (`Sí`).

---

## 5. Control de Cambios

| Versión | Fecha | Autor | Descripción del Cambio |
|---|---|---|---|
| `1.0` | `2026-03-01` | Tomás Torres | Emisión del procedimiento operativo estándar con validación de identidad, re-sellado TPM y rotación obligatoria en Intune. |
