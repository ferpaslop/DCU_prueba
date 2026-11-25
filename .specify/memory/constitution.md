<!--
Sync Impact Report
==================
Version change: 1.1.0 → 1.2.0
Bump rationale: Se incorpora OAuth 2.0 como mecanismo común de autorización de acceso a las
APIs, con validación de tokens en servidor. Se conserva el alcance específico de G2 añadido
en la versión 1.1.0. Quedan pendientes el proveedor y el flujo OAuth concreto.

La versión 1.0.0 sustituyó a:
  - constitution v0.2.0 (Gestor del Club de Rugby)
  - constitution v1.0.0 (Módulo base: identidad y autorización)
  - Parte Común de la constitución del Módulo 3 v1.1.0 (Comunicación y Equipaciones)
  - Principios y flujo comunes de la constitución del Módulo de Socios v1.1.0

Alcance específico incorporado en esta versión:
  - G2 Gestión Deportiva: alcance, superficies PWA/web, usuarios y permisos, reglas de dominio,
    límites entre módulos, contratos y criterios de aceptación.

Se mantiene en las especificaciones/constituciones propias de cada módulo:
  - G3 Socios y Cuotas: detalle del directorio, cuotas, pagos y facturación.
  - G4 Comunicaciones y Equipaciones: detalle de Noticias/Avisos/Equipaciones/Incidencias.

Principios:
  (Sección conservada: Módulos, Responsabilidades y Dependencias — G1 Infra, G2 Gest. Deportiva,
   G3 Socios y Cuotas, G4 Comunicaciones y Equipaciones. Se incorpora el detalle específico de G2.)
  I. Fidelidad al dominio y control del alcance
  II. Arquitectura modular e independiente
  III. Desarrollo guiado por contratos (API-First)
  IV. Seguridad, privacidad e identidad única (NON-NEGOTIABLE)
  V. Integridad de datos y trazabilidad
  VI. Experiencia por rol, accesibilidad e idioma
  VII. Calidad comprobable y pruebas
  VIII. Simplicidad y mantenimiento
  IX. Requisitos confirmados frente a decisiones pendientes
  X. Integración progresiva con datos simulados

Conflictos entre fuentes resueltos (revisar) :
  - Canales: se adopta la visión de v0.2.0 (web pública + web de gestión PC + PWA móvil);
    "Mobile-First" del Módulo 3 se aplica a la PWA y a las vistas de entrenador.
  - Almacenamiento: el Módulo de Socios decía "reutilizar su almacenamiento"; se reinterpreta
    como "reutilizar la infraestructura de persistencia del proyecto" sin acceso a datos de
    otro módulo (Principio II).

Decisiones pendientes registradas (ver sección "Decisiones Pendientes"):
  - Catálogo único de roles; tutor/familiar con o sin cuenta; autenticación y 2FA;
    recuperación de cuentas; consentimiento de menores; asignación de roles;
    BD/alojamiento/pasarela/WhatsApp/observabilidad; fecha de ratificación.
-->

# Gestor del Club de Rugby (València Rugby Club) Constitution

Esta constitución establece las reglas de producto, ingeniería e integración que deben
cumplir las especificaciones, planes, tareas e implementaciones de **todos los módulos** del
sistema de gestión del club. Cada módulo puede añadir una constitución o sección específica
con su alcance, pero **no puede contradecir** esta constitución.

## Core Principles

### I. Fidelidad al dominio y control del alcance

El sistema MUST representar el funcionamiento real del club y MUST NOT atribuirle
responsabilidades que pertenecen a terceros.

- El club utiliza instalaciones públicas de terceros. El sistema solo gestionará y consultará
  las franjas reservadas al club; MUST NOT gestionar la instalación, su mantenimiento, su
  disponibilidad global ni las reservas de otras entidades.
- MUST NOT existir venta de entradas a espectadores como fuente de ingresos. Las cuotas son el
  modelo económico principal. Ventas, patrocinios, gastos, descuentos y regalos se registran
  cuando correspondan a la operativa administrativa.
- Se distinguen las modalidades Rugby 15, Rugby 7 y Rugby Adaptado, y la estructura
  modalidad → categoría → equipo (una categoría puede contener varios equipos).
- Se mantienen diferenciados posible cliente, simpatizante, socio y deportista federado. Ser
  socio y estar federado NO son conceptos equivalentes.
- Las etiquetas de patrocinadores no son excluyentes: un patrocinador puede tener varios ámbitos.
- Las equipaciones reutilizables tienen trazabilidad de asignación, devolución y limpieza; MUST
  NOT suponerse que todas las prendas siguen el mismo ciclo.
- Cada funcionalidad MUST declarar a qué módulo pertenece. Los módulos no aprobados o fuera de
  alcance MUST NOT introducirse como dependencias implícitas. Toda ampliación requiere
  especificación y aprobación explícitas.

### II. Arquitectura modular e independiente

Cada módulo es dueño de sus datos, reglas de negocio, validaciones y ciclo de vida.

- Ningún módulo MUST leer o escribir directamente las tablas, colecciones o repositorios
  internos de otro módulo.
- La integración entre módulos se realiza solo mediante contratos REST versionados y
  documentados, con propietario identificado.
- Cada equipo MUST poder desarrollar y probar su módulo de forma independiente.
- `userId` es la referencia de vinculación con la identidad; los módulos no duplican el perfil
  completo ni crean identidades ni modifican roles.
