# PLAN DE IMPLEMENTACIÓN DETALLADO: SISTEMA DE GESTIÓN PARA ESTUDIO FOTOGRÁFICO

## 1. ESTRUCTURA DE CARPETAS Y ARQUITECTURA
La arquitectura sigue el patrón de capas limpias (Clean Architecture) adaptado para escalabilidad, mantenibilidad y gestión de estado con Provider. Cada capa tiene responsabilidades estrictas y flujos de dependencia unidireccionales.

```
lib/
├── core/
│   ├── constants/
│   │   ├── app_colors/
│   │   ├── app_routes/
│   │   └── app_strings/
│   ├── errors/
│   ├── network/
│   ├── theme/
│   └── utils/
├── data/
│   ├── datasources/
│   ├── models/
│   └── repositories/
├── domain/
│   ├── entities/
│   ├── repositories/
│   └── usecases/
├── presentation/
│   ├── providers/
│   ├── screens/
│   ├── widgets/
│   └── navigation/
└── main/
    └── app_entry/
assets/
├── images/
├── fonts/
└── icons/
config/
└── environments/
test/
├── unit/
├── widget/
└── integration/
```

**Descripción de capas:**
- `core`: Configuración global, constantes visuales, manejo de errores, cliente de red, utilidades de formato y navegación.
- `data`: Capa de persistencia y comunicación externa. Contiene adaptadores de API, mapeo JSON a modelos y repositorios concretos.
- `domain`: Lógica de negocio pura. Entidades inmutables, contratos de repositorios (interfaces) y casos de uso.
- `presentation`: Capa visual. Pantallas, componentes reutilizables, gestión de estado con Provider y enrutamiento.
- `main`: Punto de entrada, configuración de inyección de dependencias, inicialización de providers y arranque de la aplicación.
- `assets`, `config`, `test`: Recursos estáticos, variables de entorno por perfil y suites de prueba.

---

## 2. ENTIDADES DEL SISTEMA
Mapeo directo de las tablas de la base de datos a entidades de dominio, más entidades transversales necesarias para operación completa.

| Entidad de Dominio | Origen en BD | Responsabilidad en Aplicación | Atributos Clave (lógicos) |
|--------------------|--------------|-------------------------------|---------------------------|
| `Cliente` | `cliente` | Gestión de datos de usuarios finales, historial y preferencias | Identificador, nombre completo, contacto, fecha nacimiento, estado activo, fecha registro |
| `Fotografo` | `fotografo` | Gestión del talento humano, asignación por especialidad y disponibilidad | Identificador, contacto, especialidad, fecha contrato, estado activo |
| `Estudio` | `estudio` | Inventario de espacios físicos, características técnicas y disponibilidad | Identificador, nombre, descripción, capacidad, color fondo, área, estado disponible |
| `Paquete` | `paquete` | Catálogo comercial, precios, duración y alcance del servicio | Identificador, nombre, descripción, cantidad fotos, duración, precio, estado activo |
| `Reservacion` | `reservacion` | Agenda central, vincula cliente, paquete, estudio y fotógrafo | Identificador, fecha/hora, estado, canal origen, notas, relaciones con entidades base |
| `Sesion` | `sesion` | Ejecución operativa de la reservación, control de tiempos y avance | Identificador, referencia reservación, inicio/fin, estado operativo, conteo fotos, observaciones |
| `Equipo` | `equipo` | Gestión de activos técnicos por estudio, mantenimiento y trazabilidad | Identificador, estudio asociado, nombre, tipo, marca/modelo, serie, estado operativo, fechas revisión |
| `Pedido` | `pedido` | Documento comercial derivado de la sesión, base para facturación y entregas | Identificador, referencia sesión, subtotal, descuento, total, estado pago, fecha |
| `ProductoFinal` | `producto_final` | Entregables físicos o digitales generados por pedido | Identificador, tipo, formato, dimensiones, cantidad, precio unitario, enlace archivo, estado entrega |
| `Pago` | `pago` | Registro financiero, conciliación y trazabilidad de transacciones | Identificador, monto, método, referencia, fecha, estado confirmación |

