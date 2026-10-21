# HOSPIT-IT — Plan maestro del proyecto

**Proyecto:** Plataforma integral de gestión IT para el hospital ficticio *Hospital Sierra Salud*  
**Documento:** Plan maestro, arquitectura, hoja de ruta, checklist y criterios de aceptación  
**Versión:** 1.0 · **Fecha:** 21 de octubre de 2026  
**Objetivo principal:** tener una única referencia para planificar, construir, probar, documentar y mantener el proyecto sin olvidar requisitos.

> **Regla de alcance:** HOSPIT-IT es un laboratorio educativo. Se utilizarán datos, usuarios, pedidos y activos ficticios. No se conectará a una red hospitalaria real ni se introducirán datos de pacientes.

## Índice de contenido

1. Resumen ejecutivo
2. Principios de diseño y límites
3. Arquitectura objetivo
4. Herramientas elegidas
5. Inventario de funciones y dónde se implementan
6. Infraestructura y requisitos orientativos
7. Plan de red de laboratorio
8. Identidad, roles y permisos
9. Datos y fuentes de verdad
10. Estructura del repositorio Git
11. Seguridad mínima obligatoria
12. Copias, continuidad y recuperación
13. Monitorización y gestión de alertas
14. Automatización y gestión de cambios
15. Hoja de ruta por fases
16. Priorización: no intentar hacerlo todo a la vez
17. Batería de pruebas de aceptación
18. Operación y mantenimiento recurrentes
19. Gestión de incidentes y errores
20. Riesgos principales y mitigaciones
21. Licencias, costes y sostenibilidad
22. Estructura recomendada de la memoria ASIR
23. Checklist global de seguimiento
24. Primeras diez acciones concretas
25. Fuentes oficiales para consultar antes de instalar

\newpage

## 1. Resumen ejecutivo

HOSPIT-IT no se desarrollará desde cero como una aplicación monolítica. Será una solución compuesta por herramientas de código abierto, configuradas y conectadas de forma controlada. La mayor parte del trabajo consistirá en infraestructura, configuración, seguridad, automatización, pruebas y documentación; el código propio se limitará a pequeñas integraciones que aporten valor demostrable.

El núcleo de gestión IT será **GLPI**. **Zabbix** supervisará servidores, servicios y disponibilidad. **Ansible** y **PowerShell** automatizarán tareas de sistemas. **OPNsense** proporcionará firewall y segmentación de red. **Samba AD** se incorporará si se necesita un dominio Windows/Linux de laboratorio. **PrestaShop autohospedado** será una tienda electrónica de pruebas, separada funcionalmente de GLPI. Las integraciones usarán API/webhooks cuando resulte viable y se implementarán después de que ambos sistemas funcionen por separado.

### Resultado final esperado

- Portal de autoservicio IT con usuarios, perfiles, tickets, solicitudes y base de conocimiento.
- Inventario trazable de equipos, periféricos, software, licencias, contratos y asignaciones.
- Avisos de vencimiento y tareas de mantenimiento documentadas.
- Monitorización con alertas por correo y, opcionalmente, Discord; los eventos seleccionados podrán generar tickets en GLPI.
- Administración controlada de usuarios y equipos; automatización reproducible con Ansible y PowerShell.
- Red de laboratorio segmentada, con reglas explícitas entre zonas.
- Tienda de pruebas con catálogo, SKU, stock, clientes ficticios, carrito, pedidos, cupones y reseñas mediante una función o módulo compatible. El checkout será simulado o usará únicamente un entorno sandbox; no se procesarán cobros reales.
- Copias de seguridad, prueba de restauración, procedimiento de actualización/rollback, repositorio Git y documentación reproducible.
- Demostración final con evidencias: capturas, registros, diagramas, pruebas y manuales.

## 2. Principios de diseño y límites

1. **Código abierto primero:** priorizar ediciones autohospedadas, repositorios oficiales y licencias verificables.
2. **No reinventar lo que ya existe:** GLPI y PrestaShop aportarán las funciones principales; el código propio será pequeño y estará aislado.
3. **Un sistema, una fuente de verdad:** no duplicar todos los datos de GLPI y PrestaShop. Cada aplicación será autoridad de sus datos propios.
4. **Construir por capas:** primero funcionamiento individual, luego seguridad, después integraciones.
5. **Cambios reversibles:** backup probado antes de actualizaciones importantes; pruebas en staging cuando sea viable; rollback documentado.
6. **Privilegios mínimos:** cada cuenta y servicio tendrá solo los permisos necesarios.
7. **Nada de secretos en Git:** contraseñas, tokens, claves privadas y ficheros de configuración sensibles quedarán fuera del repositorio o cifrados.
8. **No exponer servicios innecesariamente a Internet:** el laboratorio deberá ser privado; cualquier publicación posterior exige revisión específica.
9. **Compatibilidad explícita:** anotar versiones compatibles de aplicación, PHP, base de datos, agentes, plugins y sistema operativo. No actualizar componentes aislados sin revisar la matriz de compatibilidad.
10. **No prometer operación clínica real:** el proyecto no es una plataforma validada para datos de pacientes ni para operaciones asistenciales reales.

### Fuera del alcance inicial

- Integración con historias clínicas, datos de pacientes o sistemas hospitalarios reales.
- Pagos reales, tratamiento real de tarjetas o datos personales de clientes reales.
- Despliegue automático e indiscriminado de actualizaciones de producción.
- Una única aplicación hecha a medida que reimplemente GLPI o PrestaShop.
- Automatizaciones destructivas sin confirmación, registro, pruebas y posibilidad de recuperación.

## 3. Arquitectura objetivo

```text
Usuarios ficticios -> DNS interno / HTTPS
  -> GLPI: portal IT, tickets, activos, solicitudes y conocimiento
  -> PrestaShop: catálogo, stock, carrito y pedidos ficticios

Zabbix -> alertas -> correo / Discord -> GLPI (eventos seleccionados)
PrestaShop <-> API/webhooks <-> integración mínima <-> GLPI
Ansible / PowerShell -> Linux y Windows de laboratorio
Samba AD (opcional) -> usuarios, grupos y políticas de dominio
OPNsense -> firewall, VLAN y reglas entre segmentos
Hipervisor -> máquinas virtuales | backups separados | Git -> código y docs
```

El diagrama expresa la arquitectura prevista, no una integración que venga activada de serie. Cada enlace entre aplicaciones necesita configuración, credenciales de mínimo privilegio y pruebas de extremo a extremo.