- Un módulo se integra en la navegación del proyecto sin alterar rutas ni estilos de otros
  módulos, y reutiliza la sesión y los permisos del proyecto a través de los contratos
  definidos.
- Se evitan duplicaciones de reglas de negocio entre web de gestión, PWA y web pública; las
  reglas viven en el backend.

### III. Desarrollo guiado por contratos (API-First)

Toda comunicación entre frontends, backend y módulos se realiza mediante endpoints REST.

- El contrato MUST definirse antes de implementar proveedor o consumidor, e incluir método,
  ruta, parámetros, autenticación, autorización, esquema de respuesta, errores y versión.
- Los consumidores MUST NOT depender de detalles internos del proveedor.
- Un cambio incompatible requiere nueva versión de contrato, plan de migración y periodo de
  compatibilidad explícito.
- Las respuestas son deterministas y usan nombres de campo consistentes en español cuando sean
  datos funcionales; los nombres de endpoints permanecen estables.
- Convenciones comunes:
  - Colecciones: lista con metadatos de paginación cuando proceda; recurso: objeto identificado.
  - Fechas en ISO 8601 con zona horaria.
  - Identificadores opacos; no se interpretan como índices secuenciales.
  - `400` petición inválida, `401` identidad ausente o inválida, `403` rol insuficiente,
    `404` recurso inexistente o no visible, `409` conflicto de estado.
  - Los errores incluyen código estable, mensaje para la interfaz y, si es seguro, campos
    afectados. MUST NOT devolverse trazas internas.
- Los contratos se documentan con OpenAPI y Swagger UI (ver Restricciones Técnicas).

### IV. Seguridad, privacidad e identidad única (NON-NEGOTIABLE)

- **Validación en servidor.** La identidad, el rol, el ámbito del recurso y las reglas de
  negocio MUST validarse en el servidor o capa de dominio. Ocultar un botón o validar solo en
  la interfaz NO es control de acceso. El cliente nunca puede elevar privilegios mediante
  parámetros, mocks o cambios de interfaz; un `userId` o `rol` enviado por el cliente no basta
  y MUST contrastarse con la identidad autenticada.
- **Denegación por defecto y mínimo privilegio.** Cada persona tiene una identidad única, aunque
  pueda acumular varios perfiles. La autorización se expresa mediante una matriz explícita de
  permisos por perfil, recurso y ámbito. Añadir un perfil o permiso es un cambio deliberado y
  revisado. La asignación de roles registra quién la concede, a quién y cuándo.
- **Respuestas explícitas.** Las operaciones restringidas devuelven un error explícito, no una
  respuesta vacía que parezca éxito. Los datos se minimizan según rol y contexto; las alertas o
  datos restringidos se filtran antes de llegar al cliente y no son accesibles por
  identificadores alternativos.
- **Datos sensibles.** Los datos de menores y de lesiones/salud requieren protección reforzada:
  solo los consultan quienes los necesitan para sus funciones. Los datos del menor y su tutor
  tienen tratamiento diferenciado, con autorización del tutor, visibilidad restringida y sin
  exposición pública por defecto. Correo y teléfono son obligatorios en el registro; el DNI es
  opcional y solo se exige si el perfil o un requisito legal lo imponen, con acceso restringido.
  Se recoge el mínimo de datos necesario con finalidad declarada.
- **Credenciales y secretos.** Contraseñas con hash adaptativo y sal; sesiones con expiración y
  revocables; secretos fuera del repositorio. Ninguna credencial, token, dato de pago completo,
  dato personal o de salud innecesario MUST aparecer en código, registros, trazas o mensajes de
  error.
- **Reparto:** autenticación, verificación, recuperación, 2FA, roles y matriz base son
  responsabilidad de G1; el resto de módulos las consume por contrato y solo implementa la
  autorización de sus propios recursos. El acceso a las APIs se protegerá mediante OAuth 2.0,
  utilizando tokens de acceso (Bearer) emitidos por el proveedor de autorización acordado.
  OAuth 2.0 define la autorización para acceder a las APIs; si el sistema necesita un protocolo
  de identidad/autenticación de usuario, se concretará si se utiliza OpenID Connect sobre OAuth 2.0.
- **Verificación y recuperación (G1).** El estado de verificación de correo y teléfono forma parte de
  la identidad. La recuperación de cuenta y cualquier segundo factor no revelan si una cuenta
  existe y usan mecanismos de un solo uso con caducidad.
- **Auditoría.** Los eventos de autenticación, cambios de permisos, operaciones administrativas
  sensibles y cambios relevantes de estado dejan traza con marca temporal, actor y resultado,
  sin secretos ni datos personales innecesarios.
- **Errores al usuario.** Comprensibles y sin información interna, datos de terceros ni
  detalles que faciliten accesos no autorizados.
- El cumplimiento de RGPD y normativa española de protección de datos MUST verificarse antes de
  exponer datos reales; los tratamientos de menores, salud y pagos requieren validación legal y
  operativa del club. Esta constitución no sustituye dicha validación.

### V. Integridad de datos y trazabilidad

Las reglas de dominio de esta sección las implementa **únicamente el módulo propietario**
(ver Módulos y Responsabilidades); los demás las consumen por contrato.

- Las reglas de negocio críticas no dependen solo del estado mostrado en pantalla. Las
  modificaciones se validan y mantienen coherencia entre interfaces. Las integraciones externas
  no son fuente de verdad implícita.
- Las entradas se validan y las operaciones relacionadas evitan resultados parciales
  inconsistentes.
