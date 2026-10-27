# NOVA STORE · Diseño y experiencia

Versión 1.0 · 27 de octubre de 2026 · Especificación visual, no interfaz implementada.

## 1. Dirección creativa

Identidad original: objetos cotidianos presentados con claridad editorial, fondos cálidos, tipografía limpia y un acento azul violeta. La sensación premium se consigue con jerarquía, fotografía coherente, alineación y espacio, no con animaciones constantes o efectos de vidrio que reduzcan legibilidad.

Marca tipográfica «NOVA STORE» acompañada de un símbolo geométrico propio inspirado en una órbita incompleta. No copiar logos, tarjetas, textos ni composiciones de una marca existente. La letra y el símbolo deben funcionar en monocromo y tamaños pequeños.

Tono: directo, cordial, español comprensible. Mantener «DEMO» visible incluso cuando la página se imprime o el usuario llega directamente a una ruta de pago. No usar testimonios, valoraciones o contadores falsos para simular confianza.

## 2. Tokens

| Token | Valor inicial | Uso |
|---|---|---|
| background | `#F7F7F2` | Fondo general cálido |
| surface | `#FFFFFF` | Tarjetas, paneles y formularios |
| foreground | `#172126` | Texto principal y títulos |
| muted-foreground | `#526068` | Metadatos y ayudas |
| primary | `#4338CA` | Acción principal y enlaces |
| primary-hover | `#3730A3` | Hover de acción |
| on-primary | `#FFFFFF` | Texto sobre primary |
| border | `#D6DCDE` | Separaciones decorativas |
| control-border | `#6B7780` | Bordes funcionales cuando se necesitan para identificar controles |
| success | `#146C43` | Aprobado, disponible y completado |
| warning | `#8A4B08` | Reserva por vencer, bajo stock |
| danger | `#B42318` | Error, rechazo y acción destructiva |
| info-surface | `#EEF2FF` | Avisos DEMO e información |
| focus | `#4338CA` | Anillo de foco con separación de la superficie |

Los colores son una propuesta que debe medirse sobre cada pareja real antes de aprobar E6. No asumir conformidad solo por elegir una paleta. Los estados llevan texto/icono además de color.

- Tipografía inicial: pila del sistema (`ui-sans-serif`, `system-ui`, sans-serif), sin dependencia remota. Si se añade una fuente, alojarla localmente con licencia registrada.
- Texto base 16 px, line-height 1.5; ayudas 14 px; etiqueta de campo 14–16 px. Evitar texto funcional de 12 px.
- Escala de títulos: 48/56 desktop y 32/38 móvil para portada; 32/40 para página; 24/32 para sección; 18/26 para tarjetas. `clamp` puede interpolar sin desbordar.
- Espaciado: 4, 8, 12, 16, 24, 32, 48, 64 y 96 px. Formularios con ritmo vertical constante.
- Radios: 8 px controles, 12 px tarjetas, 16 px paneles; chips redondeados. Sombras cortas y discretas; borde cuando aporta delimitación.
- Contenedor de tienda: máximo 1280 px, margen móvil 16–20 px, desktop 32–48 px. Administración puede usar 1440 px.
- Capas: contenido 0, cabecera 20, panel lateral 40, modal 60, avisos 80; centralizar tokens para evitar z-index arbitrarios.

Tema claro en v1. Un modo oscuro no es condición de terminado; solo se añade cuando todos los estados tengan contrastes probados.

## 3. Composición por pantalla

| Pantalla | Jerarquía y distribución |
|---|---|
| Inicio | Aviso DEMO → cabecera → titular editorial breve con producto destacado → seis categorías → selección de productos → explicación de simulación |
| Catálogo desktop | Título y número de resultados → búsqueda/orden → filtros laterales 240 px + cuadrícula de tres columnas; cuatro si hay espacio suficiente |
| Catálogo móvil | Búsqueda → botones Filtros/Ordenar → chips activos → dos columnas desde 360 px; una columna donde texto/controles lo requieran |
| Ficha | Galería 55% y compra 45% en desktop; móvil imagen, título/precio, datos y botón en orden natural; stock y cantidad próximos al CTA |
| Carrito | Líneas editables y resumen lateral; en móvil resumen después de líneas; no ocultar costes adicionales hasta el último paso |
| Checkout | Indicador de progreso, formulario principal y resumen estable; en móvil, resumen expandible cuyo total siempre se ve |
| Pago demo | Aviso prominente, pedido/total/reserva, tres opciones de escenario y resultado; apariencia claramente simulada, sin formularios bancarios |
| Pedido | Estado textual, siguiente acción válida, línea temporal, artículos/importes y recibo; separar intentos rechazados de estado del pedido |
| Dashboard | Rango de fechas, métricas definidas, serie temporal, distribución por estados, tabla y alertas de inventario |
| Listados admin | Título/acción principal, filtros, tabla paginada, acciones explícitas; selección masiva fuera de alcance |

