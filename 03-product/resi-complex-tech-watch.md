# Vigilancia Tecnológica — resi-complex

**Proyecto:** Residential-complex management (resi-complex) — Tecnólogo ADSO, SENA
**Equipo:** Alexandra (Metodología) · Félix (Entorno Tecnológico) · Fabián (Competencia y Patentes) · Julián (Introducción/Conclusiones/organización)

---

## 1. Introducción

La vigilancia tecnológica es el proceso sistemático de búsqueda, análisis y difusión de información sobre el entorno tecnológico y competitivo de un proyecto, con el fin de fundamentar decisiones técnicas antes de iniciar el desarrollo. Para **resi-complex** (sistema de gestión para conjuntos residenciales mixtos, residenciales y comerciales), este ejercicio respondió tres preguntas: qué soluciones ya existen en el mercado colombiano para este dominio, qué tendencias tecnológicas está adoptando ese mercado, y si existen patentes que restrinjan el desarrollo del sistema.

El trabajo se desarrolló en tres frentes complementarios: una **metodología** de vigilancia que identificó y validó cuatro plataformas activas en Colombia (ConjuntoApp, TUCO 360, Edifia, PH360); un **análisis del entorno tecnológico** que contrastó esas plataformas con el alcance de resi-complex para decidir con evidencia qué adoptar o descartar; y un **análisis de competencia y patentes** que confirmó que no existen restricciones legales de propiedad industrial para el proyecto. Cada hallazgo se tradujo en una decisión de alcance documentada, verificable por el equipo y el comité evaluador.

---

## 2. Metodología

La vigilancia se realizó siguiendo el ciclo clásico de inteligencia tecnológica: planificación, búsqueda y captación, tratamiento y análisis, y difusión de resultados, combinando vigilancia **competitiva** (actores del mercado y su propuesta de valor) y vigilancia **funcional** (funcionalidades y patrones relevantes para el dominio).

**Fases del proceso:**
- **Planificación:** se definieron ejes temáticos a partir de los módulos ya establecidos en resi-complex (acceso/portería, cuotas/pagos, mantenimiento, comunicados, correspondencia, asambleas/Junta, reportes, seguridad y datos personales).
- **Búsqueda:** consultas web dirigidas por eje temático, priorizando plataformas activas y con presencia comprobable en Colombia (relevante por la Ley 675 de 2001).
- **Selección de fuentes:** cada fuente debía ser pertinente, de origen oficial, vigente, con presencia en Colombia y describir funcionalidades concretas (no solo publicidad genérica).
- **Difusión:** los hallazgos se consolidan como insumo de discusión del equipo, no como decisiones automáticas.

**Alcance y limitaciones:** ejercicio documental, basado en información pública en línea; no incluyó pruebas directas de las plataformas ni acceso a documentación técnica interna de los proveedores. Los hallazgos son una aproximación funcional y de mercado, útil para orientar el alcance del proyecto formativo.

**Sistemas analizados:**

| Producto | Enfoque principal |
|---|---|
| **ConjuntoApp** | Cuotas, residentes, parqueaderos, accesos, PQR, asambleas, comunicados, documentos; pre-registro de visitas y control de paquetes |
| **TUCO 360** | Portería, cartera, mantenimiento, reservas y comunicación; registro de visitantes, órdenes de trabajo con fotos, anuncios segmentados, reportes |
| **Edifia** | Cartera, facturación, portería, visitantes, vehículos, correspondencia, comunicados, presupuesto, reservas, notificaciones, pagos; **incorpora IA para documentos** |
| **PH360** | Cartera, contabilidad, asambleas y portal del residente; asistencia, quórum y votaciones ponderadas |

**Comparación con resi-complex (resumen):**

| Aspecto | resi-complex | Competencia | Hallazgo |
|---|---|---|---|
| Cuotas diferenciadas | Sí | Sí, con automatización | Oportunidad: recordatorios y conciliación |
| Mantenimiento | Solicitudes, asignación, estados | PQR/órdenes con fotos | Oportunidad: evidencia fotográfica |
| Visitantes | Ingreso/salida manual | Pre-registro, QR | Oportunidad: preautorización/QR |
| Correspondencia | Registro y consulta | + aviso y confirmación de entrega | Oportunidad: notificación |
| Comunicados | Segmentado residencial/comercial | Segmentación general | Diferenciador de resi-complex |
| Gastos/Junta | Propuesta + aprobación con histórico | Presupuestos/asambleas | Diferenciador de resi-complex |
| Reservas, pagos en línea, asambleas, IA | No contemplados | Sí en varios | Líneas futuras posibles |

---

## 3. Análisis del Entorno Tecnológico

Construido directamente sobre los hallazgos de la sección 2: qué tendencias adoptar, adaptar o descartar en resi-complex, con criterio de relevancia y viabilidad (no "lo que más beneficia" de forma arbitraria).

