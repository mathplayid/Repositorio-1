# NOVA STORE · Entregas, validación y despliegue

Versión 1.0 · 27 de octubre de 2026.

**Estado actual: planificación terminada en este paquete. Aplicación no construida, pruebas no ejecutadas y ningún despliegue realizado.** Todos los criterios siguientes son condiciones futuras de aceptación.

## 1. Método de trabajo

Cada entrega agrega un recorrido utilizable de extremo a extremo: datos, reglas, servidor, permisos, interfaz y pruebas. No terminar primero «todo el frontend» con botones ficticios y dejar transacciones para el final. La integridad mínima de inventario, dinero e identidad existe desde la primera compra.

E0 es una comprobación técnica previa breve; E1 es la primera tienda funcional. Las entregas posteriores amplían recorridos existentes. Una entrega puede dividirse en varios cambios pequeños, pero se acepta solamente cuando su recorrido completo funciona.

No fijar una fecha de finalización sin conocer experiencia del ejecutor, disponibilidad y entorno. Las entregas se estimarán después de E0; no usar un número de semanas inventado como compromiso.

| Entrega | Recorrido verificable | Dependencia | Evidencia de salida |
|---|---|---|---|
| E0 | Arrancar entorno, acceder con identidad de prueba y ejecutar una transacción contra PostgreSQL | Documentos aprobados como base | Matriz de versiones, smoke real y ADR |
| E1 | Ver catálogo → comprar → aprobar → consultar pedido/recibo | E0 | Demo mínima persistente y pruebas de stock/deduplicación |
| E2 | Buscar → filtrar → agregar como invitado → acceder → recuperar carrito | E1 | URL reproducible y persistencia/fusión verificadas |
| E3 | Cotizar con cupón/envío → fallar/reintentar/cancelar/vencer | E1; UI de E2 | Matriz de estados completa y pruebas de carreras |
| E4 | Admin crear producto → reponer → cliente compra → admin entrega | E3 | Operación administrativa aislada y auditada |
| E5 | Generar escenarios → analizar métricas → cambiar reglas para nuevas compras | E4 | Cifras reconciliadas y reglas versionadas |
| E6 | Ejecutar demo completa desde instalación limpia y entorno desplegable | E1–E5 | Accesibilidad, seguridad, restauración y entrega final documentadas |

La tabla no concede autorización actual para construir o publicar. Sirve de guía para la siguiente solicitud del proyecto.

## 2. Entregas verticales

### E0 · Compatibilidad y base reproducible

Objetivo: eliminar incertidumbre de versiones antes de extender la aplicación.

Trabajo: inspeccionar repo/AGENTS; elegir versiones mantenidas compatibles; fijar Node y gestor; configurar Next.js/TypeScript/Tailwind/shadcn mínimo; PostgreSQL de desarrollo y pruebas independientes; conexión Prisma y migración pequeña; acceso Credentials/JWT de prueba; variables validadas; primera prueba Vitest y Playwright.

Probar realmente: cliente ORM y migraciones con la misma versión; transacción y rollback; mecanismo de bloqueo/SQL parametrizado; login/logout y lectura de rol vigente; build de producción. Verificar mantenimiento/avisos de seguridad de dependencias. Si un paquete preferido no satisface compatibilidad o soporte, presentar evidencia y una alternativa mínima en ADR; no cambiar el stack por gusto.

Salida: README para arrancar, `.env.example` sin valores reales, lockfile, matriz de compatibilidad, decisión de estrategia Auth.js y evidencia del smoke. No desarrollar un framework interno ni ampliar alcance para «preparar» todo.

### E1 · Tienda mínima funcional y consistente

Recorrido: cliente inicia sesión → ve 30 productos → añade unidades → revisa total → crea pedido → aprueba pago demo → ve historial y recibo.

Trabajo de datos: catálogo/activos aprobados, User, Cart, Quote, Order/Items, Payment/Attempt, Inventory/Reservation/Movement, eventos e idempotencia. Configuración inicial sin cupón, impuesto 0% y envío estándar; esquema preparado para ampliar sin romper snapshots.

Servidor: registro CUSTOMER y acceso; carrito autenticado; cotización; confirmación atómica; pago APPROVE; caducidad durable y liberación. **Concurrencia, permisos y idempotencia no se posponen.** UI: catálogo simple, ficha, carrito, checkout, pago, cuenta y recibo imprimible con DEMO.