Los filtros por precio incluyen campos numéricos etiquetados, no solo un slider. En móvil el panel de filtros ofrece Aplicar/Limpiar y restaura foco al cerrar. La URL se actualiza al aplicar y se preserva al navegar atrás.

No usar carruseles automáticos, popups promocionales, temporizadores de falsa escasez o navegación que dependa de hover. El temporizador de reserva refleja una fecha persistida real de la simulación.

## 4. Componentes y contratos visuales

Base shadcn/ui seleccionada: Button, Input, Label, Select, Checkbox, Dialog/AlertDialog, Sheet, Table, Tabs, Badge, Skeleton, Tooltip y notificaciones. Personalizar a los tokens de NOVA; mantener comportamiento accesible de sus primitivas y verificarlo al componerlas.

Componentes de producto: ProductCard, ProductGallery, Price, StockBadge, QuantityControl, CatalogFilters, CartLine, CheckoutSummary, DemoBanner, PaymentScenarioPicker, OrderTimeline, Receipt, InventoryAdjustmentForm y EmptyState.

- ProductCard: imagen 4:5 o 1:1 consistente por colección; categoría discreta, nombre, precio y acción. Toda la tarjeta no envuelve controles anidados dentro de otro enlace.
- Price recibe centavos y currency USD; jamás calcula totales. Precios comparativos solo cuando `compareAtPrice > unitPrice`, con contexto textual.
- QuantityControl expone nombre accesible de aumentar/disminuir y límites; no depende exclusivamente de botones diminutos.
- OrderTimeline muestra evento, fecha, actor visible por rol y motivo saneado. Los detalles de auditoría interna no se filtran a clientes.
- InventoryAdjustmentForm muestra saldo actual, reservado, cambio y saldo previsto, pide motivo y confirmación; el servidor valida la versión real.
- Receipt usa texto «RECIBO DEMO — SIN VALIDEZ FISCAL», sin identificación tributaria, autorización fiscal ni logos de entidades financieras. CSS de impresión repite aviso y evita cortes de líneas/artículos.

## 5. Estados y mensajes

| Situación | Tratamiento |
|---|---|
| Primera carga | Skeleton con dimensiones finales y `aria-busy`; no anunciar cada elemento |
| Agregar al carrito | Respuesta inmediata de estado pendiente; éxito anunciado una sola vez, contador actualizado; rollback claro si falla |
| Guardando formulario | Botón con texto «Guardando…», bloqueo de envío duplicado y campos conservados; idempotencia sigue siendo del servidor |
| Sin productos coincidentes | «No encontramos productos con esos filtros» + Limpiar filtros |
| Carrito vacío | «Tu carrito está vacío» + Explorar catálogo |
| Sin pedidos | «Todavía no tienes compras de demostración» + Ver productos |
| Sin métricas | Valores cero, rango visible y texto «No hay actividad en este período»; no datos inventados |
| Stock insuficiente | «La cantidad disponible cambió. Revisa tu carrito.» + cantidad actual si autorizada |
| Precio/configuración cambió | Comparación del resumen anterior y nuevo; CTA «Aceptar nuevo total» antes de crear pedido |
| Pago aprobado | «Pago simulado aprobado. No se realizó ningún cobro.» + Ver pedido |
| Pago rechazado | «Este escenario simula un rechazo. Puedes volver a intentarlo antes de que venza la reserva.» |
| Intento cancelado | «Cancelaste el intento. Tu pedido sigue pendiente hasta [hora].» |
| Pedido vencido | «La reserva venció y el pedido se cerró.» + Volver a agregar productos |
| Error técnico / sin conexión | Conservar formulario; «No pudimos confirmar el resultado. Reintentar» con la misma clave; consultar pedido para resolver resultado incierto |
| Conflicto administrativo | «Otra operación actualizó este registro. Recarga los datos antes de guardar.» No sobrescribir automáticamente |

Los errores de campo se muestran junto al campo y en un resumen enfocable. Notificaciones transitorias no son la única vía para conocer un error o un pago. Formularios largos preservan valores inocuos tras un fallo; nunca persistir contraseña.

## 6. Microinteracciones