### 3.1 Pagos y transacciones (blockchain / pagos en línea)
Ninguna de las 4 plataformas usa blockchain — **no se adopta**, no hay evidencia de mercado que lo respalde para este caso de uso. Los pagos en línea sí son comunes en la competencia — **se documentan como integración futura**, sin ser requisito de esta entrega.

### 3.2 Patrones de UX de la competencia
TUCO 360 y Edifia muestran preautorización/QR de visitantes; TUCO 360 permite fotos en mantenimiento; varias ofrecen reservas de zonas comunes.
- **Se adopta el principio, no la tecnología completa:** el modelo de datos de Visita queda preparado para evolucionar a QR, sin implementarlo ahora.
- **Se evalúa incorporar ya** la evidencia fotográfica en mantenimiento (bajo costo, extiende RF06).
- **No se adopta** el módulo de reservas: fuera de las entidades/RF ya definidos; línea futura.

### 3.3 Automatización e inteligencia artificial
Solo Edifia (1 de 4) tiene IA sobre documentos; varias automatizan cuotas/recordatorios; PH360 automatiza asambleas/votaciones.
- **No se adopta IA:** evidencia insuficiente (1 de 4 competidores) para tratarla como requisito; queda como tendencia a vigilar.
- **Se prioriza** automatización de recordatorios/estado de cuotas (RF08–RF10): funcionalidad recurrente en el mercado y viable con Spring Boot + tareas programadas.
- **No se adopta** el módulo de asambleas/votaciones: fuera del alcance actual (sin entidades ni RF definidos para ello).

### Síntesis de decisiones

| Hallazgo | Evidencia | Decisión |
|---|---|---|
| Blockchain | 0 de 4 | No se adopta |
| Pagos en línea | Varios | Integración futura |
| QR/preautorización | TUCO 360, Edifia | Prevista en el modelo, no implementada |
| Evidencia fotográfica | TUCO 360 | Evaluar incorporar ya |
| Reservas de zonas comunes | Varios | Línea futura |
| IA sobre documentos | 1 de 4 (Edifia) | No se adopta; vigilar |
| Recordatorios automáticos de cuota | Varios | Se prioriza (RF08–RF10) |
| Asambleas y votaciones | PH360 | Fuera de alcance actual |

---

## 4. Análisis de Competencia y Patentes

### 4.1 Plataformas de referencia
Mismas 4 plataformas de la sección 2. Ninguna resuelve de forma nativa la combinación de unidades residenciales **y** comerciales con tarifas diferenciadas, comunicados segmentados por tipo de unidad y distinción visitante personal vs. cliente de comercio — esa combinación sigue siendo la **oportunidad diferencial** de resi-complex, no una funcionalidad aislada.

### 4.2 Tecnologías emergentes y patentes
- Google Patents no arrojó patentes sobre sistemas de **gestión administrativa** de conjuntos residenciales. Sí existen patentes de **hardware** de control de acceso (cerraduras electrónicas, RFID/QR, torniquetes), relacionadas con la tendencia de preautorización vista en TUCO 360 y Edifia (sección 3.2).
- El módulo de portería de resi-complex es un **registro digital manual**, no un mecanismo físico automatizado — riesgo de solapamiento con esas patentes prácticamente nulo.
- En Colombia/CAN, **el software como tal no es patentable** — se protege por derecho de autor (Decisión 486 de la CAN). Una patente de software solo aplicaría si estuviera ligada a un efecto técnico novedoso sobre un dispositivo físico, lo cual no es el caso.
- La IA de Edifia (única entre los 4) tampoco está protegida por patente que restrinja su uso: es un diferenciador de producto, no una barrera legal.

**Conclusión:** el proyecto no requiere licenciamiento de patentes de terceros. Su forma de protección de propiedad intelectual aplicable sería el registro de derechos de autor sobre el software.

---

## 5. Conclusiones generales

1. El mercado colombiano está maduro en funcionalidades base, pero **ninguno de los 4 competidores** cubre unidades residenciales + comerciales con tarifas diferenciadas — ese sigue siendo el diferencial de resi-complex.
2. Cada decisión de alcance quedó **trazada con evidencia** de mercado, no por preferencia del equipo: se adoptaron mejoras de bajo costo y alineadas a los RF (fotos en mantenimiento, recordatorios de cuota); se descartaron o dejaron como futuras las que aparecen en pocos competidores o exceden el alcance formativo (IA, blockchain, asambleas, reservas).
3. **No existen restricciones de patentes** para el desarrollo del proyecto; el software se protege por derecho de autor, no por patente.
4. Quedan documentadas líneas de evolución futura (pagos en línea, QR de visitantes, IA sobre documentos) para retomar si el proyecto continúa más allá de esta entrega formativa.
5. Este ejercicio deja una base replicable: toda decisión futura de alcance debería seguir el mismo patrón — evidencia de mercado primero, decisión de diseño después.