**Entidades Transversales Adicionales (necesarias para operación completa):**
- `UsuarioSistema`: Credenciales, roles, permisos y sesión activa (admin, fotógrafo, recepcionista).
- `Notificacion`: Plantillas, estados de envío, historial de alertas por reservación/pago/entrega.
- `RolPermisos`: Matriz de acceso a módulos (lectura, escritura, aprobación, auditoría).
- `ArchivoAdjunto`: Metadatos de imágenes, contratos, comprobantes y entregables digitales.
- `ConfigSistema`: Parámetros globales (impuestos, moneda, zona horaria, políticas de cancelación).

---

## 3. PALETA DE COLORES Y DIRECTRICES DE UI
Los colores proporcionados se aplicarán bajo principios de accesibilidad (contraste WCAG AA), jerarquía visual y coherencia de marca.

| Color | Hex | Uso Recomendado | Contexto de Aplicación |
|-------|-----|-----------------|------------------------|
| Beige Cálido | `#D4CFC3` | Fondos de tarjetas, separadores sutiles, estados inactivos suaves | Componentes secundarios, áreas de descanso visual, bordes de formularios |
| Azul Oscuro Neutro | `#384057` | Texto principal, encabezados, iconos estructurales | Títulos, etiquetas, descripciones largas, navegación de texto |
| Gris Medio | `#B3B3B3` | Texto secundario, placeholders, bordes deshabilitados, divisores | Campos vacíos, ayudas de formulario, líneas de separación, estados no interactivos |
| Gris Claro/Off-White | `#E8E8E6` | Fondos de pantalla, áreas de entrada de datos, modales | Pantallas principales, inputs, listas scroll, fondos de dashboards |
| Azul Profundo | `#2E3B6E` | Acciones primarias, estados activos, enlaces, indicadores de progreso | Botones principales, pestañas seleccionadas, íconos interactivos, barras de navegación, validación positiva |

**Directrices de aplicación:**
- Jerarquía: `#2E3B6E` para interacción prioritaria, `#384057` para lectura, `#E8E8E6` para espacio negativo, `#D4CFC3` para contención, `#B3B3B3` para contexto secundario.
- Accesibilidad: Nunca usar `#B3B3B3` sobre `#E8E8E6` para texto crítico. Mantener contraste mínimo de 4.5:1.
- Estados: Éxito/Confirmación se mantiene en `#2E3B6E` o variantes claras; advertencias y errores usarán paleta complementaria no incluida aquí, pero respetando la neutralidad base.

---

## 4. ESTRATEGIA DE GESTIÓN DE ESTADO CON PROVIDER
Se utilizará Provider como capa de orquestación entre presentación y dominio, manteniendo separación estricta de responsabilidades.

**Arquitectura de Providers:**
- `AppProvider`: Estado global (tema, idioma, configuración sistema, conexión de red).
- `AuthProvider`: Sesión, permisos, rol activo, token, redirecciones.
- `CatalogoProvider`: Agrupa Estudio, Paquete y Fotografo. Maneja carga, filtrado y cache local.
- `ClienteProvider`: CRUD de clientes, búsqueda, historial vinculado.
- `ReservacionProvider`: Creación, validación de disponibilidad, cambios de estado, conflictos de agenda.
- `SesionProvider`: Control en tiempo real de la sesión, conteo de fotos, observaciones, transición a pedido.
- `PedidoFinanzasProvider`: Agrupa Pedido y Pago. Cálculo de totales, estados de cobro, generación de comprobantes.
- `EntregablesProvider`: Gestión de ProductoFinal, estados de procesamiento, enlaces, marca de entregado.
- `InventarioEquipoProvider`: Registro de activos, mantenimiento, asignación por estudio, alertas de revisión.

**Flujo de datos con Provider:**
1. La presentación solicita datos o acciones a los Providers.
2. Los Providers invocan Casos de Uso del dominio.
3. Los Casos de Uso interactúan con Repositorios (contratos).
4. Los Repositorios concretos en `data` comunican con API o caché local.
5. Las respuestas se mapean a Entidades y se notifican a la UI mediante `notifyListeners` o flujos derivados.
6. Se implementa `MultiProvider` en el nivel raíz para inyección jerárquica. Los providers hijos dependen de padres cuando requieren datos compartidos (ej: `ReservacionProvider` necesita `CatalogoProvider` para validar disponibilidad).

---

## 5. PLAN DE IMPLEMENTACIÓN DETALLADO POR FASES