Comprobar: última unidad con dos clientes, repetición de confirmación y pago, recarga tras compra, reserva vencida, recibo ajeno bloqueado, persistencia y reconciliación. Al aprobar, inventario se reduce una vez. Si caduca sin pagar, se libera una vez.

Salida: US01/03/04/06/11/13 y parte esencial de US09/20. No mostrar cupones, escenarios o módulos admin todavía no funcionales. Los 30 recursos requieren la evidencia descrita en DATABASE; el paquete actual no los proporciona.

### E2 · Descubrimiento y continuidad de compra

Recorrido: visitante busca y filtra → revisa producto → crea carrito local → inicia sesión → combina carrito → completa compra.

Trabajo: filtros/orden/paginación en URL; categorías; búsqueda segura; carrito invitado; fusión idempotente; controles de cantidad con manejo de cambios; estados vacíos y errores de conectividad; mejor adaptación móvil.

Comprobar: filtros combinados y navegación atrás, enlace compartible, fusión repetida en dos pestañas, precios falsificados ignorados, stock cambiado entre sesiones y carrito persistente del cliente en otro navegador.

Salida: US02/05 y ampliación de US03/19. La fusión informa cada reducción o producto descartado; no oculta pérdida de líneas.

### E3 · Ciclo completo de compra simulada

Recorrido: aplicar cupón → elegir envío → confirmar → rechazar → reintentar y aprobar; segundo recorrido cancela o vence y verifica reposición.

Trabajo: cupones con reservas de uso y límites; tres envíos; configuración de impuesto no fiscal; escenarios DECLINE/CANCEL; límite de intentos; cancelación de pedido; reembolso completo ficticio antes de envío; histórico de intentos y eventos; mensajes de diferencia entre cancelar intento/pedido.

Comprobar: todas las filas de las máquinas de estados; transiciones no listadas rechazadas; redondeos; última utilización de cupón concurrente; carrera aprobación/cancelación/caducidad; reloj al límite; cambios de cotización; reenvío tras error de red después del commit.

Salida: US07/08/09/10/12 y US20 completas. Deben existir aserciones de estado de BD, no solo un mensaje verde en pantalla.

### E4 · Administración y cumplimiento del pedido

Recorrido: ADMIN publica un producto con imagen aprobada → ingresa stock → un cliente compra → ADMIN prepara/envía/entrega → el cliente consulta seguimiento.

Trabajo: layout/admin; productos y categorías; archivo/borrado restringido; selección de imágenes del Asset existente; cupones; inventario con motivos y alertas; listado/detalle de pedidos; transiciones, versiones esperadas y auditoría. Aprovisionamiento de ADMIN fuera del registro web.

Comprobar: cliente invocando directamente acción admin, intento de asignarse rol, edición concurrente, ajuste por debajo de reserved, archivo con pedido pendiente, cancelación admin de PROCESSING y bloqueo después de SHIPPED.

Salida: US14/15/16/21. Todo cambio administrativo relevante registra actor y diferencia, junto con el cambio.

### E5 · Análisis y configuración comercial

Recorrido: ejecutar escenarios con datos coherentes → consultar rango → reconciliar tarjetas/gráficos/tablas → cambiar tasa/envío → comprobar que solo cambia una nueva compra.

Trabajo: Recharts y tablas equivalentes; métricas según ARCHITECTURE; filtros por fechas con zona horaria; configuración versionada; auditoría consultable; seed de actividad separado; reporte de reconciliación. Agregar páginas /demo y /creditos definitivas.

Comprobar: pedidos pendientes fuera de aprobaciones; reembolso en fecha distinta; neto negativo en un período; rango vacío; frontera diaria Guayaquil/UTC; sums de filas = gráficos; snapshot histórico tras edición de producto/configuración.

Salida: US17/18. No inventar métricas de conversión ni series aleatorias para rellenar dashboards.

### E6 · Calidad y entrega operable

Recorrido: una persona instala desde cero, ejecuta tests, usa la demo completa y sigue un procedimiento de despliegue/restauración sin conocimiento del desarrollo.

Trabajo: revisión responsive/accesible; estados no ideales; rendimiento medido; revisión de seguridad; migraciones desde versión previa; documentación de operación, copia/restauración y jobs; capturas reales y demo guiada; CI y empaquetado listos.

