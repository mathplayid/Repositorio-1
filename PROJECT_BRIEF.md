# NOVA STORE · Documento maestro

Versión 1.0 · 27 de octubre de 2026 · Idioma: español · Moneda: USD.

**Estado: especificación propuesta. No se ha construido, desplegado ni probado la aplicación.** Los criterios y comandos de este paquete describen trabajo futuro; no constituyen evidencia de implementación. No se han generado todavía las imágenes del catálogo.

## Cómo utilizar este paquete

Entregar los cinco archivos juntos a Codex. Este documento fija el producto; [ARCHITECTURE.md](ARCHITECTURE.md) fija reglas, seguridad y transacciones; [DATABASE.md](DATABASE.md) fija el modelo y el catálogo inicial; [DESIGN_SYSTEM.md](DESIGN_SYSTEM.md) fija experiencia y presentación; [ROADMAP.md](ROADMAP.md) fija entregas, pruebas y condiciones de salida.

| Solicitud | Documento responsable |
|---|---|
| 1. Visión; 2. Alcance; 3. Historias; 4. Navegación | PROJECT_BRIEF.md |
| 5. Diseño y sistema visual | DESIGN_SYSTEM.md |
| 6. Arquitectura; 8. Reglas y transiciones; 9. Carpetas; 12. Seguridad | ARCHITECTURE.md |
| 7. Base de datos, relaciones y datos iniciales | DATABASE.md |
| 10. Fases; 11. Pruebas; 13. Despliegue; 14. Terminado; 15. Riesgos | ROADMAP.md |

En caso de discrepancia, las reglas de negocio pertenecen a ARCHITECTURE, los campos y restricciones a DATABASE. No resolver contradicciones silenciosamente: corregir todos los documentos afectados y registrar la decisión. Los estados internos son códigos en inglés; toda interfaz y mensaje para el usuario estarán en español.

## 1. Visión

Crear una tienda de artículos de tecnología cotidiana, escritorio, hogar, movilidad y bienestar que permita aprender el ciclo completo de un ecommerce. Debe funcionar como un producto coherente: explorar, comparar, reservar productos, simular un pago, consultar un pedido y administrar sus consecuencias sobre inventario y métricas.

NOVA STORE tendrá identidad propia, sobria y editorial. Su carácter educativo será visible en una banda persistente, en el checkout, en los resultados y en los recibos. Texto común: **«NOVA STORE DEMO · Simulación educativa. No se realizan cobros ni envíos reales.»**

Audiencias: estudiante o visitante que prueba una compra; docente o evaluador que verifica escenarios; administrador que gestiona catálogo y operaciones ficticias. Se prioriza trazabilidad y consistencia frente a cantidad de funciones.

### Resultados observables

- Completar una compra ficticia y encontrarla después de recargar o volver a iniciar sesión.
- Demostrar aprobación, rechazo, cancelación del intento, caducidad del pedido y devolución simulada por cancelación antes del envío.
- Demostrar con dos compradores simultáneos que la última unidad solo se adjudica una vez.
- Explicar cada variación de existencias mediante un movimiento y cada cambio de estado mediante un evento.
- Crear, editar y archivar productos desde administración sin alterar pedidos históricos.
- Consultar gráficos calculados con datos persistidos, con rango de fechas y definiciones visibles.

## 2. Alcance y exclusiones

### Alcance de la versión completa