## 4. Herramientas elegidas

| Componente | Estado | Responsabilidad | Decisión práctica |
|---|---|---|---|
| Hipervisor (Proxmox VE u otro que ya se domine) | Recomendado | Ejecutar máquinas virtuales y redes de laboratorio | Elegir según hardware y experiencia; no dedicar la fase inicial a comparar hipervisores durante semanas. |
| OPNsense | Núcleo de red | Firewall, gateway, DHCP si procede y VLAN | Mantener la administración del firewall solo desde la red de administración. |
| GLPI autohospedado | Núcleo IT | Tickets, usuarios, inventario, activos, contratos/licencias y solicitudes | Instalar el núcleo open source; no depender de un servicio alojado de pago. Confirmar qué funciones requieren plugins. |
| GLPI Agent | Recomendado | Recoger inventario de equipos compatibles | Probar primero en dos o tres máquinas de laboratorio. |
| Zabbix | Núcleo de monitorización | Disponibilidad, métricas, triggers, alertas y monitorización web | Empezar por pocos indicadores útiles y ajustar umbrales. |
| Ansible | Núcleo de automatización | Configurar Linux, desplegar aplicaciones y ejecutar tareas repetibles | Usar inventarios y playbooks versionados; comenzar en modo comprobación cuando proceda. |
| PowerShell | Necesario para tareas Windows | Administración y configuración de Windows | Versionar scripts, registrar resultados y evitar credenciales embebidas. |
| PrestaShop Open Source autohospedado | Núcleo ecommerce | Catálogo, stock, clientes ficticios, carrito y pedidos | Usar modo catálogo/checkout de prueba; verificar módulos antes de añadirlos. |
| Git (GitHub, GitLab o servidor Git propio) | Núcleo de trabajo | Historial, colaboración, revisiones y versiones | Repositorio privado durante el desarrollo; publicar solo lo que no contenga secretos ni datos sensibles. |
| Copias (Restic, BorgBackup o herramienta equivalente) | Núcleo de resiliencia | Backups cifrados y recuperación | La copia debe estar separada del servidor que protege. Probar restauración. |
| Samba AD | Condicional | Dominio e identidad compatible con un laboratorio Windows | Añadir solo si el proyecto necesita demostrar dominio, grupos y políticas. Validar escenarios concretos de Windows. |
| NetBox | Opcional | IPAM, VLAN, racks y documentación de red | No instalar si una tabla controlada y los diagramas cubren la necesidad del proyecto. |
| FOG Project o Clonezilla | Opcional | Imagen y despliegue de sistemas operativos | Solo si se va a demostrar instalación de equipos nuevos por imagen/PXE. |
| Caddy o proxy inverso equivalente | Opcional | Terminación HTTPS y nombres internos | No es un requisito para completar el MVP si cada aplicación ya está correctamente publicada dentro del laboratorio. |
| Wazuh u otra plataforma de seguridad | Fase futura | Recolección de logs y detección adicional | No incluir en el MVP; añade carga y complejidad. Zabbix cubre la monitorización inicial. |

**Base de datos de GLPI:** usar una versión de MySQL o MariaDB compatible con la versión concreta de GLPI instalada; no forzar PostgreSQL. Revisar siempre los requisitos oficiales antes de instalar. Para PrestaShop, respetar la base de datos y versiones admitidas por la edición elegida.

## 5. Inventario de funciones y dónde se implementan

| Necesidad | Solución base | Requisitos, límites o notas |
|---|---|---|
| Portal de empleados | GLPI | Configurar perfiles, entidades, categorías y permisos. |
| Incidencias y peticiones | GLPI | Categorías, prioridad, asignación, estados, fechas y notificaciones. |
| Base de conocimiento | GLPI | Procedimientos reutilizables, preguntas frecuentes y guías internas. |
| Ordenadores, servidores y periféricos | GLPI + GLPI Agent | Importar/descubrir lo que permita el entorno y corregir los datos manualmente. |
| SKU, stock, movimientos y responsable | GLPI para activos IT; PrestaShop para stock de tienda | Definir qué aplicación es autoridad del stock de cada tipo de artículo. No mantener dos stocks independientes para el mismo inventario. |
| Licencias, garantías y contratos | GLPI | Registrar titular, proveedor, vencimiento, cobertura y responsable. Probar las alertas reales de la versión instalada. |
| Certificados y mantenimiento | GLPI + procedimiento/recordatorio | Crear elementos y reglas de aviso; confirmar si la edición/plugin dispone de objeto específico. No asumir automatismo sin probarlo. |
| Lotes o caducidad física de productos | Posible plugin o desarrollo pequeño | Fuera del MVP salvo que sea requisito académico explícito. Diseñar lote, fecha de caducidad, cantidad y responsable. |
| Reservas de activos | GLPI, si la funcionalidad aplica en la versión | Validar flujo con usuarios reales de prueba y permisos. |
| Formularios avanzados | GLPI más plugin compatible si hace falta | Verificar compatibilidad y licencia del plugin. Formcreator es una opción histórica; no instalarlo sin validar la versión actual. |
| Aprobaciones | Flujo nativo o plugin, según caso | Documentar quién solicita, quién aprueba y qué evidencia queda. |
| Encuestas de satisfacción | Funciones disponibles en GLPI o módulo compatible | Probar envío, respuesta y reporte; tratarlo como requisito con criterio de aceptación. |
| Alertas de servidores y servicios | Zabbix | CPU/RAM/disco, proceso, disponibilidad, HTTPS, base de datos, sitio web y capacidad de backup. |
| Tickets automáticos a partir de alertas | Zabbix + API/webhook hacia GLPI | Integración explícita; aplicar deduplicación, recuperación y credenciales de mínimo privilegio. |
| Correo/Discord | Zabbix y/o GLPI | Separar alertas urgentes de avisos informativos para evitar ruido. No publicar información sensible en Discord. |
| Alta de usuario | Samba AD opcional + procedimiento y automatización | Cuenta, grupo, permisos, equipo asignado y registro de inventario. Evitar crear permisos excesivos por defecto. |
| Políticas y software en equipos Windows | GPO si existe dominio + PowerShell/Ansible según compatibilidad | Instalar y comprobar primero en máquinas de prueba. No asumir que un portal web controla un equipo sin agente o mecanismo autorizado. |
| Configuración de Linux | Ansible | Playbooks idempotentes, variables por entorno y pruebas. |
| Instalación de sistemas operativos | FOG/Clonezilla opcional | Solo para equipos nuevos o reinstalaciones planificadas; probar drivers y método de recuperación. |
| Tienda de prueba | PrestaShop | Catálogo, SKU, imágenes propias/licenciadas, precio ficticio, stock y pedidos de prueba. |
| Reseñas y cupones | PrestaShop o módulo compatible | Verificar funciones disponibles en la versión instalada; algunos módulos pueden ser de pago. |
| Checkout | PrestaShop con pago simulado o sandbox | Nunca introducir tarjetas reales ni activar cobros reales para la demostración. |
| Integración tienda–IT | API/webhook + pequeño servicio de integración | Intercambiar solo datos necesarios; usar identificadores de pedido/SKU ficticios. |
| Actualización y rollback | Ansible + backups + procedimiento aprobado | Automatizar pasos seguros, no actualización ciega de todas las aplicaciones a la última versión. |