- La asistencia distingue como mínimo confirmada, ausencia, lesión y falta de respuesta; la falta
  de respuesta NO se interpreta como confirmación.
- Actividades y entrenamientos se asocian a las franjas reservadas por el club.
- Cuotas, pagos, ventas, patrocinios, gastos, descuentos y regalos son consultables de forma
  consistente desde áreas autorizadas, y el historial identifica concepto y naturaleza de cada
  operación. Un regalo se registra como tal y no se confunde con ingreso efectivo, ni siquiera
  como operación de importe cero. Registrar un pago es dejar constancia de un cobro; no ejecuta
  un cargo bancario.
- Inventario y asignaciones de equipación reflejan el estado real de cada unidad (asignada,
  pendiente de devolución, pendiente de lavar, en lavado, limpia/lista, según el flujo definido).
- Antes de usar datos reales MUST comprobarse la recuperación de copias de seguridad. Los
  cambios con riesgo de pérdida de información requieren estrategia de recuperación.

### VI. Experiencia por rol, accesibilidad e idioma

- **Canales:** web pública (noticias, eventos abiertos, vías de registro), web de gestión para
  PC (administración) y PWA móvil para socios y jugadores, con vista móvil específica para el
  entrenador. La PWA se diseña primero para teléfono y uso táctil con una mano.
- **Segmentación por rol:** la interfaz refleja necesidades y permisos de cada perfil; no se
  impone una interfaz única que exponga funciones irrelevantes o prohibidas. La segmentación no
  puede ocultar el motivo de una acción no disponible ni exponer datos de otro rol.
- **Accesibilidad:** cumplimiento de WCAG 2.1 AA (contraste, foco visible, etiquetas de
  formulario, mensajes de error). Operaciones esenciales compatibles con teclado y tecnologías
  de asistencia; MUST NOT depender de gestos complejos (arrastrar, pellizcar, mantener); áreas
  táctiles suficientes; información importante MUST NOT transmitirse solo por color o iconos sin
  etiqueta. La accesibilidad se comprueba en los flujos principales, no al final.
- **Estados:** todo flujo muestra de forma visible carga, vacío, éxito y error con texto
  comprensible; los errores indican qué ocurrió y cómo corregirlo. Las acciones críticas piden
  confirmación perceptible.
- **Idioma:** español como idioma de referencia, con vocabulario del club (modalidades,
  categorías, equipos, convocatorias, cuotas, fichas federativas, equipaciones) y comprensible
  sin conocimientos técnicos; textos centralizados y traducibles.
- Las vistas de gestión se revisan en resolución de ordenador y las generales en móvil y
  ordenador antes de aceptarse.

### VII. Calidad comprobable y pruebas

Una función no está terminada porque funcione en el caso ideal ni mientras incumpla criterios
obligatorios.

- Cada función define quién la usa, qué necesidad resuelve, cómo se comporta y cómo se verifica.
- Cada entrega indica qué criterios se verificaron y cuáles siguen pendientes, e incluye
  instrucciones de uso y limitaciones o integraciones pendientes.
- Las reglas de negocio, permisos y persistencia tienen pruebas automatizadas proporcionales al
  riesgo. Las rutas de autenticación/autorización tienen pruebas de acceso permitido **y**
  denegado. Cada API tiene pruebas de contrato.
- Backend: xUnit (reglas de negocio e integración de APIs/persistencia). Navegador y E2E:
  Playwright (flujos críticos y combinaciones de roles relevantes).
- Los errores corregidos añaden prueba de regresión cuando sea viable.
- Antes de integrar se ejecutan pruebas, análisis estático, formato y compilación.
- Se priorizan los flujos de necesidad acreditada (la encuesta inicial de 12 respuestas respalda
  confirmación de asistencia y consulta de cuota/ficha federativa; es orientativa).

### VIII. Simplicidad y mantenimiento

- Se elige la solución más sencilla que cumpla los requisitos. No se añaden capas,
  dependencias, servicios o automatizaciones sin necesidad demostrable; la complejidad añadida
  se justifica en el plan.
- La configuración se separa de la lógica de negocio. Las instrucciones de instalación,
  ejecución y verificación se mantienen actualizadas.
- La lógica de negocio es comprobable sin depender de la interfaz y se mantiene separada de la
  presentación y de proveedores externos; las integraciones de pago y mensajería se encapsulan
  para poder sustituir proveedor.
- Las decisiones de arquitectura, persistencia, despliegue, autenticación y proveedores que
  afecten a más de un módulo se documentan (ADR).

### IX. Requisitos confirmados frente a decisiones pendientes

- Las especificaciones separan visiblemente requisitos confirmados, decisiones pendientes y
  fuera de alcance; las suposiciones se distinguen de los requisitos.
- Una decisión pendiente MUST NOT resolverse implícitamente en código, valores por defecto no
  documentados o suposiciones. Se registra con alcance, alternativas, responsable y efecto en
  seguridad, menores y UX. Mientras siga abierta, el código la trata como configurable o
  explícitamente no soportada, y las pruebas lo reflejan.
- Las decisiones pendientes viven en un registro único y se cierran antes de implementar el
  flujo afectado; su cierre actualiza la especificación.

### X. Integración progresiva con datos simulados

- Los mocks permiten desarrollo en paralelo mientras la infraestructura real no esté
  disponible. Están en español, usan los mismos campos, estados, permisos y errores que la
  integración real y son sustituibles sin cambiar consumidores.