Salida: todos los criterios de terminado; evidencias con fecha/versión/entorno. Corregir defectos de entregas previas; no añadir funciones comerciales nuevas. El despliegue efectivo se realiza únicamente cuando sea parte de la instrucción de construcción/publicación posterior.

## 3. Plan de pruebas automatizadas

### Capas y herramientas

- Vitest: reglas puras de dinero, transiciones, límites y validación; componentes síncronos con Testing Library cuando haya comportamiento relevante.
- Integración con PostgreSQL real: servicios, restricciones, transacciones, permisos y concurrencia. SQLite o mocks de Prisma no validan bloqueos PostgreSQL.
- Playwright: recorridos de navegador, accesos directos, sesión, navegación, responsive y componentes de servidor asíncronos.
- axe-core: comprobaciones automáticas en pantallas relevantes, complementadas con revisión manual. No declarar accesibilidad completa solo por «0 errores axe».

La [guía oficial de Next.js para Vitest](https://nextjs.org/docs/app/guides/testing/vitest), consultada el 27-10-2026, señala limitaciones con Server Components asíncronos; estos se cubrirán mediante E2E. Seguir [prácticas de Playwright](https://playwright.dev/docs/best-practices): pruebas aisladas y localizadores por rol/nombre cuando sea posible.

### Matriz de casos críticos

| ID | Prueba | Resultado esperado | Capa / entrega |
|---|---|---|---|
| T01 | S=10970, cupón10%, estándar, tasa demo10% | D=1097; H=0; T=987; total=10860; asignaciones exactas | Unidad/E3 |
| T02 | Redondeo justo por debajo/en mitad/encima; restos entre líneas | HALF_UP, suma distribuida exacta y desempate estable | Unidad/E3 |
| T03 | Cupón fijo mayor que S, mínimo, fecha exacta y 100% | Descuento acotado; ventanas semiabiertas; total cero admitido según reglas | Unidad/integración/E3 |
| T04 | Cualquier par de estados no permitido | Rechazo y cero cambios; pares permitidos cumplen precondiciones | Unidad/integración/E3 |
| T05 | Dos clientes, una unidad y dos cotizaciones | Un pedido reserva; otro falla por stock; onHand=1/reserved=1 | Integración concurrente/E1 |
| T06 | Carrito con dos SKU, segundo agotado | Ninguna reserva parcial, pedido, uso de cupón ni consumo de carrito | Integración/E1–E3 |
| T07 | Misma clave enviada en paralelo | Un resultado de negocio, un movimiento por causa, mismo pedido/intento | Integración/E1 |
| T08 | Misma clave con distinto payload | 409; no se ejecuta segundo efecto | Integración/E1 |
| T09 | Misma quote con claves distintas | Máximo un Order por quote | Integración/E1 |
| T10 | Fallo inyectado después de consumir stock antes de terminar pago | Rollback completo; reintento posterior seguro | Integración/E1 |
| T11 | Respuesta perdida después del commit | Mismo resultado al reenviar; sin nueva compra | Integración/E1 |
| T12 | Aprobación simultánea con caducidad y con cancelación | Solo combinación de estados permitida; venta o liberación/reposición una vez | Integración concurrente/E3 |
| T13 | Rechazo → reintento → aprobado / cancelar intento | Intentos inmutables; reserva no renovada; máximo un aprobado | Integración/E3 |
| T14 | Último uso global/por usuario del cupón en paralelo | RESERVED+REDEEMED no excede cupo | Integración/E3 |
| T15 | Cancelación pagada repetida incluso con clave diferente | Un Refund y una reposición por línea; uso de cupón sigue REDEEMED | Integración/E3 |
| T16 | Dos jobs sobre pedido vencido; job interrumpido | Liberación única; próxima ejecución completa lo pendiente | Integración/E1–E3 |
| T17 | Ajuste negativo mientras hay reserva | No deja onHand<reserved; conflicto visible | Integración/E4 |
| T18 | Cliente A pide detalle/recibo de B o llama función ADMIN | 404/403 sin datos ni efectos; probar también acceso directo HTTP | Integración/E2E/E1–E4 |
| T19 | Usuario con JWT previo tras desactivación/cambio de rol | Acceso invalidado según sessionVersion/rol DB | Integración/E0–E4 |
| T20 | Manipular precio, cantidad decimal/negativa, actor/role/status | Validación server-side; precios calculados por servidor | Integración/E1 |
| T21 | Seed base repetido y seed de escenarios repetido | Sin productos/movimientos/pedidos duplicados; saldos existentes intactos | Integración/E1–E5 |
| T22 | Snapshot tras cambiar precio, categoría, imagen o tasa | Historial/recibo conserva información original | Integración/E4–E5 |
| T23 | Fusión invitado repetida; dos pestañas con versión vieja | No duplica cantidades; informa conflicto/reducciones | Integración/E2E/E2 |
| T24 | Reembolso en otro mes, rango vacío y frontera diaria | Métricas según definiciones; tablas concuerdan | Unidad/integración/E5 |
| T25 | Checkout de teclado y móvil; error y resultado incierto | Flujo usable, foco correcto, reintento seguro | E2E/manual/E6 |
| T26 | Registro intenta ADMIN; mutación desde otro origen; input de script | Rol no escalado, CSRF bloqueado y texto escapado | Integración/E2E/E4–E6 |
| T27 | Transición admin con versión antigua o salto de estado | Conflicto sin sobrescritura ni auditoría de éxito falsa | Integración/E4 |
| T28 | Recibo de pedido cancelado tras aprobar, también en impresión | DEMO y SIN VALIDEZ FISCAL visibles; reembolso indicado | E2E/manual/E3–E6 |
| T29 | Inventario/Payment/Refund/cupones tras cada escenario | Reporte de reconciliación sin diferencias | Integración/E1–E6 |

Pruebas de carreras con dos conexiones reales y barreras controladas; no confiar en lanzar promesas sin garantizar solapamiento. Para fecha límite usar fixtures/reloj de servicio con comprobación equivalente a hora de DB; ningún modo de reloj alterable desde cliente llega a producción. Verificar también valores exactos inmediatamente antes, en y después de expiresAt.

Fallos inyectados solo en tests; nunca endpoints públicos para forzar rollback. Cada suite usa esquema/base aislada y limpia, con seed determinista y credenciales de prueba propias. No usar base compartida de staging como entorno de CI.

### CI y evidencia

Scripts previstos, a implementar y ajustar en E0: `lint`, `typecheck`, `test:unit`, `test:integration`, `test:e2e`, `build`, `db:migrate`, `db:seed`, `db:reconcile`. Son nombres de contrato, no comandos ya existentes ni ejecutados.

En cada cambio: instalación con lockfile congelado, lint/types, tests pertinentes, build. En integración continua: PostgreSQL temporal, migraciones desde cero, integración y E2E de flujos principales. Antes de release: Chromium/Firefox/WebKit y vistas móviles críticas; registrar diferencias reales. Guardar trazas y capturas de fallos sin secretos. No ocultar fallos con retries ilimitados o pruebas deshabilitadas.

Cobertura como señal: cada regla monetaria, transición y carrera crítica tiene casos explícitos; no exigir un porcentaje global que premie pruebas triviales. Cambios de dominio exigen ampliar sus casos. No crear tests que solo repitan literales del código.

## 4. Pruebas manuales

| Área | Procedimiento y evidencia |
|---|---|
| Experiencia | Compra completa sin instrucciones del desarrollador; verificar diferencias entre intento cancelado y pedido cancelado |
| Teclado | Tab/Shift+Tab, Enter, Espacio, Escape; modales, filtros, errores y vuelta de foco |
| Lector de pantalla | Un flujo completo con NVDA o VoiceOver; nombres y anuncios comprensibles; combinación probada anotada |
| Responsive | 320, 390, 768, 1024 y 1440 px; orientación, zoom 200%, textos largos y tablas |
| Estados | BD lenta, red interrumpida, sesión expirada, carrito vacío, producto agotado, validación y resultado de pago incierto |
| Recibo | Imprimir o guardar PDF desde navegador; marca DEMO visible y ninguna línea/total truncados |
| Operación | Detener job, dejar vencer reserva, reiniciar y verificar liberación única; simular reejecución |
| Datos | Revisión de las 30 fichas/imagenes, precio, categoría, alt y evidencia de procedencia |
| Seguridad | Cliente intentando rutas y acciones ADMIN; otro pedido; sin datos privados en HTML/caché/logs |

Registrar resultado real, entorno, versión del commit y defecto si lo hay. Una casilla pendiente no equivale a aprobada.

## 5. Condiciones de despliegue

### Entorno requerido

- Hosting que ejecute Next.js con Node.js y permita conexión a PostgreSQL, transacciones y dependencias nativas de hashing. **Una exportación estática no satisface esta aplicación.**
- PostgreSQL gestionado o administrado con copias y TLS; aplicación y DB en regiones cercanas; pool compatible con el ORM y transacciones utilizadas. Medir límites de conexiones y duración real.
- Job durable programado al menos cada minuto, invocando un comando/endpoint protegido; no depender de un proceso serverless que permanezca vivo.
- Recursos de producto empaquetados y servidos de forma estable; si el host exige almacenamiento externo, usar dominio controlado y lista permitida, sin alterar las reglas de procedencia.
- Tres entornos separados: desarrollo, pruebas/CI y demo persistente; staging adicional cuando se prepare publicación. Nunca compartir credenciales o datos entre CI y demo.

No se ha escogido proveedor ni presupuesto. Evaluar compatibilidad y costes antes de provisionar; no prometer alojamiento gratuito ni capacidades no comprobadas.

### Variables y secretos

`DATABASE_URL`, URL directa de migración si el proveedor la requiere, `AUTH_SECRET`, origen público/URL de Auth según la versión fijada, `CRON_SECRET`, `PAYMENT_MODE=simulated`, `DEMO_MODE=true`, `APP_TIMEZONE=America/Guayaquil`. Variables de seed: contraseñas demo/admin y seedEpoch, usadas solo durante aprovisionamiento; no mantener secretos de seed innecesarios en runtime.

Los nombres específicos de Auth/Prisma dependen de E0; no añadir aliases contradictorios de guías antiguas. `.env.example` describe propósito y obligatoriedad sin valores reales. Validar al arrancar; `DEMO_MODE=false` o modo de pago diferente no habilita pagos reales, sino que bloquea el inicio por configuración no admitida.

### Secuencia de publicación futura

1. Completar checks y build del mismo commit; revisar licencias y secretos.
2. Crear backup antes de migraciones sobre datos persistidos y demostrar restauración en una base separada.
3. Aplicar migraciones revisadas con usuario de migración; preparar cambios compatibles hacia delante. No ejecutar seed/reset destructivo automáticamente al iniciar.
4. Desplegar aplicación con variables/secretos, HTTPS y cabeceras; configurar job y verificar su autenticación.
5. Ejecutar smoke en el destino: login, compra demo, autorización negativa, caducidad, recibo y consulta admin.
6. Reconciliar datos; revisar logs, errores, pools y última ejecución exitosa del job.
7. Mantener acceso privado inicial. Abrir al público solo cuando sea parte del alcance autorizado y estén resueltos controles de abuso/datos.

Rollback: volver a build anterior solo si el esquema sigue siendo compatible. Preferir corrección hacia delante para migraciones incompatibles; una restauración de backup pierde operaciones posteriores y requiere decisión explícita del responsable. No ejecutar down-migrations destructivas como recuperación automática.

### Operación mínima

- Health check de proceso y readiness de conexión DB sin exponer credenciales o tablas.
- Alertar por errores repetidos de transacción, job con más de dos minutos de retraso, falta de conectividad y diferencias de reconciliación.
- Backup diario como propuesta inicial, retención de siete días; objetivo RPO 24 h/RTO 4 h para demo, sujeto a capacidad real del proveedor y prueba. No presentarlos como SLA medido.
- Logs técnicos 14 días, buckets de rate limit hasta fin de ventana + margen; quotes sin pedido y claves idempotentes sin referencias, purga después de siete días. Conservar historial y referencias de pedidos de la demo hasta reset explícito.
- Para demo pública no compartir una sola cuenta de cliente con datos visibles a desconocidos. Usar cuentas individuales, direcciones ficticias, límites y caducidad. No publicar credenciales ADMIN ni crear botón de acceso admin sin autenticación.

## 6. Definición de terminado

Una entrega está terminada cuando su recorrido funciona desde UI hasta DB, maneja fallos relevantes, aplica permisos, tiene pruebas pertinentes ejecutadas y documenta limitaciones. **La versión completa** requiere además:

- [ ] US01–US21 aceptadas y trazables a entregas/pruebas.
- [ ] 30 productos, seis categorías y 30 imágenes correctas con procedencia comprobada; seed reproducible y no destructivo.
- [ ] Catálogo, filtros, cuentas, carrito persistente, checkout, envíos, cupones y todos los escenarios demo operativos.
- [ ] Máquina de estados, límites y reglas monetarias aplicadas en servidor.
- [ ] Pruebas concurrentes verifican ausencia de stock negativo, doble compra, doble utilización indebida y doble reposición/reembolso.
- [ ] Pedidos y recibos conservan snapshots; no hay dato bancario ni documento fiscal válido.
- [ ] Administración aislada, auditada y con detección de versión obsoleta.
- [ ] Dashboard y tablas concuerdan con consultas y reconciliación.
- [ ] Teclado, lector, responsive, carga/error/vacío y recibo impreso revisados.
- [ ] Lint/types/build y suites pertinentes pasan; ningún fallo crítico conocido queda oculto.
- [ ] Migración limpia/desde versión previa, job, backup y restauración documentados y verificados en entorno apropiado.
- [ ] README permite ejecutar el proyecto; `.env.example` y matriz de versiones actualizados; credenciales ausentes del repo.
- [ ] Evidencia separa implementado/probado/pendiente; no quedan botones falsos, métricas inventadas ni TODO críticos en el flujo de compra.

«Listo para desplegar» y «desplegado» son estados diferentes. La aceptación técnica puede concluir con paquete reproducible y procedimiento probado en staging sin declarar que existe un sitio público.

## 7. Riesgos, mitigaciones y decisiones pendientes

| Riesgo o decisión | Base propuesta | Cómo se resuelve / momento |
|---|---|---|
| Compatibilidad cambiante Next/Prisma/Auth | Mantener stack; no mezclar APIs de versiones | Matriz y smoke E0; ADR de cualquier cambio justificado |
| Ciclo de mantenimiento de autenticación | Verificar versión mantenida/avisos antes de adoptar | E0; si no hay combinación segura, proponer sustitución mínima con evidencia |
| Reservas abandonadas inmovilizan stock | Caducidad 15 min y job cada minuto | Pruebas de recuperación E1/E3; límites de pedidos/usuario |
| Carreras o duplicados de red | Transacciones, CHECK/UNIQUE, locks y claves | Tests PostgreSQL reales desde E1 |
| Licencias/imágenes poco coherentes | Recursos originales/generados o licenciados individualmente | 30 entradas del manifiesto antes de publicar E1; no inventar permiso |
| Alcance demasiado grande | Un almacén/SKU, sin pagos externos ni variantes | Cambios fuera de alcance requieren decisión explícita |
| Confusión con tienda real | Aviso DEMO, botones de escenario, recibos sin validez | Revisión de copy y navegación directa/impresión E6 |
| Dashboard con cifras engañosas | Definiciones y fechas explícitas | Reconciliar con tabla de pedidos/reembolsos E5 |
| Coste y proveedor de alojamiento | Sin proveedor seleccionado | Resolver antes de provisionar; requisitos técnicos de esta sección |
| Acceso privado o público | Privado por defecto | Elegir antes de publicación; cuentas públicas aisladas y abuso controlado |
| Tratamiento de datos personales reales | Solicitar solo datos ficticios | Cambiar alcance y revisar privacidad si se requieren datos reales |
| Recuperación de cuenta | Fuera de v1; sin email real | Decidir después de versión completa si se necesita un flujo seguro |
| Stock contable descontado al aprobar | Decisión adoptada, no al enviar | No modificar sin migración de reglas/ledger y pruebas |
| Moneda/locale/fecha | USD, español, America/Guayaquil | Adoptado; no cambia por zona horaria del host |
| Importes de envío y tasa | 4,99/9,99/0; gratis desde 80 netos; tasa 0 | Configurable con versiones; no bloquea inicio |
| Diseño definitivo | Paleta clara y objetos cotidianos | Validación visual en E2/E6; modo oscuro aplazado |

Las decisiones de producto ya propuestas permiten empezar sin una ronda extensa de preguntas. Solo bloquear la implementación dependiente cuando falle una comprobación real de compatibilidad, autorización, integridad o disponibilidad de recursos esenciales.

## 8. Formato de cierre de cada entrega para Codex

```text
Entrega y commit:
Recorrido disponible:
Cambios de datos/reglas/UI:
Cómo ejecutarlo:
Pruebas realmente ejecutadas y resultados:
Evidencia de concurrencia/permisos cuando aplique:
Limitaciones, defectos y trabajo pendiente:
Cambios a decisiones/documentación:
Siguiente entrega propuesta:
```

No rellenar «probado» con pasos sugeridos. No atribuir a la aplicación funcionalidades que solo estén descritas en estos documentos.