## 6. Infraestructura y requisitos orientativos

Los recursos reales dependen de cuántas máquinas se ejecuten al mismo tiempo. Para un laboratorio compacto que aloje varias aplicaciones, **32 GB de RAM y SSD de 1 TB** son un punto de partida recomendable; **16 GB** puede servir para una demo reducida si se limitan las máquinas encendidas. Conviene disponer de CPU con virtualización habilitada y, preferentemente, almacenamiento de backup separado. Es una estimación de planificación, no un mínimo oficial de cada producto.

### Diseño de máquinas virtuales de referencia

| VM/servicio | Recursos iniciales orientativos | Notas |
|---|---:|---|
| Firewall OPNsense | 2 vCPU / 2 GB RAM | Ajustar interfaces virtuales y comprobar que el hipervisor/switch virtual respeta las VLAN. |
| GLPI + servidor web | 2 vCPU / 4 GB RAM | Para el MVP puede convivir con la base de datos compatible en la misma VM. Separar base de datos si la carga crece. |
| Zabbix | 2 vCPU / 4 GB RAM | El uso de RAM y almacenamiento crece con los hosts, métricas e historial. |
| PrestaShop | 2 vCPU / 4 GB RAM | VM independiente de GLPI para aislar actualizaciones y datos. |
| Samba AD (opcional) | 2 vCPU / 2–4 GB RAM | Dominio ficticio; usar DNS del dominio correctamente. |
| Nodo de automatización (opcional) | 2 vCPU / 2 GB RAM | Puede ser la propia VM de administración en el MVP. |
| Cliente Windows y cliente Linux | 2 vCPU / 2–4 GB RAM cada uno | Solo encender durante las pruebas si la memoria es limitada. |
| Backup | según volumen | Preferir un destino externo/separado del disco de las VMs. |

### Principio de dimensionamiento

- No encender todas las VMs por defecto si el equipo no tiene suficiente memoria.
- No ubicar el único backup en el mismo disco físico que las VMs.
- Documentar CPU, RAM, almacenamiento, redes, IP, sistema operativo y propietario de cada VM.
- Si solo hay un equipo, declarar que es un laboratorio de nodo único sin alta disponibilidad. No afirmar que tolera la avería física del host.

## 7. Plan de red de laboratorio

Usar segmentos lógicos para practicar aislamiento. Los siguientes rangos son **un ejemplo**, no una configuración que deba aplicarse en una red existente sin comprobar conflictos. Mantener el laboratorio aislado de redes reales.

| VLAN | Nombre | Subred de ejemplo | Propósito |
|---:|---|---|---|
| 10 | Administración | `10.10.10.0/24` | Gestión del hipervisor, firewall y consolas administrativas. |
| 20 | Clientes de prueba | `10.10.20.0/24` | PCs ficticios y pruebas de acceso al portal. |
| 30 | Servidores | `10.10.30.0/24` | GLPI, PrestaShop, Zabbix y servicios internos. |
| 40 | Gestión IT | `10.10.40.0/24` | Estaciones de técnicos y automatización. |
| 50 | Impresoras/IoT de prueba | `10.10.50.0/24` | Periféricos simulados o máquinas de prueba equivalentes. |
| 60 | Backup | `10.10.60.0/24` | Tráfico de copias hacia el destino separado, cuando corresponda. |

### Reglas de red de referencia

- Por defecto, denegar tráfico entre VLAN salvo excepciones documentadas.
- Permitir administración del hipervisor, firewall y servidores solo desde la VLAN de administración/IT.
- Permitir acceso HTTPS a GLPI y PrestaShop desde los clientes que lo necesiten.
- Permitir desde Zabbix solo los puertos y agentes necesarios para monitorización.
- Restringir backup a los orígenes/destinos que lo requieren; no abrir la VLAN completa a usuarios.
- Limitar el acceso a SSH/WinRM y otros canales de administración a la estación/nodo de automatización autorizado.
- No publicar interfaces de administración directamente en Internet.
- Probar las reglas con casos permitidos y denegados y guardar capturas o exportación de configuración.

## 8. Identidad, roles y permisos

Definir roles antes de crear cuentas para no terminar con permisos improvisados.

| Rol | Puede hacer | No debe poder hacer por defecto |
|---|---|---|
| Empleado | Crear y consultar sus tickets; consultar artículos internos y sus solicitudes | Ver tickets de otros usuarios, administrar activos o modificar reglas globales. |
| Técnico IT | Atender tickets asignados, actualizar inventario y documentar intervenciones | Cambiar firewall/dominio o conceder privilegios globales sin autorización. |
| Responsable/aprobador | Aprobar solicitudes concretas y consultar reportes autorizados | Acceder a credenciales o modificar todo el sistema por defecto. |
| Administrador de plataforma | Mantener GLPI/PrestaShop/Zabbix dentro de su alcance | Compartir la cuenta con otros administradores o usarla para el trabajo diario. |
| Administrador de red/dominio | Administrar OPNsense o Samba AD según responsabilidad | Usar esa cuenta como cuenta cotidiana o exponerla al portal. |
| Cuenta de integración | Ejecutar una tarea API concreta | Iniciar sesión interactiva o tener permisos de superadministrador si no es imprescindible. |

Si se incorpora Samba AD, definir nombres, OU/grupos, convención de usuarios, DNS, proceso de alta/baja y política de contraseñas. Las bajas deben incluir revocación de sesiones/credenciales y reasignación de activos. Mantener una cuenta de emergencia protegida y documentada, sin compartirla.

## 9. Datos y fuentes de verdad