- Los mocks NO son una vía alternativa de autorización. Incluyen casos públicos, restringidos,
  vacíos, inválidos y de error.
- La documentación identifica qué datos están simulados y qué queda pendiente de integración
  real. Una demo aislada no acredita integración: esta se considera completa cuando se conecta a
  los servicios reales y se verifican los permisos.
- Demostraciones, pruebas y seeds usan datos ficticios, sin secretos ni datos personales reales.

## Módulos, Responsabilidades y Dependencias

El proyecto se divide en cuatro módulos, uno por grupo. **Cada grupo entrega, solo para su
dominio, la parte de usuario (PWA móvil) y la parte de administración (web de gestión en PC).**

| Grupo | Módulo | Es propietario de (y nadie más implementa) |
|---|---|---|
| G1 | Infraestructura | Identidad única, usuarios, registro, autenticación, sesión, verificación, recuperación de cuenta, 2FA, roles y matriz base de permisos, auditoría de autenticación, administración de usuarios, estructura común del proyecto (navegación/panel general, ver pendiente) |
| G2 | Gestión Deportiva | Catálogo y estructura modalidad → categoría → equipo; relación deportiva y composición de plantillas; datos deportivos del deportista; entrenamientos, partidos y franjas ya reservadas al club; convocatorias, respuestas de asistencia y disponibilidad/lesiones; consulta móvil del entrenador. Detalle y límites: sección específica G2. |
| G3 | Socios y Cuotas | Socios y su directorio, cuotas, registro de pagos, facturación |
| G4 | Comunicaciones y Equipaciones | Noticias, agenda pública, avisos y alertas, inventario y ciclo de equipaciones, incidencias de material |

### Alcance específico G2 — Gestión Deportiva

Esta sección concreta el alcance del grupo G2 sin modificar los principios comunes. En caso de
conflicto, prevalecen los principios y las reglas compartidas anteriores.

#### Objetivo y alcance funcional

G2 es responsable de organizar la actividad deportiva del club y mantener una visión fiable de
sus modalidades, categorías, equipos, deportistas, actividades, convocatorias y asistencia. El
módulo debe cubrir dos superficies coordinadas: la **web de gestión para PC**, destinada a la
administración deportiva, y las **vistas de Gestión Deportiva de la PWA**, destinadas a jugadores
y entrenadores. Las reglas de negocio residen en el backend de G2 y son compartidas por ambas
interfaces.

El alcance deportivo inicial comprende Rugby 15, Rugby 7 y Rugby Adaptado, con la jerarquía
**modalidad → categoría → equipo**. Una categoría MUST poder contener varios equipos y cada equipo
MUST poder gestionarse de forma independiente. El diseño MUST NOT asumir que todos los equipos,
modalidades o deportistas tienen las mismas necesidades de interacción.

#### 1. Web de gestión deportiva (PC)

La web de gestión MUST proporcionar, al menos, las siguientes capacidades:

1. **Panel deportivo.** Mostrar una visión rápida de la agenda de franjas/actividades del club y
   del estado de los equipos. Los indicadores deportivos deben permitir localizar equipos con
   respuestas pendientes o jugadores declarados como lesionados, sin mostrar datos médicos
   innecesarios. La información económica, las equipaciones y el envío de comunicaciones se
   obtienen de sus módulos propietarios y no se administran desde G2.
2. **Estructura deportiva.** Navegar y filtrar por modalidad, categoría y equipo (por ejemplo,
   Rugby 7 → Juvenil → Equipo A). Consultar y mantener las categorías y equipos, y gestionar la
   composición de cada equipo de manera independiente. La gestión de modalidades debe respetar
   el catálogo del club y no crear variantes incompatibles con los datos compartidos.
3. **Plantillas y deportistas.** Consultar el listado de miembros de cada equipo, vincular un
   deportista a uno o varios equipos cuando las reglas aprobadas lo permitan, y gestionar las
   altas, bajas o cambios de pertenencia deportiva. La ficha de G2 contendrá solo los atributos
   deportivos necesarios (por ejemplo, modalidad, posición habitual y antigüedad en el club) y
   referencias a la identidad. Los datos personales, de contacto y de tutoría se consultan a su
   módulo propietario; MUST NOT duplicarse en G2.
4. **Agenda y actividades deportivas.** Crear, consultar, modificar y cancelar registros de
   entrenamientos y partidos del club, indicando el equipo o ámbito deportivo correspondiente,
   fecha, horario y ubicación. Cuando una actividad utilice una instalación pública, podrá
   vincularse a una franja que el club ya tenga reservada.
5. **Franjas reservadas.** Registrar y consultar la representación interna de las franjas
   asignadas al club, relacionándolas con las actividades que se desarrollan en ellas. La agenda
   permitirá consultar las franjas por día/semana/mes y su detalle. G2 MUST NOT solicitar,
   confirmar ni administrar reservas en nombre de la entidad propietaria de la instalación.
6. **Convocatorias y asistencia.** Asociar una convocatoria a un entrenamiento o partido y al
   conjunto de deportistas convocados; consultar las respuestas recibidas y distinguir los
   estados definidos por el proyecto: asistencia confirmada, ausencia, lesión y falta de
   respuesta. La falta de respuesta MUST permanecer visible como tal y MUST NOT interpretarse
   como asistencia confirmada.
7. **Seguimiento por equipo.** Presentar de forma clara el estado de cada deportista para una
   actividad concreta y permitir consultar el historial deportivo necesario para organizar el
   equipo, respetando los permisos y la minimización de datos.