| Área | Incluido |
|---|---|
| Tienda | Inicio, catálogo paginado, búsqueda por nombre/SKU/descripción, categoría, rango de precio, disponibilidad, ordenación y ficha |
| Catálogo | 30 productos publicados, seis categorías, una unidad comercial/SKU por producto; al menos una imagen por SKU y varias imágenes en productos destacados |
| Identidad | Registro y acceso de clientes mediante credenciales de demostración, cierre de sesión, perfil básico, roles CUSTOMER y ADMIN |
| Carrito | Invitado persistente en navegador; cliente persistente en PostgreSQL; fusión controlada al iniciar sesión |
| Compra | Inicio de sesión obligatorio para confirmar; dirección ficticia; resumen cotizado en servidor; reserva de stock; envío simulado; un cupón |
| Pago | Simulador interno con opciones aprobar/rechazar/cancelar intento; resultados deterministas y reintentos antes de caducar |
| Pedidos | Historial, detalle, línea temporal, pago pendiente, cancelación autorizada, seguimiento ficticio y recibo imprimible DEMO |
| Administración | Dashboard, CRUD de productos/categorías/cupones, movimientos y ajustes de inventario, alertas, pedidos y configuración comercial |
| Calidad | Responsive, teclado y lector de pantalla, estados completos, pruebas de concurrencia, auditoría y documentación reproducible |

### Fuera de alcance

Cobros reales; SDK de pasarelas incluso en modo sandbox; campos de tarjeta, CVV, cuenta bancaria o identidad fiscal; facturas fiscales, impuestos legales o integraciones tributarias; paquetería real; emails/SMS reales; OAuth y recuperación de contraseña por correo; marketplace; múltiples monedas, almacenes o vendedores; variantes combinatorias; suscripciones; devoluciones posteriores al envío; reembolsos parciales; wishlist, reseñas, recomendaciones por IA, chat y programa de puntos.

No habrá checkout como invitado en esta versión. No habrá importación CSV ni carga arbitraria de archivos desde administración: se seleccionarán recursos de imagen previamente aprobados. No habrá editor de roles en la web. El administrador se provisiona mediante una operación de mantenimiento protegida.

**Simplicidad deliberada:** una aplicación, una base de datos, un almacén lógico y pagos locales. No introducir microservicios, colas externas, Redis o Elasticsearch sin una necesidad medida y una decisión documentada.

## 3. Historias de usuario y criterios de aceptación

Las historias se aceptan con comportamiento observable y estado final en base de datos. Las entregas E0–E6 se definen en ROADMAP. Una entrega no puede mostrar como disponible una función que todavía no ejecuta.