### FASE 0: Configuración Inicial y Cimientos Arquitectónicos
**Dependencias:** Ninguna (inicio del proyecto)
- Definición de variables de entorno y perfiles (desarrollo, staging, producción).
- Configuración de cliente HTTP, interceptores de autenticación y manejo de errores de red.
- Implementación de estructura de carpetas y aliases de rutas internas.
- Creación de `AppTheme` con la paleta proporcionada y tipografía base.
- Configuración de enrutamiento base y navegación global.
**Entregables:** Repositorio inicializado, arquitectura vacía funcional, tema aplicado, rutas base.

### FASE 1: Capa de Dominio y Contratos de Repositorio
**Dependencias:** Fase 0 completada
- Definición de todas las Entidades de dominio (10 tablas + 5 transversales).
- Creación de interfaces de repositorios con métodos de negocio (obtener, crear, actualizar, filtrar, validar).
- Implementación de Casos de Uso por módulo (ej: `CrearReservacion`, `ValidarDisponibilidadEstudio`, `CalcularTotalPedido`).
- Validaciones de negocio (formatos, rangos, reglas de estado, concurrencia de agendas).
**Entregables:** Dominio completo, contratos listos para implementación, reglas de negocio documentadas y aisladas.

### FASE 2: Capa de Datos y Modelos
**Dependencias:** Fase 1 completada
- Creación de modelos de transferencia (mapeo JSON → Entidades).
- Implementación de fuentes de datos remotas (endpoints por entidad).
- Implementación de caché local para catálogo y configuraciones.
- Desarrollo de repositorios concretos que unen fuentes de datos con contratos de dominio.
- Maneo de errores de red, timeouts y respuestas parciales.
**Entregables:** Capa de datos operativa, mapeos funcionales, repositorios listos para consumo por providers.

### FASE 3: Integración de Provider y Orquestación
**Dependencias:** Fases 1 y 2 completadas
- Creación de `MultiProvider` en `main` con inyección jerárquica.
- Implementación de cada Provider por módulo con estados de carga, éxito y error.
- Conexión de Providers con Casos de Uso y repositorios.
- Implementación de mecanismos de actualización en cascada (ej: al confirmar reservación, actualizar disponibilidad estudio y fotógrafo).
- Pruebas de flujo de datos unitarias por Provider.
**Entregables:** Estado global y por feature funcional, flujo de datos bidireccional UI ↔ dominio, manejo de estados de interfaz estandarizado.

### FASE 4: Desarrollo de Módulos Críticos (Agenda y Operación)
**Dependencias:** Fase 3 completada, catálogo cargado
- Módulo Cliente: registro, búsqueda, historial vinculado.
- Módulo Reservacion: selector de fecha/hora, validación de solapamientos, asignación de recursos, estados de ciclo de vida.
- Módulo Sesion: inicio/fin, registro de fotos tomadas, observaciones, transición automática a pedido.
- Validación de reglas de negocio en tiempo real (capacidad estudio, especialidad fotógrafo vs paquete, disponibilidad equipo).
**Entregables:** Flujo completo de agendamiento y ejecución operativa, validaciones robustas, navegación entre pantallas clave.

### FASE 5: Módulos Comerciales y Financieros
**Dependencias:** Fase 4 completada, sesion finalizada
- Módulo Pedido: cálculo automático de subtotal/descuento/total, generación de documento comercial.
- Módulo Pago: registro de transacciones, conciliación de estados, soporte multi-método.
- Módulo Entregables: definición de producto final, gestión de archivos, estados de procesamiento/entrega.
- Integración con `NotificacionProvider` para alertas de pago y entrega.
**Entregables:** Ciclo de venta completo, trazabilidad financiera, gestión de entregables vinculada a pedido y sesión.

### FASE 6: Inventario y Mantenimiento
**Dependencias:** Fase 5 en paralelo o posterior (no bloqueante)
- Módulo Equipo: registro, asignación por estudio, estados operativos.
- Programación de revisiones y alertas de mantenimiento.
- Historial de uso por sesión y trazabilidad de fallos.
**Entregables:** Control de activos operativo, reportes de disponibilidad, integración con agenda de sesiones.

### FASE 7: UI/UX Final, Navegación y Pulido
**Dependencias:** Fases 4, 5 y 6 completadas, providers estables
- Construcción de pantallas finales aplicando paleta `#D4CFC3`, `#384057`, `#B3B3B3`, `#E8E8E6`, `#2E3B6E`.
- Componentes reutilizables (tarjetas, formularios, listas, indicadores, modales).
- Animaciones de transición, estados vacíos, carga progresiva.
- Accesibilidad, contraste, soporte para orientación y tamaños de fuente.
- Navegación profunda y restauración de estado al reinicio.
**Entregables:** Aplicación visualmente coherente, navegación fluida, experiencia de usuario validada.