- Hover/foco 120–160 ms; apertura de panel 180–220 ms; movimiento máximo discreto de 2–4 px donde aporte orientación.
- Usar opacity/transform cuando proceda, sin animar layout de forma que mueva el objetivo del usuario.
- Respetar `prefers-reduced-motion`: eliminar desplazamientos, celebraciones y loaders decorativos. Mantener texto de estado.
- No añadir esperas artificiales a las transacciones. Para enseñanza, una animación opcional refleja el tiempo real de la solicitud y nunca decide la aprobación.
- Reserva: reloj visual derivado de `expiresAt`; aviso a dos minutos; anuncios de lector de pantalla al entrar en umbral y al vencer, no cada segundo.
- Los botones de confirmar y cancelar muestran qué ocurre con unidades y pago ficticio antes de ejecutar.

## 7. Accesibilidad y responsive

Objetivo WCAG 2.2 AA en los flujos implementados, verificado con comprobaciones automáticas y manuales. No declarar certificación automática por usar shadcn o axe.

- `lang="es"`, landmarks, enlace para saltar al contenido, un h1 principal y encabezados coherentes.
- Todos los flujos por teclado, foco visible y sin trampas; modales atrapan foco durante su apertura, permiten cerrar según contexto y devuelven foco al activador.
- Contraste de texto normal al menos 4.5:1, texto grande 3:1 y componentes/indicadores pertinentes 3:1. No usar border decorativo claro como única identificación de un control.
- Objetivos táctiles propuestos de al menos 44×44 px para controles frecuentes; suficiente separación. Es una regla de producto, no una afirmación de que todo WCAG exija esa medida.
- Nombre accesible de iconos; imágenes con alt pertinente, decorativas con alt vacío; no escribir en alt información redundante de precio.
- Reflujo a 320 px, zoom 200%, cambio de orientación y texto largo; tablas administrativas pueden tener scroll local etiquetado, nunca desbordar toda la página.
- Gráficos con título, resumen textual, leyenda y tabla alternativa. Tooltip no será la única forma de conocer valores.
- Errores no dependen del color; `aria-describedby` y regiones de estado discretas. No anunciar continuamente el catálogo al mover filtros.
- Validar al menos un flujo con NVDA o VoiceOver, además de axe; reportar combinación de navegador y lector probada.

## 8. Imágenes y rendimiento visual

30 imágenes de producto, con procedencia aprobada según DATABASE. Dirección común: fondo neutro, luz suave, objeto completo y espacio de respiro. No borrar marcas de una foto de terceros para fingir autoría. Crear o elegir desde origen recursos adecuados.

WEBP/AVIF según soporte, tamaños explícitos y versiones responsive. Objetivo orientativo por imagen de tarjeta: 80–180 KB; imagen principal no más de 350 KB cuando la calidad lo permita. Usar el componente de imagen de Next.js correctamente configurado, sin lazy-loading para la imagen principal que determine LCP y con carga diferida para contenido inferior. Evitar cambiar dimensiones después de cargar.

Objetivos de evaluación en E6: LCP <=2,5 s, CLS <=0,1 e INP <=200 ms como metas de experiencia, medidos en condiciones documentadas. Lighthouse es evidencia de laboratorio; no equivale a métricas reales de usuarios. Fijar presupuestos después de la primera medición y registrar dispositivo, red, tamaño de catálogo y entorno.

## 9. Revisión visual antes de aceptar

Capturas de inicio, catálogo, ficha, carrito, checkout, resultado de pago y admin a 390 px y 1440 px; prueba adicional a 320 px. Verificar: texto no cortado, imágenes correctas, alineación de precios, acciones visibles, formularios sin saltos, errores legibles, tablas útiles y recibo impreso completo.

Probar loading/error/empty además del estado ideal. Aceptar con checklist y defectos registrados; no sustituir revisión visual por una única puntuación automática.

## 10. Referencias

- [WCAG 2.2, W3C](https://www.w3.org/TR/WCAG22/): referencia de accesibilidad consultada el 27-10-2026.
- Guías explicativas W3C de [contraste mínimo](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html), [foco visible](https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html) y [reflujo](https://www.w3.org/WAI/WCAG22/Understanding/reflow.html): criterios específicos para la revisión manual.
- [shadcn/ui con Next.js](https://ui.shadcn.com/docs/installation/next) y [Tailwind con Next.js](https://tailwindcss.com/docs/installation/framework-guides/nextjs): consultar instalación correspondiente a las versiones fijadas en E0.

La identidad, tokens y composiciones de este documento son propuestas propias del proyecto; no son una copia de esas guías.