| ID / entrega | Historia | Criterios de aceptación |
|---|---|---|
| US01 / E1 | Como visitante, exploro productos | El catálogo muestra 30 productos publicados al completarse E1; precio USD, imagen, nombre y disponibilidad. Borradores/archivados no se pueden comprar. |
| US02 / E2 | Busco y filtro productos | Consulta de 1–80 caracteres, filtros y orden viajan en URL; recargar y volver atrás preserva selección; los filtros se combinan con AND; «Limpiar» restaura valores. Sin resultados ofrece una salida útil. |
| US03 / E1 | Evalúo una ficha | Descripción, especificaciones ficticias coherentes, imagen con alt y precio visibles; cantidad de 1 a 10; producto agotado no se agrega; un slug inexistente devuelve 404. |
| US04 / E1 | Accedo como cliente | La contraseña nunca aparece en respuestas/logs; errores de acceso genéricos; registro siempre crea CUSTOMER; logout elimina acceso; un usuario desactivado no puede operar con una sesión anterior. |
| US05 / E2 | Conservo mi carrito | Invitado conserva SKU/cantidad al recargar; cliente conserva carrito al cambiar navegador tras identificarse; la fusión es idempotente y comunica reducciones por límite/stock; el precio se lee del servidor. |
| US06 / E1 | Confirmo una compra ficticia | Checkout muestra «sin cobro real»; servidor recalcula precios y disponibilidad; cambios de precio/configuración producen una nueva cotización que debo aceptar; un envío repetido crea un solo pedido. |
| US07 / E3 | Aplico un cupón | Solo un cupón por pedido; se explican vencimiento, mínimo y límite; nunca genera total negativo; dos compras por la última utilización del cupón no exceden su cupo. |
| US08 / E3 | Elijo envío simulado | Estándar, exprés o retiro muestran coste y plazo ficticios; estándar gratis desde USD 80 netos de mercancía; cambiar método recalcula la cotización. |
| US09 / E1, E3 | Simulo un pago | E1 admite aprobación; E3 incorpora rechazo y cancelación de intento. El resultado lo decide el servidor a partir de un escenario permitido, sin números bancarios. Cada reintento queda registrado. |
| US10 / E3 | Recupero un pago fallido | Rechazo/cancelación de intento conserva pedido y reserva hasta el vencimiento original; al aprobar después existe una sola venta y un solo consumo de stock; no se extiende el plazo. |
| US11 / E1 | Consulto mis pedidos | Historial paginado, más reciente primero; detalle muestra importes y datos históricos; otro cliente no puede consultarlo cambiando la URL, ni descargar su recibo. |
| US12 / E3 | Cancelo dentro del plazo permitido | Cliente cancela PENDING_PAYMENT o CONFIRMED; ADMIN también PROCESSING. Se libera reserva o repone stock según corresponda. SHIPPED/DELIVERED bloquean la acción. Repetir no duplica efectos. |
| US13 / E1 | Obtengo un recibo de demostración | Disponible tras aprobación; identificador DEMO, importes persistidos y leyenda «SIN VALIDEZ FISCAL» también al imprimir. Una cancelación pagada conserva recibo y añade estado/reembolso simulado. |
| US14 / E4 | Administro productos | Solo ADMIN crea/edita/archiva; SKU y slug únicos; precio positivo; imagen aprobada obligatoria para publicar. «Eliminar» borra físicamente solo un borrador sin referencias. |
| US15 / E4 | Corrijo inventario | Ajuste exige cantidad, motivo y versión actual; no reduce onHand por debajo de reserved; movimiento y saldo se guardan juntos; agotado y stock bajo se calculan con disponible. |
| US16 / E4 | Opero pedidos | ADMIN avanza CONFIRMED → PROCESSING → SHIPPED → DELIVERED; servidor rechaza saltos y versiones obsoletas; cada transición guarda actor, fecha y motivo. |
| US17 / E5 | Entiendo resultados | Gráficos y tarjetas usan las mismas consultas/definiciones; sumar la tabla concuerda con el gráfico; rango sin actividad muestra cero; ningún pedido rechazado aumenta ventas. |
| US18 / E5 | Configuro reglas de demostración | ADMIN cambia tasa no fiscal, envío y umbral de alerta con validación y auditoría; se crea nueva versión; pedidos ya creados conservan sus importes. |
| US19 / E6 | Utilizo una interfaz accesible | Flujos principales completos con teclado; foco visible y restaurado en modales; errores asociados al campo; zoom 200% y ancho 320 px sin perder acciones. |
| US20 / E1, E3 | Confío en la consistencia | Dos compradores y una unidad: como máximo uno reserva; aprobar/cancelar/caducar en paralelo produce un único resultado válido; nunca inventario negativo ni doble reembolso. |
| US21 / E4 | Reviso lo ocurrido | Inventario, eventos de pedido y auditoría administrativa permiten seguir una operación mediante correlationId; ningún administrador cambia el historial desde la UI. |

## 4. Mapa de páginas y navegación

