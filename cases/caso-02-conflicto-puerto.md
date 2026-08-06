\# Caso 02: Conflicto de puerto sin error visible



\## Síntoma

Un segundo proceso levantado en el mismo puerto que otro ya en uso no muestra

ningún error, pero no responde a peticiones.



\## Herramientas usadas

\- PowerShell (netstat)

\- Process Explorer (Sysinternals)

\- Python http.server (para provocar el conflicto)



\## Diagnóstico

Levante un servidor HTTP con Python en el puerto 5000 desde una terminal y luego levanté un segundo servidor en el mismo puerto desde una terminal distinta, esperando que el segundo fallara con un error de puerto ocupado. En vez de eso, ambos procesos reportaron estar sirviendo sin ningún error.



Para descartar que se tratara de un error silencioso, usé `netstat -ano | findstr :5000` y confirmé que ambos procesos, con PIDs

distintos, aparecían en estado LISTENING sobre el mismo puerto, tanto en IPv4 como en IPv6 — lo que confirmaba que el sistema sí había permitido el doble bind.



Con Process Explorer identifiqué cada PID como un proceso python.exe independiente. Para determinar cuál de los dos realmente estaba sirviendo tráfico, refresqué el navegador apuntando a localhost:5000 y observé en qué terminal aparecía el log de la petición HTTP — solo la terminal del PID 24532 mostró actividad, confirmando que era el proceso activo y que el PID 7952 estaba en un estado de "escucha inerte".



\## Causa raíz

El sistema operativo permitió el bind de un segundo proceso sobre un puerto ya ocupado, pero solo el primer proceso en hacer bind recibe el trafico real, el segundo queda en estado de listening sin funcionar y sin generar ningún error visible.



\## Resolución

Cerré el proceso duplicado (PID 7952) presionando `Ctrl+C` en la terminal donde estaba corriendo.



\## Verificación

Repetí `netstat -ano | findstr :5000` después de cerrar el proceso. El resultado mostró un único PID (24532) en estado LISTENING, confirmando que el conflicto quedó resuelto.



\## Lección aprendida

\- Un puerto puede aceptar más de un bind sin error visible en Windows, dependiendo

&#x20; de opciones de socket o binding dual-stack.

\- "Sin error" no significa "sin problema" — el proceso zombi puede ser más difícil

&#x20; de detectar que uno que falla explícitamente.

\- netstat + Process Explorer + verificación de logs es la combinación necesaria

&#x20; para diagnosticar esto; ninguna herramienta sola lo hubiera revelado.