| Dato | Sistema autoritativo | Sincronización permitida |
|---|---|---|
| Tickets, historial, comentarios y soluciones | GLPI | Resumen/ID de la alerta Zabbix, cuando exista relación. |
| Activos informáticos, asignación y soporte | GLPI | Enlaces a SKU o referencia de pedido si ayudan a la trazabilidad. |
| Stock disponible para venta de prueba y pedidos | PrestaShop | Notificación de pedido o evento mínimo hacia la integración. |
| Usuarios de dominio | Samba AD, si se instala | Autenticación/directorio en GLPI solo tras configurar y probar LDAP/LDAPS. |
| Estado de monitorización y métricas | Zabbix | Evento resumido a GLPI/Discord/correo según política. |
| Redes, VLAN, IP, conexiones y esquema | NetBox si se instala; si no, repositorio documental | Diagramas y tablas versionadas. |
| Automatización, procedimientos y configuración no secreta | Git | No copiar contraseñas/tokens al repositorio. |

### Flujo de ejemplo: pedido ficticio de un ordenador

1. Un usuario de pruebas tramita un pedido de un SKU ficticio en PrestaShop.
2. PrestaShop mantiene el pedido y el stock de tienda como fuente de verdad.
3. Una integración validada recibe el evento o consulta la API de forma programada.
4. Si procede, crea en GLPI una solicitud interna con referencia del pedido y SKU, no con datos innecesarios del cliente.
5. El técnico gestiona preparación/asignación. Cuando el equipo existe, se registra como activo en GLPI.
6. El estado se refleja en la documentación del proyecto. No se modifica stock en dos sistemas a la vez salvo que exista una regla de sincronización explícita, probada y documentada.

## 10. Estructura del repositorio Git

```text
hospit-it/
├── README.md
├── docs/
│   ├── plan-maestro.md
│   ├── arquitectura.md
│   ├── inventario-vms.md
│   ├── red-vlans-firewall.md
│   ├── seguridad.md
│   ├── copias-y-restauracion.md
│   ├── actualizaciones-y-rollback.md
│   ├── procedimientos/
│   ├── pruebas/
│   └── decisiones/              # ADR: decisiones y motivos
├── diagrams/
├── ansible/
│   ├── inventories/             # Sin secretos ni contraseñas en claro
│   ├── group_vars/
│   ├── playbooks/
│   └── roles/
├── powershell/
├── integrations/
│   ├── glpi-zabbix/
│   └── prestashop-glpi/
├── configs-examples/            # Ejemplos saneados, no exportaciones secretas
├── tests/
└── CHANGELOG.md
```

### Reglas del repositorio

- Rama principal protegida en la medida que permita la plataforma; cambios mediante ramas y revisión.
- Mensajes de commit descriptivos; etiquetar hitos demostrables.
- Añadir `README` para instalación, arranque, pruebas y recuperación.
- Mantener `.env`, credenciales, tokens, claves SSH privadas, copias de base de datos y exportaciones sensibles fuera de Git.
- Si hace falta guardar variables secretas para Ansible, cifrarlas con Ansible Vault y no subir la contraseña del vault.
- Guardar versiones, hashes o procedencia de instaladores y módulos, además de licencias.
- Registrar decisiones importantes en `docs/decisiones/`: fecha, contexto, opciones, decisión y consecuencias.

## 11. Seguridad mínima obligatoria

- [ ] Laboratorio aislado, direcciones no conflictivas y sin datos reales.
- [ ] Aplicaciones accesibles únicamente desde segmentos requeridos.
- [ ] HTTPS para accesos web cuando proceda; certificado y renovación documentados.
- [ ] Cuentas individuales, roles mínimos y MFA cuando la aplicación/entorno lo soporte.
- [ ] Contraseñas únicas y fuertes; secretos fuera del repositorio.
- [ ] Cuentas API dedicadas por integración, token revocable y permisos mínimos.
- [ ] Sistema operativo y aplicaciones con soporte vigente; paquetes y plugins de procedencia fiable.
- [ ] Puertos y servicios innecesarios deshabilitados; administración no expuesta públicamente.
- [ ] Logs con fecha/hora coherentes, permisos restrictivos y retención definida.
- [ ] Backups cifrados cuando contengan datos sensibles de laboratorio y acceso separado.
- [ ] Restauración probada; un backup que nunca se ha restaurado no cuenta como recuperación validada.
- [ ] Procedimiento de parcheo y rollback; actualización de plugins solo tras verificar compatibilidad.
- [ ] Revisar licencias de módulos, temas, imágenes y código de terceros.
- [ ] Evitar enviar nombres personales, tokens o datos técnicos sensibles a Discord.
- [ ] Documentar quién aprobó los cambios que puedan bloquear acceso, borrar datos o afectar al dominio.

## 12. Copias, continuidad y recuperación

### Política de partida para el laboratorio

- Copia diaria de configuraciones y datos importantes.
- Retención inicial de 7 copias diarias y 4 semanales, si el almacenamiento lo permite.
- Copia separada del host o, al menos, en un disco/servidor distinto; una copia offline adicional es preferible.
- Copias de GLPI (base de datos y ficheros relevantes), PrestaShop (base de datos, imágenes, módulos/configuración necesarios), configuración de Zabbix, OPNsense, Samba AD si se instala, repositorio y documentación.
- Las instantáneas de la VM ayudan, pero no sustituyen un backup recuperable y separado.

### Procedimiento de restauración

1. Abrir una incidencia/registro de prueba de recuperación.
2. Elegir una copia conocida y verificar fecha/integridad.
3. Restaurar en una VM o red aislada, nunca encima del único sistema funcional durante la primera prueba.
4. Verificar inicio de sesión, datos de prueba, archivos, conectividad y logs.
5. Registrar duración, errores, versión y acciones correctivas.
6. Actualizar el procedimiento si algo no coincide.

**Objetivo interno sugerido para el ejercicio:** RPO máximo de 24 horas y RTO de 4 horas en una restauración de laboratorio. Son metas didácticas, no garantías de un sistema productivo. Revisarlas según recursos y alcance.

## 13. Monitorización y gestión de alertas

### Elementos que debe vigilar Zabbix

- Disponibilidad ICMP/TCP de las VMs relevantes.
- Estado de servicios de GLPI, PrestaShop, base de datos, web server y DNS cuando proceda.
- URL HTTP/HTTPS: respuesta, código esperado y certificado TLS.
- CPU, memoria, espacio en disco y crecimiento del almacenamiento.
- Uso y resultado de los trabajos de backup (incluyendo aviso por ausencia de copia reciente).
- Eventos de sistema relevantes y estado del agente, donde corresponda.
- Pruebas de tienda de forma sintética sin pagos reales ni datos personales.