| Ruta | Acceso | Contenido y acciones |
|---|---|---|
| `/` | Público | Presentación, categorías, selección de productos y aviso DEMO |
| `/catalogo` | Público | `q`, `categoria`, `min`, `max`, `stock`, `orden`, `pagina`; 12 productos/página |
| `/categorias/[slug]` | Público | Entrada editorial a la misma consulta de catálogo, sin lógica duplicada |
| `/productos/[slug]` | Público | Galería, ficha, disponibilidad y agregar |
| `/carrito` | Público | Cantidades, eliminar, subtotal orientativo y acceso al checkout |
| `/acceso`, `/registro` | Público | Credenciales demo; retorno únicamente a una ruta interna permitida |
| `/checkout` | CUSTOMER o ADMIN comprador | Dirección ficticia, envío, cupón, cotización y crear pedido |
| `/checkout/[orderId]/pago` | Propietario | Reserva restante y escenarios de pago; recuperable tras recargar |
| `/cuenta` | Identificado | Alias y datos básicos ficticios |
| `/cuenta/pedidos` | Identificado | Solo pedidos propios |
| `/cuenta/pedidos/[orderId]` | Propietario | Estado, importes, intentos, seguimiento y acciones permitidas |
| `/cuenta/pedidos/[orderId]/recibo` | Propietario | Vista imprimible protegida; disponible según US13 |
| `/demo`, `/privacidad`, `/creditos` | Público | Cómo usar escenarios, tratamiento de datos demo y procedencia de imágenes |
| `/admin` | ADMIN | Indicadores y actividad |
| `/admin/productos`, `/admin/productos/nuevo`, `/admin/productos/[id]` | ADMIN | Listado y formularios de producto |
| `/admin/categorias`, `/admin/cupones` | ADMIN | Listados y formularios en página o diálogo |
| `/admin/inventario` | ADMIN | Saldos, filtros, alertas, ajuste y movimientos |
| `/admin/pedidos`, `/admin/pedidos/[id]` | ADMIN | Listado global y operación controlada |
| `/admin/pedidos/[id]/recibo` | ADMIN | Recibo autorizado desde contexto administrativo |
| `/admin/configuracion`, `/admin/auditoria` | ADMIN | Versiones comerciales e historial de acciones |

Cabecera de tienda: logo propio, Catálogo, categorías, búsqueda, cuenta y carrito con cantidad. En móvil: búsqueda disponible, menú accesible y acceso persistente al carrito sin cubrir contenido. Pie: aviso de simulación, privacidad y créditos. Administración usa un layout diferenciado con navegación lateral y retorno a tienda; no permite «hacerse pasar» por otro cliente.

Checkout: Carrito → Datos y envío → Revisar → Pedido reservado → Simular pago → Detalle. La creación del pedido ocurre al confirmar el resumen; la aprobación del pago es una operación posterior. No mostrar un pedido como confirmado antes de la aprobación persistida.

## 5. Supuestos aceptados para avanzar

- Se permiten cuentas con direcciones `@example.test`; no se verifica correo ni se envían comunicaciones.
- Se ofrecen direcciones de ejemplo autocompletadas; no se solicita teléfono, identificación fiscal ni datos financieros.
- Una reserva dura 15 minutos; cotización, 5 minutos; un usuario puede mantener como máximo tres pedidos pendientes no vencidos.
- No se editan líneas ni importes de un pedido creado. Para cambiarlo: cancelar si está permitido y crear otro.
- El tipo impositivo inicial es 0%; el 10% es exclusivamente un escenario de prueba, no una referencia a legislación de ningún país.
- El despliegue inicial será privado. Una demo pública compartida requiere controles adicionales descritos en ROADMAP.

## 6. Instrucción de arranque para Codex

```text
Lee PROJECT_BRIEF.md, ARCHITECTURE.md, DATABASE.md, DESIGN_SYSTEM.md y ROADMAP.md.
Son la especificación de NOVA STORE, una simulación educativa en español y USD.
Inspecciona primero el repositorio y sus AGENTS.md. No sobrescribas trabajo existente.
Empieza únicamente por E0 y después E1, en cambios pequeños y verificables.
Mantén el stack preferido salvo incompatibilidad demostrada y documentada.
No uses pagos, datos bancarios, facturas ni envíos reales.
Implementa cada entrega de extremo a extremo: datos, reglas, permisos, interfaz y pruebas.
La base de datos y el servidor son la autoridad para precios, estados y stock.
No sustituyas reglas de negocio por mocks fuera de pruebas o del simulador explícito.
No construyas entregas posteriores ni publiques un sitio sin una nueva instrucción de alcance.
Reporta al terminar: cambios, cómo ejecutarlos, pruebas realmente realizadas, resultados,
limitaciones y siguiente entrega. Distingue siempre planeado, implementado y verificado.
```

Este bloque sirve para una solicitud futura de construcción. **La solicitud actual termina con estos documentos.**
