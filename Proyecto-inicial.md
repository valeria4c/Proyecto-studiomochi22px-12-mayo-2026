Actua como un administrador de base de datos ; proyecto un estudio fotografico llamado STUDIO MOCHI 22PX , que  entidades se necesitan para su gestion
<img width="931" height="634" alt="image" src="https://github.com/user-attachments/assets/8f6cdca7-33d0-4789-81da-d655a7aa0946" />
<img width="899" height="631" alt="image" src="https://github.com/user-attachments/assets/429ae823-6737-4fe8-b874-77e1888f5863" />
Como DBA, aquí está mi análisis de las entidades necesarias para gestionar **Studio Mochi 22px**:Aquí el desglose por dominio funcional:

**Clientes y reservaciones**
`CLIENTE` es el núcleo del negocio. Cada cliente puede hacer múltiples `RESERVACION`es, que a su vez están ligadas a una `SESION`. Separar reservación de sesión permite registrar intentos de agendar que quizás se cancelen o reprogramen sin perder el historial.

**Operaciones del estudio**
`SESION` es la entidad más relacionada: conecta al cliente, el fotógrafo, el paquete contratado y el espacio físico. `ESTUDIO` representa cada sala o set disponible (fondo blanco, fondo negro, set temático, etc.). `EQUIPO` se asocia al estudio para controlar qué cámara, flash o props están en cada sala y cuándo requieren mantenimiento.

**Catálogo y personal**
`PAQUETE` define las ofertas comerciales (mini sesión, sesión familiar, sesión de producto). `FOTOGRAFO` registra al equipo con sus especialidades para asignarlos correctamente.

**Ventas y entregables**
`PEDIDO` se genera al concluir la sesión y agrupa todo lo que el cliente va a recibir. `PRODUCTO_FINAL` representa cada entregable concreto: impresión, álbum digital, archivo en alta resolución, etc. `PAGO` maneja los cobros con soporte para pagos parciales o múltiples métodos.

**Relaciones clave a considerar**

- Un cliente puede tener varias sesiones a lo largo del tiempo (recurrencia).
- Un paquete puede usarse en muchas sesiones, pero cada sesión tiene un solo paquete.
- Un pedido puede tener varios pagos (anticipo + saldo).
- Un estudio puede tener varios equipos asignados.

¿Quieres que profundice en alguna entidad, agregue tablas de catálogo (tipos de producto, descuentos, promociones) o diseñe las queries más comunes para este negocio?
