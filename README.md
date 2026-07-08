# Restricción de Dispositivos USB con Directivas de Grupo (DLP)

Implementación de un control de **Prevención de Pérdida de Datos (DLP)** sobre Windows 10: bloqueo del acceso a dispositivos de almacenamiento USB para los usuarios estándar, con una **excepción por grupo** para las cuentas que legítimamente necesitan acceso (por ejemplo, IT). Trabajo de la especialización Blue Team / SOC de 4Geeks Academy.

Todo el procedimiento fue ejecutado y verificado sobre una máquina real; las capturas corresponden a esa ejecución.

---

## Objetivo

Cerrar uno de los vectores de exfiltración más comunes —copiar datos a un pendrive— apoyándose en capacidades nativas del sistema operativo, sin software adicional. La restricción bloquea a los usuarios sin privilegios y deja el acceso intacto para los administradores.

## Entorno

| Componente | Detalle |
|---|---|
| Hipervisor | Oracle VirtualBox (host Linux) |
| Sistema invitado | Windows 10 (interfaz en inglés) |
| Complemento | VirtualBox Extension Pack (soporte USB) |
| Herramientas nativas | `gpedit.msc`, `mmc.exe` (MLGPO) |

> El sistema invitado está en inglés, por lo que los nombres de menús y directivas de la documentación se mantienen en inglés, tal como aparecen en pantalla.

## Fases del trabajo

**Fase 1 — Restricción general.** Se habilitan *Deny read/write access* para discos removibles bajo `Computer Configuration`, se crea un usuario estándar (`testuser`) y se valida que el acceso queda denegado.

**Fase 2 — Excepción por grupo (MLGPO).** Se revierte la directiva general y se aplica la restricción únicamente al grupo `Non-Administrators` mediante una directiva local por usuario. Los administradores, al quedar fuera de ese grupo, conservan el acceso.

## El punto clave

Una directiva bajo `Computer Configuration` se aplica a **toda la máquina y a todas las cuentas por igual**, con prioridad sobre cualquier configuración por usuario. Por eso **no admite excepciones**: no se puede abrir un hueco en ella para una sola cuenta.

La solución es invertir el enfoque: en lugar de bloquear a todos y exceptuar a alguien, no se restringe la máquina en general y se restringe **solo al grupo `Non-Administrators`** vía MLGPO. Los administradores, al no pertenecer a ese grupo, quedan sin restricción — y esa ausencia de restricción *es* la excepción.

## Evidencia

**Usuario estándar (`testuser`) — acceso denegado:** el pendrive se monta (`PENDRIVE (E:)` visible en el panel lateral), pero el contenido queda bloqueado por directiva, no por un fallo físico.

![Acceso denegado para el usuario estándar](screenshots/07-testuser-acceso-denegado.png)

**Administrador — acceso permitido:** el mismo dispositivo, en el mismo equipo, abre y muestra su contenido.

![Acceso permitido para el administrador](screenshots/08-admin-acceso-permitido.png)

El mismo pendrive, en el mismo equipo, con dos resultados distintos según la cuenta: eso es lo que demuestra que la excepción por grupo funciona.

## Contenido del repositorio

```
.
├── README.md
├── Informe_Restriccion_USB_DLP.docx     # Informe completo (desarrollo + anexo de comandos)
└── screenshots/
    ├── 01-computer-config-deny-enabled.png
    ├── 02-usuario-estandar-testuser.png
    ├── 03-validacion-restriccion-general.png
    ├── 04-computer-config-revertido.png
    ├── 05-seleccion-non-administrators.png
    ├── 06-non-admin-policy-enabled.png
    ├── 07-testuser-acceso-denegado.png
    └── 08-admin-acceso-permitido.png
```

## Documentación completa

El informe formal, con la fundamentación, el desarrollo paso a paso de ambas fases y un anexo con la secuencia de comandos y rutas para reproducir el ejercicio, está en **[`Informe_Restriccion_USB_DLP.docx`](Informe_Restriccion_USB_DLP.docx)**.

## Autor

**Gonzalo Rodríguez de Mello**
Especialización Blue Team / SOC — 4Geeks Academy
[GitHub](https://github.com/GonzaBot) · [LinkedIn](https://linkedin.com/in/gonzardem)