El panel y las vistas de G2 MUST priorizar agenda, composición de equipos y respuestas a
convocatorias. No deben convertirse en un panel económico o en una herramienta de comunicación
general del club.

#### 2. PWA — jugador/deportista

El jugador adulto federado y el jugador menor de edad utilizarán las funciones deportivas desde
la PWA. Cada jugador MUST poder:

- Consultar su calendario de entrenamientos y partidos, con fecha, horario y ubicación.
- Consultar el detalle de las convocatorias que le correspondan.
- Confirmar su asistencia o indicar que no asistirá.
- Declarar que está lesionado o que su lesión le impide participar en la actividad, de acuerdo
  con el flujo de respuesta definido por G2.
- Consultar el estado de su propia respuesta y corregirla mientras la convocatoria admita
  modificaciones.

La interfaz del jugador MUST mostrar únicamente sus convocatorias y sus propios datos
deportivos; MUST NOT permitir consultar listados, respuestas o lesiones de otros deportistas.

El jugador menor tendrá una experiencia simplificada y las mismas funciones deportivas básicas
que correspondan a su condición de jugador. No podrá acceder a pagos, compras ni gestiones
administrativas reservadas al adulto/tutor. G2 no gestiona la cuenta del tutor: utiliza la
referencia y las autorizaciones que proporcione G1 o el módulo propietario.

La documentación de investigación propone solicitar las respuestas aproximadamente tres o
cuatro días antes de cada entrenamiento o partido. Ese plazo sirve como referencia de diseño,
pero no se convierte en una fecha límite obligatoria hasta que el equipo acuerde y documente la
regla de cierre de convocatorias.

#### 3. PWA — entrenador

El entrenador dispondrá de una vista móvil específica, limitada a los equipos que tenga
asignados. Desde ella MUST poder:

- Consultar el calendario, entrenamientos, partidos y convocatorias de sus equipos.
- Ver el listado de deportistas convocados y el estado individual: va, no va, lesionado o sin
  responder.
- Identificar rápidamente las respuestas pendientes y la disponibilidad del equipo antes y
  durante la actividad.

En la primera versión, la vista del entrenador es de **consulta**: no podrá modificar datos
personales o deportivos de los jugadores, administrar equipos, consultar cuotas/facturas ni crear
o enviar avisos generales. Si detecta un cambio de horario o una necesidad de comunicación,
lo trasladará al responsable autorizado del club, que utilizará el módulo de comunicaciones.
La autoridad exacta para crear/publicar una convocatoria se mantiene como decisión pendiente
(ver Decisiones Pendientes).

La navegación debe adaptarse al rol. La PWA de jugadores mantiene las secciones Inicio,
Calendario, Noticias y Perfil; la vista del entrenador añade Equipos. Los perfiles de socio no
jugador, simpatizante y posible cliente MUST NOT mostrar funciones de equipos, entrenamientos,
convocatorias ni estado físico.

#### 4. Usuarios, permisos y ámbito de acceso de G2

| Perfil | Interfaz | Permisos de Gestión Deportiva |
|---|---|---|
| Gestor/admin del club | Web de gestión | Consulta global y gestión de los recursos deportivos de G2, de acuerdo con los permisos base de G1. |
| Gestor de equipos | Web de gestión | Consulta los equipos de su ámbito y gestiona su composición; consulta deportistas, convocatorias, respuestas y lesiones necesarias para esa función. No accede a facturas, historial económico, administración de usuarios ni envío de avisos. |
| Entrenador | PWA móvil | Consulta calendario, convocatorias, plantillas y estados de los equipos asignados. Sin permisos de edición de datos de personas/equipos ni de comunicación general. |
| Jugador adulto federado | PWA móvil | Consulta su calendario y convocatorias; registra y modifica su propia respuesta mientras esté abierta. No consulta datos de otros jugadores ni funciones de gestión. |
| Jugador menor de edad | PWA móvil | Consulta sus actividades y convocatorias; registra su propia respuesta. No accede a pagos, compras ni funciones administrativas. |
| Socio no jugador, simpatizante y posible cliente | PWA/web pública | Sin acceso a recursos internos de G2. Solo podrán ver información deportiva que otro módulo publique expresamente como contenido público. |
| Director deportivo | Fuera de la primera versión | La supervisión global, informes de rendimiento y permisos específicos quedan reservados para una fase futura aprobada. |

Los permisos anteriores se aplican en el backend y se restringen por recurso y ámbito (club,
modalidad, categoría, equipo y deportista). El frontend solo refleja dichos permisos; MUST NOT ser
la única barrera de seguridad. La asignación de roles y ámbitos sigue perteneciendo a G1.

#### 5. Datos y reglas de dominio propiedad de G2

G2 es propietario de las reglas y del ciclo de vida de los siguientes conceptos, cuyo esquema
exacto se definirá en la especificación del módulo:

- Catálogo de modalidades deportivas del club y estructura de categorías/equipos.
- Relación entre deportistas y equipos, y atributos estrictamente deportivos del deportista.
- Entrenamientos, partidos y su planificación deportiva.
- Registro interno de franjas ya reservadas al club y su asociación con actividades.
- Convocatorias, deportistas convocados y respuestas individuales de asistencia.
- Declaración de disponibilidad/lesión necesaria para determinar la participación en una
  actividad.

Reglas obligatorias:

- Toda pertenencia a equipo MUST quedar vinculada a una identidad existente mediante el
  identificador opaco proporcionado por G1. G2 MUST NOT crear usuarios, credenciales ni roles.
