# Turnos App — Etapas y tareas

Plataforma web responsive para que profesionales independientes (coaching, tarot, astrología, reiki, etc.) gestionen agendas, reservas y cobros, y para que sus clientes reserven y paguen online sin fricción. Toma como referencia el relevamiento de enronda.com (noviembre 2026).

## Cómo usar este archivo con Trello

- Cada `## Etapa` es una **lista** de Trello.
- Cada `### Exx-yy` es una **tarjeta**. El código va al inicio del título para mantener el orden.
- Las etiquetas entre backticks (`Backend`, `Frontend`, `Pagos`…) son **labels** de Trello.
- Las casillas `- [ ]` son el **checklist** de la tarjeta.
- Las horas son de desarrollo puro: no incluyen gestión ni contingencia.
- Orden de ejecución: de E0 a E12 y, dentro de cada etapa, de arriba hacia abajo salvo que se indique una dependencia.

## Perfiles de usuario

| Perfil | Qué hace |
|---|---|
| **Anónimo** | Explora disciplinas, busca profesionales, ve perfiles, servicios y disponibilidad. Se registra recién cuando quiere reservar. |
| **Cliente** | Reserva, paga (Mercado Pago o transferencia con comprobante), completa datos previos, cancela o reprograma según la política y deja valoraciones. |
| **Profesional** | Paga una suscripción. Configura perfil, agendas, vacaciones, servicios y medios de cobro. Gestiona reservas, pagos y consultantes. |
| **Admin** | Dueño de la plataforma. Aprueba profesionales; ve clientes, la relación profesional ↔ clientes, reservas, planes, suscripciones y métricas. |

## Decisiones técnicas (resumen)

| Tema | Decisión |
|---|---|
| Backend | .NET 10 con DDD en capas (Domain / Application / Infrastructure / Web), monolito modular |
| Frontend | ASP.NET Core Razor Pages + htmx + Alpine.js (sin jQuery), Bootstrap 5, mobile-first. Un solo despliegue: no se separan back y front |
| API | Minimal APIs solo para webhooks (Mercado Pago, Telegram) y fragmentos htmx. La capa Application queda lista para exponer una API REST si más adelante hay app móvil |
| Base de datos | PostgreSQL con EF Core + Npgsql. SmarterASP.NET la incluye en todos sus planes; Somee solo ofrece SQL Server |
| Hosting | SmarterASP.NET: plan .NET Premium (incluye tareas programadas) o Advance + cron externo gratuito |
| Jobs | Hangfire con almacenamiento en Postgres + "tick" programado, porque en hosting compartido el app pool se apaga sin tráfico |
| Pagos | Mercado Pago Checkout Pro con OAuth (el dinero va directo a la cuenta del profesional) + Suscripciones de MP para el plan del profesional + transferencia con comprobante |
| Email | Brevo (300 emails/día gratis), con Resend como alternativa, detrás de una interfaz propia |
| Avisos al profesional | Telegram (gratis y oficial) + botón de WhatsApp prellenado para el cliente (gratis) + WhatsApp Cloud API opcional (≈ US$0,026 por aviso) |

## Resumen de etapas

| Etapa | Lista de Trello | Tareas | Horas |
|---|---|---:|---:|
| E0 | Descubrimiento y diseño | 9 | 43 |
| E1 | Fundaciones técnicas | 12 | 53 |
| E2 | Identidad y cuentas | 8 | 32 |
| E3 | Perfil profesional y catálogo público | 9 | 42 |
| E4 | Agendas, disponibilidad y vacaciones | 10 | 61 |
| E5 | Servicios y políticas | 8 | 38 |
| E6 | Reservas | 10 | 58 |
| E7 | Pagos | 9 | 47 |
| E8 | Notificaciones | 9 | 36 |
| E9 | Suscripción del profesional y onboarding | 5 | 26 |
| E10 | Panel de administración | 11 | 44 |
| E11 | Consultantes, valoraciones y dashboard | 4 | 19 |
| E12 | Calidad, seguridad y lanzamiento | 10 | 52 |
| **Total** | | **114** | **551** |

---

## Etapa 0 — Descubrimiento y diseño

**Objetivo:** cerrar alcance, modelo de dominio y pantallas antes de escribir código.
**Entregable:** wireframes navegables, glosario, diagramas de estado y ADRs aprobados.

### E0-01 · Validar requerimientos y cerrar el alcance del MVP
`Gestión` · **4 h**
- [ ] Repasar este documento y priorizar con MoSCoW
- [ ] Confirmar qué queda para la Fase 2 (ver final del documento)
- [ ] Definir monedas iniciales (ARS; USD opcional) y países
- [ ] Acordar criterios generales de aceptación: responsive, navegadores soportados, idioma

### E0-02 · Lenguaje ubicuo y modelo de dominio
`Arquitectura` · **6 h**
- [ ] Glosario español ↔ código: Reserva = Booking, Agenda = Schedule, Servicio/Sesión = Offering, Consultante = Client, Vacaciones = TimeOff
- [ ] Bounded contexts: Identidad, Catálogo, Agenda, Reservas, Pagos, Suscripciones, Notificaciones, Backoffice
- [ ] Agregados, value objects y eventos de dominio de cada contexto
- [ ] Diagramas en Mermaid dentro del repo