### Matriz inicial de alertas

| Severidad | Ejemplo | Acción |
|---|---|---|
| Alta | Servicio de portal caído; disco crítico; backup fallido repetidamente | Aviso inmediato por correo; Discord solo si no hay información sensible; crear/actualizar ticket GLPI en alertas seleccionadas. |
| Media | Espacio por debajo del umbral preventivo; certificado próximo a vencer | Aviso a técnico y tarea planificada. |
| Baja | Reinicio programado o cambio normal | Registro o resumen, sin generar ruido innecesario. |

### Reglas anti-ruido

- Deduplicar por host, servicio y condición.
- Mantener un ticket abierto mientras el incidente esté activo en lugar de crear uno cada minuto.
- Cerrar/resolver el evento o añadir comentario de recuperación cuando Zabbix detecte vuelta al estado normal.
- Documentar umbral, destinatarios, horario y escalado de cada alerta.
- Probar la integración sin provocar indisponibilidad en el host real: usar un servicio de laboratorio o un trigger de prueba controlado.

## 14. Automatización y gestión de cambios

### Casos adecuados para automatizar primero

1. Comprobación de espacio en disco, servicios y versiones.
2. Configuración base de una VM Linux.
3. Instalación repetible de paquetes aprobados.
4. Exportación de configuración y verificación de copias.
5. Comprobaciones pre y post actualización.
6. Tareas de Windows no destructivas mediante PowerShell.
7. Aprovisionamiento de una cuenta de laboratorio con aprobación y registro.

### Flujo obligatorio de actualización

1. Identificar qué se actualiza, por qué y qué versiones son compatibles.
2. Leer notas de versión, avisos de seguridad y requisitos de plugins/dependencias.
3. Hacer backup de base de datos, archivos y configuración; comprobar que termina correctamente.
4. Probar restauración o, como mínimo para un cambio pequeño, validar el mecanismo de recuperación.
5. Aplicar en staging o clon si el recurso existe.
6. Ejecutar la actualización con ventana de mantenimiento, registro y acceso administrativo disponible.
7. Realizar comprobaciones funcionales y de seguridad posteriores.
8. Si falla la validación, detener el cambio y ejecutar rollback siguiendo el procedimiento probado.
9. Documentar resultado y actualizar la matriz de versiones.

Los parches de seguridad del sistema operativo pueden automatizarse con controles adecuados, pero las actualizaciones mayores de GLPI, PrestaShop, Zabbix, base de datos o plugins no deben aplicarse a ciegas. No lanzar un único playbook que actualice todo el entorno sin etapas ni controles.

## 15. Hoja de ruta por fases

Cada fase termina con un **hito verificable**. No empezar integraciones avanzadas mientras sus sistemas base no sean estables.

### Fase 0 — Definición y control del alcance

**Tareas**
- [ ] Crear repositorio Git y este plan como documento principal.
- [ ] Confirmar hardware, hipervisor, memoria disponible y espacio de backup.
- [ ] Hacer lista priorizada de requisitos Must/Should/Could.
- [ ] Dibujar arquitectura, inventario de VMs y diagrama de red.
- [ ] Crear registro de riesgos, decisiones y versiones por validar.

**Hito:** alcance aprobado y despliegue inicial descrito en `README.md`.

### Fase 1 — Virtualización y red base

**Tareas**
- [ ] Instalar/configurar hipervisor.
- [ ] Crear redes virtuales y la VM de OPNsense.
- [ ] Definir VLAN/subredes, gateways, DNS y DHCP según diseño.
- [ ] Crear reglas de firewall con denegación por defecto entre segmentos.
- [ ] Probar conectividad permitida y bloqueada.
- [ ] Documentar cómo recuperar el acceso administrativo si una regla corta la conexión.

**Hito:** VMs separadas por redes y políticas probadas; no se expone el laboratorio al exterior.

### Fase 2 — Administración, Git y copias

**Tareas**
- [ ] Crear una VM/estación administrativa o definir acceso dedicado.
- [ ] Configurar SSH/WinRM solo cuando sea necesario y limitar origen.
- [ ] Crear política de cuentas, secretos y registro de cambios.
- [ ] Crear destino de backup separado.
- [ ] Ejecutar y restaurar una copia de una VM o servicio sencillo.

**Hito:** se puede recuperar al menos un servicio de una copia verificada.

### Fase 3 — GLPI: núcleo de gestión IT

**Tareas**
- [ ] Instalar GLPI según requisitos oficiales y base de datos compatible.
- [ ] Configurar URL, HTTPS, correo, zona horaria, tareas programadas y permisos.
- [ ] Crear entidades, grupos, perfiles, categorías de tickets, prioridades y estados.
- [ ] Crear usuarios y equipos ficticios.
- [ ] Registrar activos: PC, servidor, monitor, impresora, móvil, periférico y licencia de ejemplo.
- [ ] Configurar relaciones entre activo, usuario, ubicación, garantía, contrato y responsable.
- [ ] Instalar GLPI Agent en clientes de prueba y comparar inventario automático con manual.
- [ ] Configurar artículos de conocimiento y una encuesta de satisfacción de prueba.
- [ ] Validar reservas, formularios y aprobaciones requeridas; instalar plugins solo tras verificar compatibilidad.
- [ ] Configurar alertas de vencimiento y probar con una fecha cercana.

**Hito:** un empleado crea un ticket, un técnico lo atiende, se asigna un activo y queda historial documentado.

### Fase 4 — Zabbix: monitorización y notificaciones

**Tareas**
- [ ] Instalar servidor/frontend y agentes necesarios.
- [ ] Añadir hipervisor o VMs y configurar nombres consistentes.
- [ ] Configurar plantillas base, métricas de CPU/RAM/disco y servicios web.
- [ ] Configurar correo; añadir Discord de forma opcional y con mensajes saneados.
- [ ] Crear umbrales de alta/media/baja y ventanas de mantenimiento.
- [ ] Verificar que una alerta se genera, llega al receptor y se registra.

**Hito:** caída simulada de un servicio provoca una alerta; recuperación genera un evento de recuperación y no deja alertas duplicadas.

### Fase 5 — Identidad y administración de equipos (condicional)