- La estructura modalidad → categoría → equipo MUST mantenerse coherente. Los cambios de
  composición o estructura MUST NOT romper la trazabilidad de actividades y convocatorias ya
  realizadas.
- Una respuesta de asistencia pertenece a un deportista y a una convocatoria concreta. Solo el
  propio deportista podrá registrar su respuesta desde la PWA; las acciones administrativas
  excepcionales deberán estar expresamente autorizadas y auditadas.
- Los estados de asistencia se validan en servidor. «Sin responder» es un estado explícito, no
  un valor por defecto que se convierta en «va».
- La lesión se tratará como dato sensible. En esta versión se recogerá únicamente la indicación
  necesaria para comunicar que el deportista no está disponible por lesión; no se solicitarán
  diagnósticos ni historiales clínicos como parte del flujo ordinario. Solo se mostrará a los
  perfiles que necesiten esa información para organizar la actividad.
- Las actividades y las franjas se validan frente a la planificación interna del club. Los
  conflictos de horario/espacio deben producir un error comprensible y no guardar una asignación
  incoherente; las reglas exactas de recurrencia y solapamiento se concretarán en la
  especificación.
- El borrado o desactivación de equipos, categorías, actividades o deportistas MUST preservar
  las referencias históricas necesarias para entender convocatorias y asistencias anteriores.

#### 6. Límites entre módulos y contratos

- **G1 — Infraestructura:** es la fuente de identidad, cuenta, sesión, roles y permisos base.
  G2 consume `userId` y los ámbitos autenticados; no mantiene copias de datos personales ni
  implementa autenticación.
- **G3 — Socios y Cuotas:** mantiene la información de socio y los datos económicos que le
  correspondan. G2 expone, mediante contrato, las referencias y datos deportivos mínimos que
  G3 necesite para relacionar socio y deportista. G2 no gestiona cuotas, pagos ni facturas.
- **G4 — Comunicaciones y Equipaciones:** consume de G2 los identificadores y metadatos mínimos
  de modalidad, categoría, equipo y deportista que necesite para segmentar avisos o asociar una
  equipación/incidencia. G2 no crea noticias, no envía notificaciones ni integra directamente
  WhatsApp. G4 no accede a las tablas internas de G2.
- La titularidad de la ficha federativa oficial y de sus trámites/documentos no queda resuelta
  por la documentación de partida. Hasta que G2 y G3 acuerden el propietario, G2 se limitará a
  la relación deportiva del deportista y no duplicará la ficha federativa administrativa.
- Los contratos de G2 MUST publicar, como mínimo, las operaciones y esquemas necesarios para
  consultar estructura deportiva, equipos y ámbitos autorizados; consultar referencias de
  deportistas y su pertenencia a equipos; consultar agenda/actividades y franjas del club; y
  gestionar/consultar convocatorias y respuestas. Cada contrato especificará método, ruta,
  parámetros, autenticación, permisos, paginación, estados, errores y versión según el Principio
  III.
- Antes de que G3 o G4 integren datos de G2, este grupo MUST proporcionar OpenAPI/Swagger y un
  mock con casos permitidos, denegados, vacíos y de error. Las respuestas destinadas a G4
  incluirán solo los datos necesarios para segmentar o relacionar recursos, nunca diagnósticos
  ni información personal ajena a su finalidad.

#### 7. Fuera de alcance de G2 en la primera versión

G2 MUST NOT implementar:

- Gestión global de instalaciones, mantenimiento, disponibilidad pública ni reservas de otras
  entidades; solo registra las franjas que el club ya tiene asignadas.
- Cuotas, pagos, facturas, directorio general de socios, compras o descuentos.
- Fichas federativas oficiales, pagos federativos o trámites de federación mientras no se
  ratifique su propietario y contrato.
- Noticias, avisos generales, envíos de WhatsApp o mensajería directa entrenador–jugador.
- Inventario, asignación, devolución o limpieza de equipaciones.
- Gestión pública de eventos sociales, web pública o publicación de resultados/clasificaciones.
- Informes globales de rendimiento, estadísticas avanzadas o supervisión del Director Deportivo.
  Estas capacidades requieren una fase y especificación posteriores.

#### 8. Prioridad y criterios de aceptación de G2

La prioridad funcional inicial es que el jugador pueda responder a una convocatoria y que el
entrenador/gestor consulte con claridad la disponibilidad del equipo. La encuesta de partida
obtuvo un 83,3 % de respuestas favorables a confirmar/cancelar asistencia (12 respuestas en
total); el dato orienta la prioridad, pero no se considera estadísticamente representativo de
todo el club.

Una entrega de G2 no se considera completa hasta verificar, como mínimo, que:

1. La web permite navegar modalidad → categoría → equipo, consultar plantillas y gestionar la
   composición dentro del ámbito autorizado.
2. Los entrenamientos/partidos y las franjas propias del club aparecen de forma coherente en la
   agenda, sin gestionar reservas de terceros.
3. Un jugador solo consulta sus convocatorias y puede registrar su propia respuesta; una
   convocatoria distingue asistencia, ausencia, lesión y falta de respuesta.
4. Un entrenador solo consulta los equipos asignados y ve el estado de cada convocado sin poder
   acceder a datos de otros equipos o a información económica.
5. Los permisos se comprueban en backend y tienen pruebas positivas y negativas por rol y
   ámbito.