### E0-03 · Máquinas de estado de Reserva y Pago
`Arquitectura` · **3 h**
- [ ] Estados de la reserva: Solicitada, PendientePago, PagoEnRevisión, Confirmada, Realizada, Ausente, Cancelada, Vencida, Rechazada
- [ ] Efecto de cada política de pago pendiente sobre el bloqueo del horario
- [ ] Notificación que dispara cada transición

### E0-04 · Wireframes del sitio público
`UX` · **8 h**
- [ ] Home, búsqueda con filtros, perfil del profesional
- [ ] Wizard de reserva en móvil: fecha → horario → resumen → pago
- [ ] Registro y login

### E0-05 · Wireframes del panel del cliente
`UX` · **3 h**
- [ ] Mis sesiones: próximas, pendientes de pago, anteriores, canceladas
- [ ] Detalle de la reserva con carga de comprobante

### E0-06 · Wireframes del panel del profesional
`UX` · **8 h**
- [ ] Onboarding guiado y dashboard
- [ ] Agendas: editor semanal, vacaciones, calendario mensual
- [ ] Servicios: formulario con secciones condicionales
- [ ] Reservas, medios de cobro, consultantes, plan

### E0-07 · Wireframes del panel de administración
`UX` · **4 h**
- [ ] Dashboard de métricas
- [ ] Profesionales, clientes, relación profesional ↔ clientes, reservas, planes

### E0-08 · Identidad visual básica
`UX` · **4 h**
- [ ] Nombre comercial y logo provisorio
- [ ] Paleta, tipografía y tokens CSS
- [ ] Componentes base: botones, inputs, cards, badges de estado

### E0-09 · Registro de decisiones de arquitectura (ADRs)
`Arquitectura` · **3 h**
- [ ] ADR de frontend: Razor Pages + htmx vs Blazor vs SPA
- [ ] ADR de base de datos y hosting: PostgreSQL en SmarterASP.NET
- [ ] ADR de pagos (MP con OAuth y Checkout Pro) y de notificaciones (email + Telegram + WhatsApp opcional)

---

## Etapa 1 — Fundaciones técnicas

**Objetivo:** dejar una base sólida, desplegada y con CI antes de construir funcionalidades.
**Entregable:** solución DDD compilando, tests corriendo en CI y un "hola mundo" en el hosting real con Postgres.