**Tareas**
- [ ] Decidir si el dominio es necesario para los objetivos de ASIR.
- [ ] Si aplica, instalar Samba AD en red interna y configurar DNS correctamente.
- [ ] Crear usuarios/grupos/OU de laboratorio y probar autenticación.
- [ ] Integrar GLPI con el directorio si corresponde y probar altas, bajas y permisos.
- [ ] Configurar una política de Windows no destructiva y validarla con un cliente.
- [ ] Probar scripts PowerShell y/o Ansible para tareas repetibles.
- [ ] Documentar método de alta/baja, cambio de equipo y revocación de acceso.

**Hito:** alta de empleado de prueba crea identidad/permisos adecuados, activo asignado y evidencia de auditoría. Si no se necesita dominio, documentar por qué y omitir esta fase.

### Fase 6 — PrestaShop: tienda de pruebas

**Tareas**
- [ ] Desplegar PrestaShop de forma independiente a GLPI.
- [ ] Crear categorías, marca ficticia, productos, SKU, imágenes autorizadas, precios de prueba y stock inicial.
- [ ] Crear clientes/direcciones completamente ficticios.
- [ ] Configurar impuestos/envío solo en modo demostración, según el caso de prueba.
- [ ] Configurar carrito, cupones y pedidos.
- [ ] Activar reseñas mediante una función/módulo compatible si la edición no lo incluye de forma suficiente.
- [ ] Configurar pago simulado o sandbox, sin transacciones reales.
- [ ] Añadir reglas de acceso, backup y monitorización de tienda.
- [ ] Registrar versiones y licencias de temas/módulos.

**Hito:** completar un pedido ficticio de principio a fin, con stock y estado del pedido verificables y sin cobro real.

### Fase 7 — Integraciones entre productos

Implementar las integraciones en este orden:

1. **Zabbix → correo**: primero validar que el canal de alertas funciona.
2. **Zabbix → GLPI**: enviar eventos seleccionados a la API/webhook de GLPI; deduplicar y probar recuperación.
3. **GLPI ↔ directorio**: solo si aplica; probar permisos y baja de usuarios.
4. **PrestaShop → GLPI**: convertir un pedido de SKU de prueba en solicitud interna o evento de preparación. Mantener el pedido y stock en PrestaShop, y los activos IT/tickets en GLPI.
5. **Discord**: añadir notificaciones operativas seleccionadas, sin secretos, datos personales ni detalles innecesarios.

**Tareas comunes**
- [ ] Crear cuenta/token por integración con permisos mínimos.
- [ ] Definir datos enviados, formato, periodicidad, tratamiento de errores y registro.
- [ ] Definir reintentos seguros y evitar tickets/pedidos duplicados.
- [ ] Probar credenciales revocadas, API no disponible y respuesta inválida.
- [ ] Documentar cómo deshabilitar la integración sin tumbar los sistemas base.

**Hito:** al menos una integración útil funciona de extremo a extremo y su fallo está controlado.

### Fase 8 — Automatización y actualizaciones controladas

**Tareas**
- [ ] Crear inventario Ansible de hosts ficticios.
- [ ] Crear playbook inicial idempotente para configurar una VM Linux.
- [ ] Crear playbook de comprobación pre-update (servicios, espacio, backup, versiones).
- [ ] Crear playbook post-update (estado de servicio, HTTP/HTTPS, conectividad y logs).
- [ ] Crear scripts PowerShell documentados para una o dos tareas Windows concretas.
- [ ] Guardar secretos mediante mecanismo cifrado o gestor externo, nunca en claro en Git.
- [ ] Definir staging/copia, ventana, aprobador, rollback y evidencia para cada cambio.

**Hito:** desplegar/configurar un host de prueba de forma repetible y ejecutar una actualización simulada con validación posterior.

### Fase 9 — Pruebas de aceptación, seguridad y recuperación

- [ ] Ejecutar la batería de pruebas de la sección 17.
- [ ] Corregir permisos excesivos, fallos de conectividad y alertas ruidosas.
- [ ] Verificar backups y restauración aislada de cada aplicación prioritaria.
- [ ] Revisar plugins, licencias, versiones y dependencias.
- [ ] Escanear/configurar servicios solo dentro del laboratorio autorizado.
- [ ] Exportar evidencias saneadas y capturas sin tokens ni contraseñas.

**Hito:** todos los casos críticos pasan o tienen una limitación registrada con plan de corrección.

### Fase 10 — Memoria ASIR, demostración y entrega

- [ ] Documentar contexto, objetivos, requisitos, alternativas y decisiones.
- [ ] Incluir diagrama lógico/físico, inventario de VMs, tabla de IP/VLAN y reglas clave.
- [ ] Explicar cada herramienta, función, licencia relevante y límites.
- [ ] Añadir instalación, operación, actualización, backup, restauración y solución de problemas.
- [ ] Añadir pruebas con pasos, resultado esperado, resultado real y evidencia.
- [ ] Incluir riesgos, medidas de seguridad, limitaciones y mejoras futuras.
- [ ] Preparar demo ordenada de 10–15 minutos y plan alternativo si una VM falla.
- [ ] Entregar repositorio saneado, documentación y copia de seguridad verificable.

**Hito final:** otra persona puede reconstruir el laboratorio siguiendo las instrucciones y demostrar los flujos principales sin depender de explicaciones orales no documentadas.

## 16. Priorización: no intentar hacerlo todo a la vez

### Must — obligatorio para un MVP sólido

- [ ] Virtualización y red aislada.
- [ ] GLPI instalado con usuarios, tickets e inventario de muestra.
- [ ] Zabbix monitorizando al menos GLPI y PrestaShop o sus servicios base.
- [ ] Backup separado y restauración demostrada.
- [ ] PrestaShop con productos ficticios y pedido de prueba sin cobro real.
- [ ] Una integración funcional demostrable (preferentemente alerta Zabbix → ticket GLPI, o evento de pedido → solicitud GLPI).
- [ ] Git, documentación, permisos y pruebas.

### Should — ampliación recomendada

- [ ] GLPI Agent en clientes de prueba.
- [ ] Avisos por correo/Discord con contenido controlado.
- [ ] Vencimientos de licencias/garantías y proceso de mantenimiento.
- [ ] Ansible para despliegue/actualización controlada de una VM Linux.
- [ ] Automatización PowerShell en Windows.
- [ ] Formulario, aprobación, reserva o encuesta probados según los requisitos académicos.

### Could — solo si hay tiempo y hardware