### FASE 8: Pruebas, Optimización y Despliegue
**Dependencias:** Fase 7 completada
- Pruebas unitarias de dominios y casos de uso.
- Pruebas de integración de providers y flujos de datos.
- Pruebas de interfaz y recorrido de usuario completo.
- Optimización de rendimiento (carga diferida, cache, reducción de rebuilds).
- Configuración de CI/CD, generación de builds, firma y distribución.
**Entregables:** Aplicación probada, optimizada, empaquetada y lista para producción.

---

## 6. COMPONENTES TRANSVERSALES NECESARIOS
Para garantizar operación profesional, se incorporan los siguientes elementos no visibles en el esquema SQL pero críticos para el plan:

- **Motor de Autorización:** Validación de permisos por rol antes de ejecutar casos de uso. Intercepción a nivel de provider y repositorio.
- **Sistema de Notificaciones:** Cola de eventos para correos/SMS/push al cambiar estado de reservación, pago o entrega. Integración con servicios externos.
- **Gestor de Archivos:** Subida, compresión, almacenamiento seguro y generación de enlaces temporales para productos finales digitales.
- **Auditoría y Logs:** Registro inmutable de acciones críticas (creación, modificación, eliminación, cambios de estado) con huella de usuario y timestamp.
- **Validación de Reglas de Negocio en Cascada:** Prevención de reservaciones dobles, bloqueo de estudios/fotógrafos ocupados, validación de capacidad vs paquete, control de stock de entregables.
- **Manejo de Estado Offline:** Cache de catálogo y reservaciones pendientes, sincronización en reconexión, indicadores de conectividad.
- **Internacionalización y Localización:** Soporte para formatos de fecha, moneda, zona horaria y mensajes de error en múltiples idiomas.
- **Backup y Recuperación:** Estrategia de respaldo de datos críticos y política de retención de registros financieros y entregables.

---

## 7. MATRIZ DE DEPENDENCIAS TÉCNICAS Y DE FLUJO

| Elemento | Depende de | Bloquea a | Notas de Implementación |
|----------|------------|-----------|-------------------------|
| Tema y Constantes | Configuración inicial | Todas las pantallas | Base visual y de navegación |
| Entidades de Dominio | Estructura de carpetas | Casos de uso, Providers | Inmutables, sin lógica de red |
| Casos de Uso | Entidades, Contratos Repositorio | Providers | Validan reglas antes de persistir |
| Repositorios Concretos | Casos de uso (implícito), Modelos JSON | Providers | Manejan errores y transformaciones |
| Providers | Repositorios, Casos de uso | Pantallas, Navegación | Orquestan estado y notificaciones UI |
| Módulo Cliente | CatalogoProvider, AuthProvider | ReservacionProvider | Base para agendamiento |
| Módulo Reservacion | Cliente, Estudio, Fotografo, Paquete | Sesion, Pedido | Validación de disponibilidad crítica |
| Módulo Sesion | Reservacion confirmada | Pedido, Entregables | Transición de estado estricta |
| Módulo Pedido | Sesion finalizada | Pago, ProductoFinal | Cálculo automático y trazabilidad |
| Módulo Pago | Pedido activo | Notificaciones, Finanzas | Conciliación y estados multi-método |
| Módulo Entregables | Pedido y Pago completados | Cliente, Notificaciones | Gestión de archivos y estados |
| Módulo Equipo | Estudio, Sesion | Mantenimiento, Inventario | Asignación temporal y revisiones |
| UI/UX Final | Todos los Providers estables | Pruebas, Despliegue | Coherencia visual y accesibilidad |
| CI/CD y Testing | Código estable, pruebas unitarias | Producción | Automatización y calidad |

**Reglas de dependencia estricta:**
1. Ninguna pantalla puede instanciar directamente un repositorio o modelo JSON.
2. Los providers nunca deben contener lógica de negocio; solo orquestación y estado.
3. Las validaciones de disponibilidad deben ejecutarse antes de persistir cualquier reservación.
4. El cambio de estado de `Reservacion` debe propagar actualizaciones a `Estudio.disponible`, `Fotografo` (calendario interno) y `Sesion` (cuando corresponda).
5. `Pedido` solo se genera tras `Sesion.estado = Finalizada` o `Post-procesamiento`.
6. `Pago` puede existir en estados parciales, pero `ProductoFinal.estado_entrega = Listo` requiere `Pedido.estado_pago = Pagado` o política explícita de anticipado.