### E1-01 · Repositorio y convenciones
`Infra` · **2 h**
- [ ] .editorconfig, .gitignore, Directory.Build.props (nullable, warnings como errores), Directory.Packages.props (versiones centralizadas)
- [ ] Estrategia de ramas (main + feature/*) y formato de commits

### E1-02 · Solución .NET 10 con capas DDD
`Backend` `Arquitectura` · **4 h**
- [ ] Proyectos `Turnos.Domain`, `Turnos.Application`, `Turnos.Infrastructure` y `Turnos.Web` en una solución .slnx
- [ ] Proyectos de test: `Turnos.Domain.Tests`, `Turnos.Application.Tests`, `Turnos.IntegrationTests`
- [ ] Test de arquitectura: Domain no referencia a ningún otro proyecto; Application solo referencia a Domain

### E1-03 · Building blocks de dominio
`Backend` · **4 h**
- [ ] Entity, AggregateRoot, ValueObject e IDs fuertemente tipados
- [ ] Eventos de dominio acumulados en el agregado
- [ ] Result/Error para fallos de negocio (sin excepciones para el flujo normal)
- [ ] Handlers de casos de uso sin MediatR (ahora tiene licencia comercial): interfaz propia o Mediator con source generators (MIT)

### E1-04 · Persistencia con EF Core y PostgreSQL
`Backend` · **6 h**
- [ ] DbContext, configuración por agregado, nombres de tablas y columnas en snake_case
- [ ] Migraciones y script idempotente para el despliegue
- [ ] Despacho de eventos de dominio al guardar (outbox simple en tabla)
- [ ] Auditoría automática: created_at, updated_at, created_by

### E1-05 · Spike de hosting en SmarterASP.NET
`Infra` · **6 h** · ⚠️ Riesgo alto: hacerlo antes de avanzar
- [ ] Abrir la cuenta de prueba (60 días gratis) y desplegar un "hola mundo" en .NET 10
- [ ] Conectar al PostgreSQL del hosting; validar el puerto (PgBouncer) y ajustar Npgsql si hace falta (Max Auto Prepare, No Reset On Close)
- [ ] Probar migraciones, escritura de archivos subidos, HTTPS y la tarea programada
- [ ] Documentar el resultado. Plan B si algo falla: Render o Railway + Neon

### E1-06 · CI/CD con GitHub Actions
`Infra` · **5 h**
- [ ] Pipeline de build + tests en cada PR
- [ ] Deploy a SmarterASP.NET (Web Deploy) desde main, con credenciales en secrets
- [ ] Ambiente de staging (subdominio) y de producción

### E1-07 · Logs, errores y health check
`Backend` · **3 h**
- [ ] Serilog a archivo con rotación
- [ ] Manejo global de excepciones y páginas de error amigables
- [ ] Endpoint /health con chequeo de la base de datos

### E1-08 · Layout responsive y componentes de UI
`Frontend` · **8 h**
- [ ] Layout público y layout de panel, con sidebar colapsable en móvil
- [ ] Bootstrap 5 + htmx + Alpine.js
- [ ] Modal, toast, validación de formularios, estados vacíos, indicadores de carga
- [ ] Tag helpers reutilizables: estado de reserva, precio, modalidad

### E1-09 · Jobs en segundo plano
`Backend` `Infra` · **5 h**
- [ ] Hangfire con almacenamiento en PostgreSQL y dashboard visible solo para Admin
- [ ] Endpoint /jobs/tick protegido con clave, llamado por la tarea programada del hosting o por cron-job.org
- [ ] Jobs idempotentes: si corren dos veces, no duplican efectos

### E1-10 · Almacenamiento de archivos
`Backend` · **4 h**
- [ ] IFileStorage con implementación en disco, en una carpeta fuera de wwwroot
- [ ] Validación del tipo real del archivo (magic bytes) y del tamaño; renombrado seguro
- [ ] Descarga autorizada: un comprobante solo lo ven el cliente, su profesional y el admin

### E1-11 · Datos semilla ficticios
`Backend` · **4 h**
- [ ] Disciplinas: Coaching, Tarot, Astrología, Reiki, Registros akáshicos, Numerología, Meditación, Yoga…
- [ ] 6 profesionales demo con perfiles creíbles (ej. "Coaching ontológico 60 min", "Tirada de tarot evolutivo", "Carta natal")
- [ ] Agendas, servicios, clientes y reservas demo, cargados solo en desarrollo y staging

### E1-12 · Fechas y zonas horarias
`Backend` · **2 h**
- [ ] IClock inyectable para tests deterministas
- [ ] Todo se guarda en UTC; la zona IANA se define por agenda (ej. America/Argentina/Buenos_Aires) y por usuario

---

## Etapa 2 — Identidad y cuentas

**Objetivo:** registro e ingreso simples para todos los perfiles.
**Entregable:** clientes y profesionales pueden registrarse, ingresar y gestionar su cuenta.

### E2-01 · Identity, roles y modo dual
`Backend` · **6 h**
- [ ] ASP.NET Core Identity con cookies y roles Admin, Profesional y Cliente
- [ ] Un mismo usuario puede ser cliente y profesional, con un selector "Usuario / Profesional"
- [ ] Políticas de contraseña y bloqueo por intentos fallidos

### E2-02 · Registro de cliente
`Backend` `Frontend` · **5 h**
- [ ] Nombre, apellido, email, celular con código de país, fecha de nacimiento (opcional) y aceptación de términos
- [ ] Confirmación de email
- [ ] Formulario corto: en móvil entra en una sola pantalla

### E2-03 · Login y recuperación de contraseña
`Backend` `Frontend` · **3 h**
- [ ] Login, logout, "olvidé mi contraseña" con link por email
- [ ] Recordarme y cierre de sesión en todos los dispositivos

### E2-04 · Login con Google
`Backend` · **3 h**
- [ ] Botón "Continuar con Google" y vinculación con una cuenta existente por email

### E2-05 · Registro y login dentro del flujo de reserva
`Frontend` · **3 h**
- [ ] Si un anónimo elige un horario, se registra o ingresa sin perder la selección

### E2-06 · Perfil del cliente
`Frontend` · **4 h**
- [ ] Edición de datos y preferencias de comunicación
- [ ] Eliminar cuenta con baja lógica y anonimización (Ley 25.326)

### E2-07 · Solicitud de alta de profesional
`Backend` `Frontend` · **5 h**
- [ ] Formulario: datos, disciplinas, formación y certificados (PNG, JPG o PDF)
- [ ] Queda en "Pendiente de aprobación" hasta que el admin la apruebe (E10-02)
- [ ] Al aprobarse: elige plan y paga la suscripción (E9)

### E2-08 · Autorización por políticas
`Backend` · **3 h**
- [ ] Políticas: dueño del recurso, rol, suscripción activa
- [ ] Tests de autorización para los endpoints sensibles

---

## Etapa 3 — Perfil profesional y catálogo público

**Objetivo:** que un visitante encuentre al profesional indicado y vea qué ofrece.
**Entregable:** home, buscador con filtros y perfiles públicos con buen SEO.

### E3-01 · Agregado Profesional
`Backend` · **5 h**
- [ ] Foto, descripción corta (140 caracteres), descripción larga, palabras clave, disciplinas
- [ ] Formación, años de experiencia, modalidad, país y provincia, idiomas, redes sociales, WhatsApp de contacto

### E3-02 · Pantalla "Mi perfil profesional"
`Frontend` · **6 h**
- [ ] Edición por secciones, recorte de foto y vista previa del perfil público
- [ ] Validación del celular con código de país y botón "probar en WhatsApp"

### E3-03 · Checklist de perfil completo y publicación
`Backend` `Frontend` · **3 h**
- [ ] Requisitos para aparecer en el buscador: foto, descripciones, un medio de cobro activo, un servicio publicado y suscripción activa
- [ ] Mensaje claro de lo que falta, con links directos a cada sección

### E3-04 · Perfil público con URL amigable
`Frontend` · **6 h**
- [ ] Slug editable (ej. /p/denise-coach)
- [ ] Datos, formación, servicios con duración, modalidad y precio, botón "Reservar"
- [ ] Botones para compartir y agregar a favoritos

### E3-05 · Home
`Frontend` · **5 h**
- [ ] Hero, grilla de disciplinas, profesionales destacados, cómo funciona, CTA "Soy profesional"

### E3-06 · Búsqueda con filtros
`Backend` `Frontend` · **8 h**
- [ ] Filtros: disciplina, modalidad, rango de precio, país y provincia, idioma
- [ ] Orden por relevancia, precio o valoración; paginación
- [ ] Filtros como chips en móvil, aplicados con htmx sin recargar la página

### E3-07 · Favoritos del cliente
`Frontend` · **2 h**
- [ ] Corazón en la card y en el perfil; listado en el panel del cliente

### E3-08 · SEO
`Frontend` · **3 h**
- [ ] Meta tags, OpenGraph para compartir por WhatsApp, sitemap.xml, robots.txt
- [ ] Datos estructurados (Person / Service)

### E3-09 · Páginas estáticas
`Frontend` · **4 h**
- [ ] Cómo funciona, FAQ, Términos, Privacidad
- [ ] Formulario de contacto con anti-spam (honeypot + rate limit)

---

## Etapa 4 — Agendas, disponibilidad y vacaciones

**Objetivo:** el corazón del sistema. Varias agendas por profesional con disponibilidad, vigencia y vacaciones correctas.
**Entregable:** el motor de turnos devuelve horarios correctos y está cubierto por tests.

### E4-01 · Agregado Agenda
`Backend` · **6 h**
- [ ] Nombre (ej. "Coaching", "Tarot"), zona horaria y estado (activa o inhabilitada)
- [ ] Disponibilidad semanal con varias franjas por día (ej. lunes 9 a 13 y 17 a 20)
- [ ] Descansos fijos (ej. almuerzo de 13 a 14)
- [ ] Invariantes: las franjas no se solapan y el inicio es anterior al fin

### E4-02 · Vigencia y visibilidad de la agenda
`Backend` · **3 h**
- [ ] "Sin fecha límite" o "visible hasta" una fecha (ej. 31/12), por si la disponibilidad va cambiando
- [ ] Fecha "desde" opcional, para una agenda que arranca más adelante
- [ ] El cliente nunca ve turnos fuera de la vigencia

### E4-03 · Vacaciones y ausencias
`Backend` · **6 h**
- [ ] Vacaciones del profesional que bloquean todas sus agendas. Mejora sobre enronda, donde se cargan agenda por agenda
- [ ] Ausencias de una sola agenda (ej. pausar solo Tarot)
- [ ] Rango de fechas, con horas opcionales y motivo interno
- [ ] Si hay reservas dentro del rango: listarlas y ofrecer reprogramarlas o cancelarlas con aviso al cliente

### E4-04 · Excepciones por día
`Backend` `Frontend` · **4 h**
- [ ] Cerrar un día puntual o cambiar su horario solo para esa fecha

### E4-05 · Motor de cálculo de turnos
`Backend` · **14 h**
- [ ] Entrada: servicio, agenda, rango de fechas y zona horaria del cliente
- [ ] Descuenta descansos, vacaciones, excepciones, reservas activas y retenciones temporales
- [ ] Respeta duración + descanso posterior, minutos de inicio (:00, :15, :30, :45), anticipación mínima y máxima, máximo de reservas por día y vigencia
- [ ] Devuelve los días con disponibilidad (para el calendario) y los horarios de un día

### E4-06 · Conflictos entre agendas
`Backend` · **3 h**
- [ ] Una reserva en una agenda bloquea ese horario en las demás agendas del mismo profesional: es una sola persona

### E4-07 · Tests del motor de turnos
`QA` · **6 h**
- [ ] Casos: cambio de mes, franjas múltiples, vacaciones parciales, anticipación de 3 días, cliente en otra zona horaria, máximo por día, vigencia vencida

### E4-08 · UI de agendas y editor semanal
`Frontend` · **8 h**
- [ ] Crear, editar y eliminar agendas; copiar el horario de un día a otros
- [ ] Vista previa: "así verá tu agenda el cliente"

### E4-09 · UI del calendario mensual del profesional
`Frontend` · **8 h**
- [ ] FullCalendar (o grilla propia) con reservas coloreadas por modalidad (presencial, virtual, híbrida) y por estado
- [ ] Resumen del día con acciones rápidas

### E4-10 · UI de vacaciones y bloqueos
`Frontend` · **3 h**
- [ ] Alta, edición y baja de vacaciones y ausencias, con aviso de las reservas afectadas

---

## Etapa 5 — Servicios y políticas

**Objetivo:** que cada profesional defina qué ofrece y con qué reglas.
**Entregable:** servicios publicados con modalidad, reglas de reserva, política de pago y prerrequisitos.

### E5-01 · Agregado Servicio
`Backend` · **5 h**
- [ ] Nombre, descripción, disciplina, imagen o ícono, duración, precio y moneda, opción gratuita
- [ ] Estados borrador, publicado y pausado; orden dentro del perfil
- [ ] Asociado a una agenda, o a dos si es híbrido

### E5-02 · Modalidades: presencial, virtual e híbrida
`Backend` · **5 h**
- [ ] Presencial: dirección, que se comparte al confirmar la reserva
- [ ] Virtual: link de videollamada
- [ ] Híbrida: el cliente elige al reservar; cada modalidad puede tener agenda, precio y reglas propias o compartidas

### E5-03 · Reglas de reserva
`Backend` · **3 h**
- [ ] Anticipación mínima configurable, de "sin anticipación" a 7 días (ej. 3 días para que una astróloga prepare la carta natal)
- [ ] Anticipación máxima en días, descanso posterior, máximo de reservas por día, minutos de inicio

### E5-04 · Política de pago
`Backend` `Pagos` · **5 h**
- [ ] Medios aceptados por servicio: Mercado Pago, transferencia, efectivo en la sesión
- [ ] Plazo para pagar: hasta el inicio, o 24, 48 o 72 h antes, o X días después de reservar (lo que ocurra primero)
- [ ] Qué pasa si no paga, a elección del profesional: (a) el horario queda retenido hasta el vencimiento; (b) no se retiene: sigue disponible y lo confirma el primero que paga; (c) se confirma igual, sin pago
- [ ] Los textos de los emails cambian según la política elegida

### E5-05 · Prerrequisitos del cliente
`Backend` `Frontend` · **5 h**
- [ ] Campos personalizados por servicio (texto, fecha, hora, lugar, archivo) y si son obligatorios
- [ ] Ejemplo precargado: fecha, hora y lugar de nacimiento para una carta natal

### E5-06 · Políticas de cancelación y reprogramación
`Backend` · **4 h**
- [ ] Horas mínimas para cancelar o reprogramar y cantidad de reprogramaciones permitidas
- [ ] Términos editables por el profesional, con plantilla por defecto: pago, reprogramación, cancelación, sin garantía de resultados
- [ ] El cliente acepta los términos al reservar y la aceptación queda registrada

### E5-07 · UI de alta y edición de servicio
`Frontend` · **8 h**
- [ ] Secciones condicionales con Alpine.js: la modalidad híbrida muestra dos bloques y la opción gratuita oculta el precio
- [ ] Vista previa de la card pública

### E5-08 · UI de listado de servicios
`Frontend` · **3 h**
- [ ] Ordenar con drag & drop y pausar o publicar con un interruptor

---

## Etapa 6 — Reservas

**Objetivo:** reservar en pocos toques y gestionar todo el ciclo de vida de la reserva.
**Entregable:** reserva completa de punta a punta, sin dobles reservas y con vencimiento automático.

### E6-01 · Agregado Reserva y máquina de estados
`Backend` · **6 h**
- [ ] Transiciones según E0-03, cada una validada en el agregado
- [ ] Eventos de dominio: ReservaSolicitada, ComprobanteSubido, PagoConfirmado, ReservaVencida, ReservaCancelada, ReservaReprogramada

### E6-02 · Anti doble reserva en la base de datos
`Backend` · **4 h**
- [ ] Constraint de exclusión en Postgres (tstzrange + btree_gist) por profesional, para reservas activas
- [ ] Mensaje claro si otra persona tomó el turno: "ese horario se acaba de ocupar"

### E6-03 · Wizard de reserva
`Frontend` · **12 h**
- [ ] Pasos: modalidad (si es híbrida) → día (calendario con días disponibles) → horario → resumen → login o registro → prerrequisitos → medio de pago → confirmar
- [ ] Antes de confirmar muestra precio, duración, política de cancelación y plazo de pago
- [ ] Pensado para usar con el pulgar: botones grandes y un paso por pantalla en móvil

### E6-04 · Retención temporal durante el checkout
`Backend` · **3 h**
- [ ] El horario elegido queda retenido 10 minutos mientras el cliente completa los pasos

### E6-05 · "Mis sesiones" del cliente
`Frontend` · **8 h**
- [ ] Pestañas: próximas, pendientes de pago, anteriores, canceladas o vencidas
- [ ] Detalle: estado, instrucciones de pago, cuenta regresiva del plazo, dirección o link, agregar al calendario (.ics)

### E6-06 · Cancelar y reprogramar desde el cliente
`Backend` `Frontend` · **5 h**
- [ ] Respeta la política del servicio (E5-06) y avisa al profesional

### E6-07 · "Reservas" del profesional
`Frontend` `Backend` · **10 h**
- [ ] Filtros por estado, agenda, servicio y fecha; alerta de "pagos para revisar"
- [ ] Acciones: confirmar pago, rechazar comprobante con motivo, cancelar, reprogramar, marcar realizada o ausente, compartir dirección o link

### E6-08 · Vencimiento automático
`Backend` · **3 h**
- [ ] Job que vence las reservas impagas al cumplirse el plazo, libera el horario y avisa a ambos

### E6-09 · Prerrequisitos pendientes
`Frontend` · **3 h**
- [ ] El cliente completa los datos desde su reserva; recordatorio si faltan 48 h antes de la sesión

### E6-10 · Reserva manual del profesional
`Frontend` `Backend` · **4 h**
- [ ] Cargar una reserva para un cliente que llegó por WhatsApp o teléfono (existente o nuevo por email)

---

## Etapa 7 — Pagos

**Objetivo:** cobrar sin intermediar el dinero: Mercado Pago inmediato o transferencia con comprobante.
**Entregable:** pagos por Mercado Pago confirmados por webhook y transferencias validadas por el profesional.

### E7-01 · Medios de cobro del profesional
`Pagos` `Frontend` · **5 h**
- [ ] Transferencia o billetera: nombre, titular, país, moneda, datos (banco, CBU/CVU, alias), instrucciones, activo
- [ ] Efectivo; vista previa del email que recibe el cliente
- [ ] Sin al menos un medio activo, la agenda no acepta reservas

### E7-02 · Conectar Mercado Pago (OAuth)
`Pagos` · **8 h**
- [ ] Botón "Conectar Mercado Pago" → flujo authorization code → access y refresh token guardados cifrados
- [ ] Renovación automática (el refresh token dura 6 meses) y desconexión
- [ ] Nunca pedirle al profesional que pegue su access token

### E7-03 · Checkout Pro
`Pagos` · **6 h**
- [ ] Preferencia creada con el token del profesional: el dinero se acredita directo en su cuenta
- [ ] Comisión de plataforma (marketplace_fee) configurable desde admin, 0 por defecto
- [ ] external_reference = id de la reserva; back_urls de éxito, pendiente y error; la preferencia vence con el plazo de pago

### E7-04 · Webhooks de Mercado Pago
`Pagos` `Backend` · **6 h**
- [ ] Endpoint público con validación de la firma x-signature (HMAC con la clave secreta)
- [ ] Idempotencia, respuesta en menos de 22 segundos y procesamiento en un job
- [ ] Consultar el pago en la API y confirmar la reserva cuando queda aprobado

### E7-05 · Conciliación de pagos
`Pagos` · **3 h**
- [ ] Job que revisa los pagos pendientes contra la API por si se perdió un webhook

### E7-06 · Transferencia con comprobante
`Pagos` `Frontend` · **6 h**
- [ ] El cliente sube el comprobante (JPG, PNG o PDF) desde su reserva y el estado pasa a "Pago en revisión"
- [ ] El profesional lo ve y lo confirma o rechaza con motivo; el cliente puede volver a subirlo
- [ ] Dispara el aviso "Dario hizo el pago por transferencia" (E8-05 y E8-07)

### E7-07 · Reembolsos
`Pagos` · **4 h**
- [ ] Mercado Pago: reembolso total o parcial por API
- [ ] Transferencia: registro manual del reembolso

### E7-08 · Pruebas con usuarios de prueba de Mercado Pago
`QA` `Pagos` · **4 h**
- [ ] Comprador y vendedor de prueba, tarjetas de prueba, casos aprobado, rechazado y pendiente

### E7-09 · Facturación del profesional
`Frontend` · **5 h**
- [ ] Ingresos del mes, por medio de cobro y por servicio; exportación a CSV

---

## Etapa 8 — Notificaciones

**Objetivo:** que nadie tenga que entrar a revisar: el sistema avisa solo.
**Entregable:** emails de todo el ciclo + avisos instantáneos al profesional.

### E8-01 · Infraestructura de notificaciones
`Notificaciones` `Backend` · **6 h**
- [ ] Interfaz INotificationChannel (Email, Telegram, WhatsApp) y orquestador por evento de dominio
- [ ] Cola en Hangfire, reintentos y registro de envíos: a quién, qué, cuándo y con qué resultado
- [ ] Preferencias por usuario y por canal

### E8-02 · Proveedor de email y dominio
`Notificaciones` `Infra` · **3 h**
- [ ] Brevo (300 emails/día gratis), con Resend como alternativa
- [ ] SPF, DKIM y DMARC en el dominio para no caer en spam

### E8-03 · Plantillas de email
`Notificaciones` `Frontend` · **4 h**
- [ ] Plantilla base responsive con la marca; textos editables desde admin (E10-09)

### E8-04 · Emails del ciclo de la reserva
`Notificaciones` · **6 h**
- [ ] Reserva solicitada, al cliente y al profesional
- [ ] Pago pendiente: "Para confirmar tu sesión tenés que pagar antes del {fecha}", con la aclaración según la política: horario retenido, no retenido o confirmada igual
- [ ] Recordatorio antes de que venza el plazo de pago
- [ ] Reserva confirmada, vencida, cancelada y reprogramada

### E8-05 · Emails de pagos
`Notificaciones` · **3 h**
- [ ] Al profesional: "Dario hizo el pago por transferencia", con el comprobante y botones Confirmar y Rechazar
- [ ] Al cliente: pago confirmado, comprobante rechazado (con motivo) o pago de Mercado Pago acreditado

### E8-06 · Recordatorio de sesión y pedido de valoración
`Notificaciones` · **2 h**
- [ ] Recordatorio 24 h antes, con link o dirección
- [ ] Pedido de valoración después de la sesión

### E8-07 · Bot de Telegram para profesionales
`Notificaciones` · **5 h**
- [ ] Vincular la cuenta desde "Mi perfil" con un deep link t.me/{bot}?start={token}
- [ ] Avisos instantáneos: nueva reserva, comprobante subido, pago de Mercado Pago acreditado, cancelación
- [ ] Gratis y oficial

### E8-08 · Botón "Avisar por WhatsApp"
`Frontend` · **1 h**
- [ ] Después de subir el comprobante, el cliente puede abrir WhatsApp con un mensaje prellenado para el profesional (wa.me). Costo cero y sin riesgo de bloqueo

### E8-09 · WhatsApp Cloud API (opcional, según el plan)
`Notificaciones` · **6 h**
- [ ] Cuenta de Meta Business, número dedicado y plantilla utility aprobada ("{cliente} hizo el pago por transferencia de {sesión}")
- [ ] Adaptador detrás de un feature flag; costo aproximado de US$0,026 por aviso en Argentina

---

## Etapa 9 — Suscripción del profesional y onboarding

**Objetivo:** monetizar la plataforma y que el profesional arranque solo.
**Entregable:** el profesional se suscribe, paga por Mercado Pago y queda publicado siguiendo un wizard.

### E9-01 · Planes de suscripción
`Backend` · **4 h**
- [ ] Plan definido por el admin: precio, moneda, período y días de prueba gratis
- [ ] Beneficios y límites por plan: cantidad de agendas, avisos por WhatsApp, estadísticas

### E9-02 · Suscripción con Mercado Pago
`Pagos` · **8 h**
- [ ] Un preapproval_plan por plan y checkout de suscripción para el profesional
- [ ] Webhooks subscription_preapproval y subscription_authorized_payment
- [ ] Período de prueba, reintentos de cobro y cancelación

### E9-03 · Reglas de acceso según la suscripción
`Backend` · **4 h**
- [ ] Sin suscripción activa: perfil oculto y sin reservas nuevas; las existentes se respetan
- [ ] Período de gracia configurable y emails de aviso (prueba por vencer, cobro fallido)

### E9-04 · Pantalla "Mi plan"
`Frontend` · **4 h**
- [ ] Plan actual, historial de cobros, cambio de plan y cancelación

### E9-05 · Onboarding guiado del profesional
`Frontend` · **6 h**
- [ ] Wizard: perfil → medios de cobro → primera agenda → primer servicio → publicar
- [ ] Barra de progreso y checklist, en lugar de un banner de "te falta completar"

---

## Etapa 10 — Panel de administración

**Objetivo:** que el dueño vea y gestione todo el negocio desde un solo lugar.
**Entregable:** backoffice con métricas, gestión de profesionales, clientes, reservas y planes.

### E10-01 · Dashboard de métricas
`Backend` `Frontend` · **6 h**
- [ ] Profesionales activos, altas nuevas, MRR, bajas (churn), reservas del mes, monto procesado, ocupación

### E10-02 · Gestión de profesionales
`Backend` `Frontend` · **8 h**
- [ ] Listado con búsqueda y filtros (estado, plan, disciplina)
- [ ] Ficha completa: perfil, plan, agendas, servicios, clientes y reservas
- [ ] Aprobar o rechazar solicitudes, suspender o reactivar, destacar en la home

### E10-03 · Gestión de clientes
`Backend` `Frontend` · **4 h**
- [ ] Listado y ficha con sus reservas y los profesionales con los que trabajó; bloquear

### E10-04 · Relación profesional ↔ clientes
`Backend` `Frontend` · **3 h**
- [ ] Qué clientes tiene cada profesional y viceversa, con filtros y exportación

### E10-05 · Reservas globales
`Frontend` · **4 h**
- [ ] Listado con filtros y detalle; intervenir (cancelar o anotar) ante un reclamo

### E10-06 · Planes y suscripciones
`Frontend` · **4 h**
- [ ] ABM de planes, estado de cada suscripción, cobros fallidos

### E10-07 · Disciplinas y palabras clave
`Frontend` · **2 h**
- [ ] ABM de disciplinas (con ícono y descripción) y de palabras clave sugeridas

### E10-08 · Moderación de valoraciones
`Frontend` · **2 h**
- [ ] Ocultar valoraciones ofensivas y dejar registro del motivo

### E10-09 · Contenidos y plantillas
`Frontend` · **4 h**
- [ ] FAQ, términos de la plataforma, plantilla de políticas por defecto y textos de las notificaciones

### E10-10 · Impersonación y auditoría
`Backend` · **5 h**
- [ ] "Entrar como" un profesional o cliente para dar soporte, con tiempo límite y registro
- [ ] Visor de auditoría: quién hizo qué y cuándo

### E10-11 · Configuración general
`Backend` · **2 h**
- [ ] Comisión de plataforma, valores por defecto (plazo de pago, anticipación) y feature flags

---

## Etapa 11 — Consultantes, valoraciones y dashboard

**Objetivo:** darle al profesional herramientas para seguir a sus clientes.
**Entregable:** CRM liviano, valoraciones públicas y un dashboard útil.

### E11-01 · "Consultantes" del profesional
`Frontend` `Backend` · **5 h**
- [ ] Listado con filtros (todos, nuevos, recurrentes) y ficha con historial de sesiones, pagos y prerrequisitos

### E11-02 · Notas privadas por consultante
`Frontend` · **3 h**
- [ ] Notas de seguimiento entre sesiones (coaching, tarot), visibles solo para el profesional

### E11-03 · Valoraciones
`Backend` `Frontend` · **5 h**
- [ ] El cliente califica de 1 a 5 y comenta después de una sesión realizada
- [ ] El profesional puede responder; el promedio se muestra en el perfil y en la card

### E11-04 · Dashboard del profesional
`Frontend` · **6 h**
- [ ] Sesiones del día y próximas, pagos para revisar, ingresos del mes, ocupación semanal

---

## Etapa 12 — Calidad, seguridad y lanzamiento

**Objetivo:** salir a producción con confianza.
**Entregable:** sistema probado con profesionales reales, con backups y monitoreo, en producción.

### E12-01 · Tests de integración
`QA` · **8 h**
- [ ] Testcontainers con Postgres para los flujos críticos: reservar, pagar, vencer, cancelar

### E12-02 · Tests E2E de humo
`QA` · **6 h**
- [ ] Playwright: reservar y pagar por transferencia, confirmar el pago, suscripción del profesional

### E12-03 · Seguridad
`QA` `Backend` · **5 h**
- [ ] Revisión OWASP, rate limiting, headers de seguridad, antiforgery, validación de archivos
- [ ] Tokens cifrados y secretos fuera del repositorio

### E12-04 · QA responsive y accesibilidad
`QA` `Frontend` · **5 h**
- [ ] Pruebas en celulares reales (Android e iOS), contraste, tamaños táctiles, navegación con teclado

### E12-05 · Rendimiento
`Backend` · **3 h**
- [ ] Índices, caché de búsquedas y perfiles, optimización de imágenes

### E12-06 · Backups
`Infra` · **3 h**
- [ ] pg_dump programado con copia externa y prueba de restauración

### E12-07 · Puesta en producción
`Infra` · **4 h**
- [ ] Dominio, SSL, credenciales productivas de Mercado Pago, email del dominio
- [ ] Monitoreo de disponibilidad (UptimeRobot) y de errores (Sentry, plan gratis)

### E12-08 · Textos legales
`Gestión` · **2 h**
- [ ] Términos, privacidad y cookies en borrador (la revisión de un abogado queda fuera de este alcance)

### E12-09 · Beta cerrada
`QA` `Gestión` · **12 h**
- [ ] 3 a 5 profesionales reales (ej. coaching y tarot) usan el sistema durante 2 semanas
- [ ] Corrección de los hallazgos

### E12-10 · Manuales y salida a producción
`Gestión` · **4 h**
- [ ] Guía corta para el profesional y para el admin
- [ ] Go-live

---

## Fase 2 — Backlog fuera del alcance inicial

Estimaciones gruesas, a confirmar cuando se prioricen.

| Funcionalidad | Horas aprox. |
|---|---:|
| Sincronización bidireccional con Google Calendar | 24 |
| Links automáticos de Zoom o Google Meet | 10 |
| Sesiones grupales con cupos | 16 |
| Sesiones recurrentes o procesos (semanal, quincenal, mensual) | 20 |
| Gift cards | 16 |
| Cupones de descuento | 10 |
| Stripe para cobrar en USD y EUR | 16 |
| Mensajería interna cliente ↔ profesional | 16 |
| Notificaciones push (PWA) | 8 |
| Estadísticas avanzadas (visitas al perfil, conversión) | 12 |
| Blog o contenidos del profesional | 12 |
| Test de "match ideal" para recomendar profesionales | 12 |
| Facturación electrónica (ARCA, ex AFIP) | 24 |
| App móvil (requiere una API REST) | A cotizar |