6. La información de lesión no se expone a jugadores distintos del afectado ni a módulos que no
   la necesiten.
7. Los flujos esenciales funcionan en PC y móvil, con estados de carga/vacío/error/éxito y con
   los requisitos de accesibilidad del Principio VI.

### Reglas de responsabilidad

- **Responsabilidad limitada:** un grupo MUST NOT implementar funcionalidad que pertenezca a
  otro módulo (p. ej. login, gestión de usuarios, equipos o fichas federativas fuera de su
  dueño). Si necesita algo ajeno, lo solicita al propietario como contrato; no lo reimplementa
  ni lo duplica.
- Un módulo guarda de otro solo referencias opacas (`userId`, `equipoId`, etc.), nunca copias
  del dato ni de las reglas de negocio del propietario.
- Los datos ajenos se muestran en la interfaz consultando la API del propietario, aplicando
  siempre sus permisos.

### Dependencias (propuesta, a confirmar entre grupos)

| Consumidor → Proveedor | Qué necesita |
|---|---|
| G2, G3, G4 → G1 | Validar sesión, `userId`, `rol` y ámbitos; todos dependen de G1 |
| G4 → G2 | Segmentar avisos por equipo/categoría; ámbito deportivo del entrenador; deportista afectado en equipaciones e incidencias |
| G3 → G2 | Relación socio ↔ deportista federado; menores y su tutor |
| G4 → G3 | Segmentar comunicaciones por colectivos (socios) |

- Las dependencias van en un solo sentido y MUST NOT formar ciclos; cualquier dependencia
  nueva se añade aquí con propietario y contrato.
- **Nadie queda bloqueado en el desarrollo:** el proveedor publica el contrato (OpenAPI) y un
  mock fiel **antes** de que el consumidor implemente (Principio X). Lo único que puede
  esperar a un proveedor es la **integración real**, nunca el desarrollo ni las pruebas
  del consumidor.
- **G1 desbloquea al resto:** su primer entregable prioritario es el contrato de identidad
  (sesión → `userId`, `rol`, ámbitos) con mock operativo. Hasta que exista la integración
  real, el resto usa ese mock sin atajos de autorización.
- Integración por fases: (1) contrato + mock, (2) pruebas de contrato contra el proveedor,
  (3) integración real con permisos verificados.
- Un cambio en un contrato compartido requiere aviso y acuerdo de proveedor y consumidores;
  los incompatibles siguen el Principio III. El proveedor mantiene disponibles su Swagger y su
  mock actualizados.
- Los bloqueos entre grupos que no se resuelvan se registran como decisión pendiente y se
  escalan al responsable del proyecto.

## Restricciones Técnicas

1. **Frontend:** React + TypeScript, aplicación independiente del backend (proyecto y
   build/despliegue propios), que consume APIs por contrato. Node.js es el entorno de
   herramientas del frontend, no del backend.
2. **Backend:** .NET 10.0 y C# 14.0 con ASP.NET Core Minimal APIs; sin controladores MVC salvo
   enmienda aprobada. Cualquier cambio de versión requiere enmienda.
3. **Autorización de acceso a las APIs:** OAuth 2.0. Las solicitudes a endpoints protegidos
   MUST incluir un token de acceso válido mediante el esquema Bearer. Cada API MUST validar
   como mínimo la firma o mecanismo de validación establecido por el proveedor, emisor,
   audiencia, expiración y permisos/ámbitos del token; además MUST aplicar la autorización
   propia del recurso y del ámbito en el servidor. La presencia de un token válido no concede
   automáticamente acceso a todos los recursos. Los secretos y credenciales de clientes MUST
   mantenerse fuera del código y de los repositorios.
4. **Documentación de API:** OpenAPI + Swagger UI, con exposición segura según entorno.
5. **Persistencia:** Entity Framework Core (compatible con .NET 10), migraciones versionadas y
   revisables; seeding idempotente de datos iniciales/referencia, sin secretos ni datos
   personales reales, que no sustituye a las migraciones.
6. **Mensajería asíncrona:** Azure Service Bus para tareas por colas, con manejo de fallos,
   reintentos y entregas duplicadas (procesamiento idempotente cuando sea posible).
7. **Tiempo real:** SignalR (WebSockets preferente) solo cuando un requisito funcional lo
   justifique; no se añade por defecto.
8. **Pruebas:** xUnit (backend) y Playwright (navegador/E2E).
9. **Comunicaciones:** segmentables por colectivos, categorías o equipos. La aplicación es el
   canal; WhatsApp es complemento, sujeto a viabilidad técnica, configuración y consentimiento.
10. **Fuera del alcance inicial:** gestión de instalaciones de terceros, venta de entradas a
   espectadores, comunicación directa entrenador–jugador desde el sistema y supervisión global
   del Director Deportivo (fase futura), salvo aprobación posterior.
11. **Entorno de desarrollo inicial:** Windows, PowerShell y Spec Kit. El plan documenta
    requisitos de ejecución y decisiones de almacenamiento, alojamiento e integraciones.

## Decisiones Pendientes

Se resuelven por registro formal (Principio IX) antes de implementar el flujo afectado:

- **Reparto de dominios sin dueño claro:** patrocinadores, tienda/ventas/descuentos/regalos,
  gastos, fichas federativas y comunicación segmentada con WhatsApp no están asignados a ningún
  grupo con certeza (G3 y G4 podrían solaparse). También: quién es propietario del panel
  general/navegación y de la web pública (¿G1?).
