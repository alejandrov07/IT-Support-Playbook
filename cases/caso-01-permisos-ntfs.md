\# Caso 01: Acceso denegado por permiso NTFS explícito



\## Síntoma

Usuario reporta no poder acceder a una carpeta local, con error de acceso denegado.



\## Herramientas usadas

\- PowerShell (New-LocalUser, Get-LocalUser, runas)

\- NTFS Security tab (Windows Explorer)

\- Event Viewer (Security log)



\## Diagnóstico

El usuario reporto no haber podido acceder a una carpeta local con un mensaje de "Acceso Denegado". Para aislar la causa, empecé confirmando el sintoma directamente, intenté abrir la carpeta desde una sesión con permisos limitados usando `runas /user:usuario.prueba explorer.exe`, pero Explorer devolvió un error de shell poco claro. Repetí la prueba usando la consola (`runas /user:usuario.prueba cmd` seguido de `cd`), lo que dio un resultado mucho más claro y citable `Access is denied.`



Antes de asumir que el problema estaba en los permisos, descarté que la cuenta del usuario tuviera algún problema propio (deshabilitada, mal creada) verificando con `Get-LocalUser`, donde confirmé que la cuenta estaba activa (`Enabled: True`).



Revisé el Event Viewer (Security log) filtrando por los Event ID 4656 y 4663, asociados a acceso a objetos, pero no encontré ningún registro. Esto no significa que no haya control de acceso funcionando — significa que la auditoría de "Object Access" no está habilitada por default en Windows, así que aunque el bloqueo se aplica correctamente, no queda rastro en el log hasta configurarla explícitamente.



Como el acceso fue local descarté que los permisos de Share tuvieran algún rol en el bloqueo, el único filtro aplicado fue el de permisos NTFS, verificado directamente en la pestaña Security de la carpeta, donde confirmé un permiso "Deny" explícito asignado al usuario.



\## Causa raíz

Permiso Deny explícito en la ACL NTFS, que tiene precedencia sobre cualquier Allow.



\## Resolución

Eliminé el permiso Deny explícito sobre el usuario en la ACL NTFS de la carpeta y lo sustituí por un Allow de control total, replicando el flujo que seguiría cualquier analista corrigiendo un permiso mal configurado desde la interfaz de Windows (pestaña Security de la carpeta).



\#Verificación

Repetí el acceso como `usuario.prueba` mediante `runas /user:usuario.prueba cmd` y `cd C:\\\\CarpetaProtegida`. El comando se ejecutó sin errores, confirmando que el acceso fue restaurado correctamente.



\## Lección aprendida

\- Deny explícito en NTFS siempre gana sobre Allow, incluso por membresía de grupo.

\- Los permisos de Share solo aplican en acceso por red, no en acceso local.

\- La auditoría de Object Access no está habilitada por default en Windows — sin ella, no hay rastro en Event Viewer aunque el control de acceso funcione correctamente.

