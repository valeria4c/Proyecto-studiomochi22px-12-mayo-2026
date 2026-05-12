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
<img width="875" height="672" alt="image" src="https://github.com/user-attachments/assets/051d2ac0-5a92-437f-85f8-86778352adc2" />
<img width="797" height="736" alt="image" src="https://github.com/user-attachments/assets/c3363344-1779-4ed3-a41c-5ea19418561e" />
<img width="769" height="365" alt="image" src="https://github.com/user-attachments/assets/24d8b5ea-ac4c-4b4f-b226-8c507eb99c59" />


puedes generar un script  en sql para descsrgr  con el nombre de dbstudiomochi22px.sql para las 10 entidades con sus relaciones

-- Creación de la base de datos
CREATE DATABASE IF NOT EXISTS dbstudiomochi22px;
USE dbstudiomochi22px;

-- 1. Tabla: cliente (Núcleo)
CREATE TABLE cliente (
    id_cliente INT AUTO_INCREMENT,
    nombre VARCHAR(80) NOT NULL,
    apellido VARCHAR(80) NOT NULL,
    email VARCHAR(120) UNIQUE,
    telefono VARCHAR(20),
    fecha_nacimiento DATE,
    fecha_registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    activo BOOLEAN DEFAULT TRUE,
    CONSTRAINT pk_cliente PRIMARY KEY (id_cliente)
);

-- 2. Tabla: fotografo (Personal)
CREATE TABLE fotografo (
    id_fotografo INT AUTO_INCREMENT,
    nombre VARCHAR(80) NOT NULL,
    apellido VARCHAR(80) NOT NULL,
    email VARCHAR(120) UNIQUE,
    telefono VARCHAR(20),
    especialidad VARCHAR(60),
    fecha_contrato DATE NOT NULL,
    activo BOOLEAN DEFAULT TRUE,
    CONSTRAINT pk_fotografo PRIMARY KEY (id_fotografo)
);

-- 3. Tabla: estudio (Instalación)
CREATE TABLE estudio (
    id_estudio INT AUTO_INCREMENT,
    nombre VARCHAR(60) NOT NULL,
    descripcion TEXT,
    capacidad_personas TINYINT NOT NULL,
    color_fondo VARCHAR(30),
    area_m2 DECIMAL(5,2),
    disponible BOOLEAN DEFAULT TRUE,
    CONSTRAINT pk_estudio PRIMARY KEY (id_estudio)
);

-- 4. Tabla: paquete (Catálogo)
CREATE TABLE paquete (
    id_paquete INT AUTO_INCREMENT,
    nombre VARCHAR(80) NOT NULL,
    descripcion TEXT,
    num_fotos_incluidas SMALLINT NOT NULL,
    duracion_minutos SMALLINT NOT NULL,
    precio DECIMAL(8,2) NOT NULL,
    activo BOOLEAN DEFAULT TRUE,
    CONSTRAINT pk_paquete PRIMARY KEY (id_paquete)
);

-- 5. Tabla: reservacion (Agenda)
CREATE TABLE reservacion (
    id_reservacion INT AUTO_INCREMENT,
    id_cliente INT,
    id_paquete INT,
    id_estudio INT,
    id_fotografo INT,
    fecha_hora DATETIME NOT NULL,
    estado ENUM('Pendiente', 'Confirmada', 'Cancelada', 'Completada') NOT NULL,
    canal_origen VARCHAR(30),
    notas TEXT,
    creada_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT pk_reservacion PRIMARY KEY (id_reservacion),
    CONSTRAINT fk_res_cliente FOREIGN KEY (id_cliente) REFERENCES cliente(id_cliente),
    CONSTRAINT fk_res_paquete FOREIGN KEY (id_paquete) REFERENCES paquete(id_paquete),
    CONSTRAINT fk_res_estudio FOREIGN KEY (id_estudio) REFERENCES estudio(id_estudio),
    CONSTRAINT fk_res_fotografo FOREIGN KEY (id_fotografo) REFERENCES fotografo(id_fotografo)
);

-- 6. Tabla: sesion (Operación)
CREATE TABLE sesion (
    id_sesion INT AUTO_INCREMENT,
    id_reservacion INT UNIQUE,
    fecha_hora_inicio DATETIME NOT NULL,
    fecha_hora_fin DATETIME,
    estado ENUM('En curso', 'Finalizada', 'Post-procesamiento') NOT NULL,
    num_fotos_tomadas SMALLINT,
    observaciones TEXT,
    CONSTRAINT pk_sesion PRIMARY KEY (id_sesion),
    CONSTRAINT fk_ses_reservacion FOREIGN KEY (id_reservacion) REFERENCES reservacion(id_reservacion)
);

-- 7. Tabla: equipo (Activo)
CREATE TABLE equipo (
    id_equipo INT AUTO_INCREMENT,
    id_estudio INT,
    nombre VARCHAR(80) NOT NULL,
    tipo ENUM('Cámara', 'Lente', 'Iluminación', 'Accesorio', 'Otro') NOT NULL,
    marca VARCHAR(50),
    modelo VARCHAR(60),
    num_serie VARCHAR(50) UNIQUE,
    estado ENUM('Operativo', 'Mantenimiento', 'Dañado', 'Baja') NOT NULL,
    fecha_adquisicion DATE,
    ultima_revision DATE,
    CONSTRAINT pk_equipo PRIMARY KEY (id_equipo),
    CONSTRAINT fk_equ_estudio FOREIGN KEY (id_estudio) REFERENCES estudio(id_estudio)
);

-- 8. Tabla: pedido (Venta)
CREATE TABLE pedido (
    id_pedido INT AUTO_INCREMENT,
    id_sesion INT UNIQUE,
    subtotal DECIMAL(10,2) NOT NULL,
    descuento DECIMAL(10,2) DEFAULT 0,
    total DECIMAL(10,2) NOT NULL,
    estado_pago ENUM('Pendiente', 'Parcial', 'Pagado', 'Cancelado') NOT NULL,
    fecha_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT pk_pedido PRIMARY KEY (id_pedido),
    CONSTRAINT fk_ped_sesion FOREIGN KEY (id_sesion) REFERENCES sesion(id_sesion)
);

-- 9. Tabla: producto_final (Entregable)
CREATE TABLE producto_final (
    id_producto INT AUTO_INCREMENT,
    id_pedido INT,
    tipo ENUM('Digital', 'Impresión', 'Álbum', 'Enmarcado') NOT NULL,
    formato VARCHAR(30),
    dimensiones VARCHAR(20),
    cantidad SMALLINT DEFAULT 1,
    precio_unitario DECIMAL(8,2) NOT NULL,
    url_archivo VARCHAR(255),
    estado_entrega ENUM('Procesando', 'Listo', 'Entregado') NOT NULL,
    CONSTRAINT pk_producto PRIMARY KEY (id_producto),
    CONSTRAINT fk_prod_pedido FOREIGN KEY (id_pedido) REFERENCES pedido(id_pedido)
);

-- 10. Tabla: pago (Finanzas)
CREATE TABLE pago (
    id_pago INT AUTO_INCREMENT,
    id_pedido INT,
    monto DECIMAL(10,2) NOT NULL,
    metodo_pago ENUM('Efectivo', 'Tarjeta', 'Transferencia', 'Otro') NOT NULL,
    referencia VARCHAR(80),
    fecha_pago TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    confirmado BOOLEAN DEFAULT FALSE,
    CONSTRAINT pk_pago PRIMARY KEY (id_pago),
    CONSTRAINT fk_pag_pedido FOREIGN KEY (id_pedido) REFERENCES pedido(id_pedido)
);