- **Confirmar el mapa de dependencias** entre G1–G4.
- **Catálogo único de roles.** Las fuentes usan nombres distintos: gestor del club /
  administrador, gestor de equipos, entrenador, jugador / deportista, familiar / tutor,
  socio, patrocinador, simpatizante, personal de material, Director Deportivo (futuro).
- **Tutor/familiar:** v0.2.0 establece que el tutor no tiene cuenta propia; el módulo base y
  el Módulo 3 contemplan perfil «familia/tutor» / `familiar` con permisos. Resolver.
- **OAuth 2.0:** seleccionar proveedor/servidor de autorización, flujo o grant type apropiado
  para cada tipo de cliente (PWA y web de gestión), estrategia de renovación/revocación de
  tokens y gestión de audiencias/ámbitos. Decidir si OpenID Connect se utilizará para la
  autenticación e identidad de usuario. G1 coordina esta decisión con los demás módulos.
- Consentimiento y tratamiento de datos de menores.
- **G2 — Ficha federativa:** decidir si la ficha federativa oficial (número/licencia, documentos,
  vigencia, renovaciones y trámites) pertenece a G2 o G3. Hasta su resolución, G2 solo mantiene
  la relación deportiva del deportista y no duplica la ficha administrativa.
- **G2 — Convocatorias:** decidir qué perfil crea, publica, cierra o reabre convocatorias y quién
  puede corregir una respuesta excepcionalmente. La documentación atribuye al entrenador una
  función de consulta y al responsable del club la comunicación de cambios, pero no concreta
  todos los permisos de creación.
- **G2 — Franjas y agenda:** acordar el modelo de franjas (recurrentes o puntuales), los datos de
  instalación/campo que mantiene el club, las reglas de solapamiento, cancelación y cambios, y
  cómo se registran las franjas que concede el tercero.
- **G2 — Composición deportiva:** decidir si un deportista puede pertenecer simultáneamente a
  varios equipos/modalidades y cómo se conserva el historial por temporada o categoría.
- **G2 — Disponibilidad/lesiones:** concretar el ciclo de vida de una declaración de lesión,
  cuándo deja de mostrarse como vigente y qué perfiles deportivos concretos pueden consultarla,
  manteniendo la minimización de datos sensibles.
- Motor de base de datos, alojamiento, pasarela de pago, proveedor de WhatsApp, copias de
  seguridad y observabilidad.

## Flujo de Desarrollo y Puertas de Calidad

1. **Especificar:** propósito, usuarios, alcance, reglas de negocio y criterios de aceptación
   verificables antes de implementar. Cambio de alcance ⇒ actualizar especificación y tareas.
2. **Aclarar** dudas que afecten a alcance, datos o flujos principales.
3. **Planificar contra la constitución** con comprobación explícita de principios; un conflicto
   se corrige, no se ignora.
4. **Descomponer** en tareas pequeñas y verificables vinculadas a requisitos, con dependencias y
   pruebas.
5. **Implementar por incrementos** y verificar los criterios afectados.
6. **Revisar** coherencia entre especificación, plan, tareas y resultado antes de entregar.

Cada pull request indica si modifica contratos, permisos, estructuras de datos o presentación
móvil/escritorio, y verifica: propiedad de datos y ausencia de dependencias directas, contrato
actualizado, permisos por rol, mocks y errores, flujo en móvil/escritorio con navegación
accesible, y documentación de cambios incompatibles.

**No se acepta una integración si:**
1. Permite acceder a datos fuera del rol o ámbito del solicitante.
2. Un mock difiere en forma de la respuesta real.
3. Un cambio incompatible no está versionado ni documentado.
4. Falta un estado de carga, vacío, error o confirmación.
5. Un endpoint no especifica IDs, campos obligatorios y errores esperados.
6. Hay pruebas relevantes fallidas, errores de compilación o incumplimientos conocidos de
   seguridad o accesibilidad.
7. Hay secretos en el código o datos personales en registros.

## Governance

Esta constitución prevalece sobre prácticas, plantillas y convenciones locales que la
contradigan, incluida cualquier constitución de módulo. Ante contradicción con una regla MUST,
prevalece hasta que se apruebe formalmente una enmienda.

- **Enmiendas:** cualquier miembro puede proponerlas con motivación e impacto en usuarios,
  datos, permisos, contratos, pruebas y trabajo existente. Requieren revisión y aprobación del
  responsable del proyecto o del equipo (y de los responsables de módulos afectados). Deben
  incluir plan de migración si afectan a trabajo entregado, actualizar la fecha de revisión,
  conservar la fecha de adopción inicial y comprobar la compatibilidad de los documentos
  dependientes.
- **Versionado (SemVer):** MAJOR para eliminar o redefinir principios o gobernanza de forma
  incompatible; MINOR para añadir principios/secciones o ampliar materialmente obligaciones;
  PATCH para aclaraciones sin cambio de obligaciones.
- **Cumplimiento:** cada revisión y entrega verifica los principios aplicables. Las excepciones
  no son tácitas: se documentan con motivo, alcance, riesgo, responsable, alternativa adoptada y
  fecha de revisión; si contradicen una regla MUST, requieren enmienda previa. Las decisiones
  pendientes no autorizan a implementar comportamiento no especificado.
- **Spec Kit:** tras una enmienda se revisan especificaciones, planes y tareas afectados.

**Version**: 1.2.0 | **Ratified**: TODO(RATIFICATION_DATE): pendiente de aprobación del equipo | **Last Amended**: 2026-11-11