- [ ] Samba AD y políticas de dominio.
- [ ] Despliegue de sistemas por PXE con FOG.
- [ ] NetBox para IPAM/DCIM.
- [ ] Integración bidireccional de más datos entre tienda y gestión IT.
- [ ] Centralización avanzada de logs/seguridad.
- [ ] Alta disponibilidad o múltiples hosts.

### No hacer al principio

- [ ] No programar un portal completo desde cero.
- [ ] No integrar cinco sistemas antes de comprobarlos por separado.
- [ ] No desplegar todos los plugins disponibles.
- [ ] No automatizar actualizaciones mayores sin ensayo de recuperación.
- [ ] No abrir el laboratorio a Internet para que “parezca real”.
- [ ] No mantener el mismo dato de stock en dos aplicaciones sin reglas de sincronización.

## 17. Batería de pruebas de aceptación

Registrar para cada prueba: ID, fecha, versión, precondiciones, pasos, resultado esperado, resultado real, evidencia y estado.

| ID | Prueba | Criterio de aprobación |
|---|---|---|
| T-01 | Acceso por rol | Un empleado no puede acceder a funciones exclusivas de técnico/administrador. |
| T-02 | Crear ticket | El ticket queda asociado al usuario, categoría, prioridad y estado correctos. |
| T-03 | Ciclo de ticket | Técnico asigna, comenta, resuelve y queda historial auditable. |
| T-04 | Inventario | Un activo tiene identificador, tipo, usuario/responsable, ubicación y estado. |
| T-05 | Descubrimiento | GLPI Agent descubre datos esperados y se corrigen/explican diferencias. |
| T-06 | Vencimiento | Un contrato/licencia de prueba próximo a vencer genera el aviso definido. |
| T-07 | Monitorización | Zabbix detecta indisponibilidad controlada en un servicio de laboratorio. |
| T-08 | Recuperación | La alerta se resuelve tras restaurar el servicio y no deja duplicados persistentes. |
| T-09 | Ticket desde alerta | Evento seleccionado crea o actualiza un ticket GLPI con referencia rastreable. |
| T-10 | Ecommerce | Usuario ficticio crea un pedido completo y el stock/estado quedan registrados. |
| T-11 | Pago seguro de prueba | No se realiza ningún cobro real; sandbox/simulación funciona. |
| T-12 | Pedido a solicitud IT | SKU/pedido de prueba genera como máximo una solicitud interna válida en GLPI. |
| T-13 | Segmentación | Un origen no autorizado no alcanza la consola administrativa de otra VLAN. |
| T-14 | Backup | La copia termina y se puede localizar/verificar. |
| T-15 | Restauración | Una aplicación o dataset se restaura en entorno aislado y se valida. |
| T-16 | Automatización | Playbook/script produce el estado esperado en dos ejecuciones sin cambios innecesarios. |
| T-17 | Actualización controlada | Comprobaciones pre/post pasan o se documenta rollback realizado. |
| T-18 | Secretos | Búsqueda en repositorio no encuentra contraseñas, tokens o claves privadas. |
| T-19 | Documentación | Otra persona sigue la guía de recuperación/despliegue sin instrucciones adicionales. |

## 18. Operación y mantenimiento recurrentes

### Diario o al comenzar una sesión

- Revisar panel de Zabbix y alertas activas.
- Comprobar estado de VMs y servicios necesarios para la tarea del día.
- Verificar el último resultado de las copias de seguridad.
- Registrar cualquier cambio relevante y abrir ticket/incidencia si corresponde.

### Semanal

- Revisar espacio en disco y crecimiento de logs/backups.
- Revisar tickets pendientes, alertas repetidas y próximos vencimientos.
- Revisar actualizaciones disponibles y leer avisos de compatibilidad; no aplicar automáticamente las mayores.
- Revisar el estado del repositorio, dependencias y tareas de documentación.

### Mensual o antes de un hito

- Probar restauración de una copia.
- Revisar cuentas administrativas, tokens de integración y accesos innecesarios.
- Actualizar la matriz de versiones y la lista de licencias/módulos.
- Comprobar enlaces, procedimientos y diagramas tras cualquier cambio importante.
- Revisar la capacidad del host y decidir si apagar VMs o reducir el alcance.

## 19. Gestión de incidentes y errores

Para cada incidencia: detectar → registrar → priorizar → diagnosticar → contener → corregir → verificar → documentar → cerrar.

- No ocultar errores ni borrarlos del historial para que una demo parezca limpia.
- Registrar hora, servicio, impacto, síntomas, pasos ejecutados y resultado.
- Tras un fallo de actualización, evitar reintentos ciegos: preservar logs, verificar backup, revisar dependencias y usar el rollback definido.
- Si la integración falla, mantener operativo el sistema de origen, generar un log saneado y reintentar con control de duplicados.
- Si una tarea automatizada puede borrar o modificar muchos datos, añadir modo de prueba, confirmación explícita y alcance limitado.

## 20. Riesgos principales y mitigaciones

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Alcance excesivo para el tiempo disponible | MVP incompleto | Priorizar Must; diferir AD/PXE/NetBox/seguridad avanzada. |
| Poco RAM/almacenamiento | VMs lentas o caídas | Arrancar por fases, reducir VMs encendidas, reservar almacenamiento para backups. |
| Actualización incompatible | Aplicación rota | Matriz de versiones, staging, backup y rollback. |
| Pérdida de credenciales | Pérdida de acceso/compromiso | Gestor seguro, cuenta de emergencia protegida, rotación y secretos fuera de Git. |
| Backup en el mismo disco | Pérdida conjunta | Copia separada y, si es posible, offline. |
| Alertas excesivas | Ignorar incidentes reales | Umbrales, deduplicación, ventanas de mantenimiento y severidades. |
| Integración crea duplicados | Tickets/eventos incorrectos | IDs idempotentes, reintentos seguros y casos de prueba. |
| Plugin abandona compatibilidad | Vulnerabilidad o rotura | Preferir menos plugins, revisar mantenimiento/licencia y probar actualización. |
| Exposición pública accidental | Acceso no autorizado | VLAN, firewall por defecto-denegar, sin port forwarding por defecto. |
| Confusión de fuentes de verdad | Stock/inventario inconsistente | Definir autoridad por campo y sincronización mínima. |
| Equipo físico averiado | Pérdida del laboratorio | Backups externos y documentación para reconstruir. |

## 21. Licencias, costes y sostenibilidad