---

Este plan cubre la arquitectura completa, el mapeo de entidades, la integración de Provider, la aplicación de la paleta de colores, las fases secuenciales con dependencias explícitas y los componentes transversales necesarios para un sistema de gestión fotográfica profesional, escalable y mantenible. No se incluye código fuente, manteniendo el enfoque exclusivo en la planificación, estructura y flujo de implementación.


Aquí tienes exclusivamente la sección de dependencias listas para integrar en tu `pubspec.yaml`, alineadas estrictamente con la arquitectura y fases del plan anterior:

```yaml
dependencies:
  flutter:
    sdk: flutter
  # Gestión de estado y orquestación
  provider: ^6.1.2
  # Navegación declarativa y gestión de rutas profundas
  go_router: ^14.2.0
  # Cliente HTTP con interceptores, reintentos y manejo de errores
  dio: ^5.4.3+1
  # Almacenamiento local ligero (config, tokens, preferencias de UI)
  shared_preferences: ^2.2.3
  # Variables de entorno por perfil (dev, staging, prod)
  flutter_dotenv: ^5.1.0
  # Validación de formularios y reglas de dominio
  formz: ^0.7.0
  # Mapeo seguro de JSON ↔ Entidades
  json_annotation: ^4.9.0
  # Internacionalización, formateo de moneda/fechas y zonas horarias
  intl: ^0.19.0
  # Registro de eventos, auditoría y trazabilidad de errores
  logger: ^2.3.0
  # Calendario para agenda y validación de solapamientos
  table_calendar: ^3.1.1
  # Renderizado optimizado de iconos y recursos vectoriales
  flutter_svg: ^2.0.10+1
  # Cache de imágenes para entregables y previews
  cached_network_image: ^3.3.1

dev_dependencies:
  flutter_test:
    sdk: flutter
  # Linter oficial para mantener arquitectura limpia y estándares
  flutter_lints: ^4.0.0
  # Generación de código para serialización JSON
  build_runner: ^2.4.9
  json_serializable: ^6.8.0
  # Mocking para pruebas unitarias de casos de uso y repositorios
  mockito: ^5.4.4
```

### Mapeo directo con el plan de implementación
| Capa del Plan | Dependencias Asignadas | Propósito |
|---------------|------------------------|-----------|
| `core/network` | `dio`, `flutter_dotenv`, `logger` | Interceptores, manejo de errores, registro de auditoría, configuración por entorno |
| `core/utils` | `intl`, `formz`, `shared_preferences` | Validación de reglas, formateo de moneda/fecha, persistencia de configuración local |
| `data/models` | `json_annotation`, `json_serializable`, `build_runner` | Generación segura de mapeo JSON ↔ Entidades inmutables |
| `domain` | `formz`, `logger` | Contratos de validación, trazabilidad de decisiones de negocio |
| `presentation/providers` | `provider`, `go_router` | Inyección jerárquica, gestión de estados `loading/success/error`, navegación declarativa |
| `presentation/screens/widgets` | `table_calendar`, `cached_network_image`, `flutter_svg` | Agenda visual, carga de entregables, iconografía coherente con la paleta |
| `test/` | `flutter_test`, `mockito` | Cobertura unitaria de casos de uso y repositorios, aislamiento de dependencias externas |

**Notas de integración:**
1. Ejecuta `flutter pub get` y luego `dart run build_runner build` tras crear los modelos con `@JsonSerializable()`.
2. Configura `flutter_dotenv` en `main` antes de instanciar `MultiProvider`.
3. Las versiones indicadas son estables y compatibles con Flutter 3.22+ (LTS vigente a 2026). Si tu canal de compilación requiere restricciones de versión mínima, ajusta los rangos manteniendo la compatibilidad semántica.


Crea un plan de implementacion detallado , que tenga dependencias y los colores son estos #D4CFC3 #384057 #B3B3B3 #E8E8E6 #2E3B6E , y la estructura de carpetas y utiliza provider y no integres codigo solo integra el plan y las entidades basandote en el siguiente codigo y agrega lo que creas que sea necesario para el plan -