- El núcleo de GLPI se distribuye bajo GPL-3.0; comprobar licencia y obligaciones de la versión concreta y de cada plugin.
- El núcleo de PrestaShop se distribuye bajo OSL 3.0; módulos, temas, imágenes, pasarelas y servicios de terceros pueden tener condiciones distintas.
- Ansible Community, Zabbix, OPNsense, Samba y otras herramientas también tienen licencias y modelos de soporte que deben consultarse en sus fuentes oficiales.
- “Código abierto” no significa coste total cero: hardware, electricidad, dominio, hosting, soporte, módulos, certificados comerciales o servicios de terceros pueden tener coste.
- Si se modifica y redistribuye software GPL/OSL, estudiar las obligaciones correspondientes y conservar avisos/licencias. Este documento no sustituye asesoramiento jurídico.
- Mantener una tabla de componentes, versión, URL oficial, licencia, fecha de revisión, plugin/dependencias y motivo de uso.

## 22. Estructura recomendada de la memoria ASIR

1. Portada y resumen ejecutivo.
2. Objetivos generales y específicos.
3. Requisitos funcionales y no funcionales.
4. Alternativas evaluadas y justificación del software elegido.
5. Arquitectura y diagramas.
6. Planificación por fases, estimación de tareas y riesgos.
7. Instalación de virtualización, red, firewall y servicios.
8. Configuración de GLPI e inventario.
9. Monitorización Zabbix y notificaciones.
10. Identidad y automatización de sistemas, si aplica.
11. Tienda PrestaShop de pruebas.
12. Integraciones API/webhook y decisiones sobre fuentes de verdad.
13. Seguridad, roles, hardening y segmentación.
14. Backups, restauración, actualización y rollback.
15. Batería de pruebas y evidencias.
16. Costes, licencias, limitaciones y sostenibilidad.
17. Conclusiones y mejoras futuras.
18. Anexos: versiones, IP, capturas saneadas, fragmentos de configuración y bibliografía.

## 23. Checklist global de seguimiento

Marca cada casilla solo cuando exista evidencia o documentación que lo demuestre.

### Planificación
- [ ] Alcance Must/Should/Could aprobado.
- [ ] Requisitos mapeados a herramienta y prueba.
- [ ] Arquitectura, riesgos, inventario de VMs y versiones documentados.

### Infraestructura
- [ ] Hipervisor operativo.
- [ ] OPNsense y VLAN/reglas probadas.
- [ ] Acceso administrativo restringido.
- [ ] Backup fuera del disco/host que protege.

### Gestión IT
- [ ] GLPI accesible para usuario de prueba.
- [ ] Tickets, permisos e inventario probados.
- [ ] Activos, responsables, contratos/licencias y vencimientos documentados.
- [ ] Agente y funciones adicionales probadas o limitaciones registradas.

### Monitorización
- [ ] Zabbix vigila los servicios previstos.
- [ ] Correo de alerta probado.
- [ ] Discord configurado solo si es apropiado.
- [ ] Recuperación y deduplicación verificadas.
- [ ] Ticket automático probado si forma parte del MVP.

### Identidad y automatización
- [ ] Roles y proceso de alta/baja documentados.
- [ ] Dominio instalado y probado, o decisión de no incluirlo documentada.
- [ ] Playbook Ansible y/o script PowerShell probado y versionado.
- [ ] Credenciales fuera de Git.

### Ecommerce
- [ ] Catálogo, categorías y SKU ficticios.
- [ ] Carrito, cliente ficticio y pedido de prueba.
- [ ] Stock y cupones probados.
- [ ] Reseñas probadas con función/módulo compatible.
- [ ] Checkout sin cobro real.
- [ ] Backup y monitorización de PrestaShop.

### Integración
- [ ] API/webhook documentado y limitado.
- [ ] No se duplican fuentes de verdad.
- [ ] Reintentos, errores y duplicados controlados.
- [ ] Flujo de extremo a extremo demostrado.

### Seguridad y recuperación
- [ ] Reglas de firewall verificadas.
- [ ] Copias y restauración probadas.
- [ ] Actualización con comprobaciones y rollback documentado.
- [ ] Licencias y dependencias revisadas.
- [ ] Evidencias saneadas sin secretos.

### Entrega
- [ ] README permite reconstruir el laboratorio.
- [ ] Memoria ASIR y manuales terminados.
- [ ] Casos de prueba registrados.
- [ ] Demo ensayada y plan alternativo preparado.

## 24. Primeras diez acciones concretas

1. Crear el repositorio `hospit-it` e incluir este plan.
2. Anotar hardware real disponible (CPU, RAM, almacenamiento y red).
3. Elegir hipervisor y confirmar virtualización en BIOS/UEFI.
4. Dibujar el diagrama y reservar las subredes de laboratorio sin conflictos.
5. Configurar OPNsense/red antes de publicar aplicaciones.
6. Preparar destino separado para backups y comprobar una restauración sencilla.
7. Instalar GLPI conforme a sus requisitos oficiales y crear roles/activos de muestra.
8. Instalar Zabbix y monitorizar GLPI antes de añadir más sistemas.
9. Instalar PrestaShop en otra VM y completar un pedido ficticio sin cobro real.
10. Solo cuando ambos sistemas funcionen, implementar una integración pequeña y probarla.

## 25. Fuentes oficiales para consultar antes de instalar

Usar siempre la documentación de la versión concreta que se vaya a instalar; las páginas pueden cambiar con el tiempo.

- GLPI, código y licencia: https://github.com/glpi-project/glpi
- GLPI, guía oficial de instalación y prerrequisitos: https://github.com/glpi-project/doc-install
- Zabbix, documentación oficial (API e integraciones): https://www.zabbix.com/documentation/current/es/manual/api
- Ansible Community, documentación: https://docs.ansible.com/
- OPNsense, documentación general: https://docs.opnsense.org/
- OPNsense, VLAN y LAGG: https://docs.opnsense.org/manual/how-tos/vlan_and_lagg.html
- PrestaShop, proyecto open source: https://www.prestashop-project.org/
- PrestaShop, licencia y compatibilidad: https://www.prestashop-project.org/maintainers-guide/general-guidelines/license/
- PrestaShop, repositorio: https://github.com/PrestaShop/PrestaShop
- Samba, documentación: https://www.samba.org/samba/docs/
- NetBox, documentación: https://docs.netbox.dev/
- Proxmox VE: https://www.proxmox.com/en/proxmox-virtual-environment/overview

---

**Regla final de HOSPIT-IT:** si una función no está instalada, configurada, probada y documentada, no debe presentarse como terminada. Si se aplaza, anotar motivo, impacto y condición para retomarla.
