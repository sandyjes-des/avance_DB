
# Script de avance del proyecto (aun no esta correjido)
```sql
CREATE DATABASE gestion_reclamos;

CREATE TABLE Empresa (
    id_empresa SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    ruc VARCHAR(20) NOT NULL UNIQUE,
    direccion VARCHAR(150),
    telefono VARCHAR(20),
    email VARCHAR(100) NOT NULL UNIQUE,
    estado VARCHAR(20) DEFAULT 'activo',
    CONSTRAINT CHK_Empresa_estado CHECK (estado IN ('activo', 'inactivo'))
);
ALTER TABLE Empresa RENAME COLUMN ruc TO nit;

CREATE TABLE Persona (
    id_persona SERIAL PRIMARY KEY,
    nombres VARCHAR(100) NOT NULL,
    apellidos VARCHAR(100) NOT NULL,
    carnet VARCHAR(15) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    telefono VARCHAR(20),
    edad INT NOT NULL,
    CONSTRAINT CHK_Persona_edad CHECK (edad >= 18)
);

CREATE TABLE Cliente (
    id_persona INT PRIMARY KEY,
    id_empresa INT NOT NULL,
    fecha_registro TIMESTAMP DEFAULT NOW(),
    CONSTRAINT FK_Cliente_Persona FOREIGN KEY (id_persona) REFERENCES Persona(id_persona),
    CONSTRAINT FK_Cliente_Empresa FOREIGN KEY (id_empresa) REFERENCES Empresa(id_empresa)
);

CREATE TABLE Empleado (
    id_persona INT PRIMARY KEY,
    id_empresa INT NOT NULL,
    cargo VARCHAR(50),
    estado VARCHAR(20) DEFAULT 'activo',
    CONSTRAINT FK_Empleado_Persona FOREIGN KEY (id_persona) REFERENCES Persona(id_persona),
    CONSTRAINT FK_Empleado_Empresa FOREIGN KEY (id_empresa) REFERENCES Empresa(id_empresa),
    CONSTRAINT CHK_Empleado_estado CHECK (estado IN ('activo', 'inactivo'))
);

CREATE TABLE CategoriaReclamo (
    id_categoria SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL UNIQUE,
    descripcion TEXT
);

CREATE TABLE EstadoReclamo (
    id_estado SERIAL PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE Reclamo (
    id_reclamo SERIAL PRIMARY KEY,
    id_cliente INT NOT NULL,
    id_categoria INT NOT NULL,
    id_estado INT NOT NULL,
    id_empleado INT,
    descripcion TEXT NOT NULL,
    fecha_reclamo TIMESTAMP DEFAULT NOW(),
    prioridad VARCHAR(20) DEFAULT 'media',
    CONSTRAINT CHK_Reclamo_prioridad CHECK (prioridad IN ('baja', 'media', 'alta')),
    CONSTRAINT FK_Reclamo_Cliente FOREIGN KEY (id_cliente) REFERENCES Cliente(id_persona),
    CONSTRAINT FK_Reclamo_CategoriaReclamo FOREIGN KEY (id_categoria) REFERENCES CategoriaReclamo(id_categoria),
    CONSTRAINT FK_Reclamo_EstadoReclamo FOREIGN KEY (id_estado) REFERENCES EstadoReclamo(id_estado),
    CONSTRAINT FK_Reclamo_Empleado FOREIGN KEY (id_empleado) REFERENCES Empleado(id_persona)
);

CREATE TABLE Seguimiento (
    id_reclamo INT,
    numero_seguimiento INT,
    comentario TEXT NOT NULL,
    fecha TIMESTAMP DEFAULT NOW(),
    CONSTRAINT PK_Seguimiento PRIMARY KEY (id_reclamo, numero_seguimiento),
    CONSTRAINT FK_Seguimiento_Reclamo FOREIGN KEY (id_reclamo) REFERENCES Reclamo(id_reclamo)
);

CREATE TABLE EncuestaSatisfaccion (
    id_encuesta SERIAL PRIMARY KEY,
    id_reclamo INT NOT NULL,
    calificacion INT NOT NULL,
    comentario TEXT,
    CONSTRAINT CHK_Encuesta_calificacion CHECK (calificacion BETWEEN 1 AND 5),
    CONSTRAINT FK_EncuestaSatisfaccion_Reclamo FOREIGN KEY (id_reclamo) REFERENCES Reclamo(id_reclamo)
);

CREATE TABLE Historico_Riesgo (
    id_cliente INT PRIMARY KEY,
    nombre_cliente VARCHAR(200),
    promedio_calificacion NUMERIC(4,2),
    fecha_registro TIMESTAMP DEFAULT NOW()
);

CREATE TABLE auditoria_general (
    id_auditoria SERIAL PRIMARY KEY,
    tabla_afectada VARCHAR(100),
    accion VARCHAR(10),
    usuario VARCHAR(100),
    fecha TIMESTAMP DEFAULT NOW(),
    datos_anteriores JSONB,
    datos_nuevos JSONB
);

-- POBLAR TABLAS DEL SISTEMA DE RECLAMOS
-- Insertar datos en orden para mantener integridad referencial

TRUNCATE TABLE 
    persona, 
    cliente, 
    empleado, 
    empresa,
    reclamo,
    categoriareclamo,
   estadoreclamo,
  EncuestaSatisfaccion,
  Seguimiento
RESTART IDENTITY CASCADE;
-- 1. EMPRESA (10 registros)
INSERT INTO Empresa (nombre, nit, direccion, telefono, email, estado) VALUES
('Tigo Bolivia', '20123456789', 'Av. San Martín 1234, Santa Cruz de la Sierra', '3-3200000', 'contacto@tigo.com.bo', 'activo'),
('PIL Andina S.A.', '20456789012', 'Av. Cristo Redentor Km 5, Santa Cruz de la Sierra', '3-3456789', 'info@pilandina.com.bo', 'activo'),
('Sofía Ltda.', '20789012345', 'Av. Banzer Km 8, Santa Cruz de la Sierra', '3-3460000', 'ventas@sofia.com.bo', 'activo'),
('Toyosa S.A.', '20111222333', 'Av. Cristo Redentor 2500, Santa Cruz de la Sierra', '3-3422222', 'contacto@toyosa.com.bo', 'activo'),
('Farmacorp S.A.', '20444555666', 'Av. San Martín 455, Santa Cruz de la Sierra', '3-3433333', 'info@farmacorp.com', 'activo'),
('Banco Mercantil Santa Cruz', '20777888999', 'Av. San Martín 789, Santa Cruz de la Sierra', '3-3500000', 'contacto@bm.com.bo', 'activo'),
('Hipermaxi S.A.', '20101020304', 'Av. Beni 1500, Santa Cruz de la Sierra', '3-3477777', 'clientes@hipermaxi.com', 'activo'),
('Entel S.A.', '20303040506', 'Av. Alemania 1200, Santa Cruz de la Sierra', '3-3600000', 'servicio@entel.bo', 'activo'),
('Cotas R.L.', '20505060708', 'Av. Cristóbal de Mendoza 600, Santa Cruz de la Sierra', '3-3520000', 'info@cotas.com.bo', 'activo'),
('Fexpocruz', '20707080910', 'Av. Roca y Coronado 1230, Santa Cruz de la Sierra', '3-3380000', 'contacto@fexpocruz.com.bo', 'activo');

-- 2. PERSONA (15 registros)
INSERT INTO Persona (nombres, apellidos, carnet, email, telefono, edad) VALUES
('Juan Carlos', 'Pérez Gómez', '12345678', 'juan.perez@gmail.com', '999888777', 35),
('María Elena', 'López Fernández', '87654321', 'maria.lopez@yahoo.com', '988777666', 28),
('Carlos Alberto', 'Ramírez Torres', '11223344', 'carlos.ramirez@hotmail.com', '977666555', 42),
('Ana Lucía', 'García Mendoza', '44332211', 'ana.garcia@outlook.com', '966555444', 31),
('Luis Fernando', 'Torres Paredes', '55667788', 'luis.torres@gmail.com', '955444333', 25),
('Patricia Beatriz', 'Sánchez Castro', '99887766', 'patricia.sanchez@empresa.com', '944333222', 45),
('Jorge Eduardo', 'Díaz Rojas', '33445566', 'jorge.diaz@empresa.com', '933222111', 38),
('Rosa María', 'Herrera Núñez', '22334455', 'rosa.herrera@gmail.com', '922111000', 29),
('Miguel Ángel', 'Vargas Cruz', '66778899', 'miguel.vargas@yahoo.com', '911000999', 52),
('Carmen Rosa', 'Flores Ortiz', '44556677', 'carmen.flores@hotmail.com', '900999888', 33),
('José Luis', 'Morales Vega', '99881122', 'jose.morales@outlook.com', '999888777', 41),
('Laura Isabel', 'Castillo Ríos', '88772233', 'laura.castillo@gmail.com', '988777666', 27),
('Ricardo Andrés', 'Ramos Silva', '77663344', 'ricardo.ramos@empresa.com', '977666555', 36),
('Verónica Paola', 'Reyes Campos', '66554433', 'veronica.reyes@yahoo.com', '966555444', 24),
('Fernando Javier', 'Molina Vargas', '55443322', 'fernando.molina@gmail.com', '955444333', 47),
('Andrea Patricia', 'Mendoza Ríos', '12345987', 'andrea.mendoza@gmail.com', '988776655', 29),
('Diego Armando', 'Cáceres Torres', '98765432', 'diego.caceres@yahoo.com', '977665544', 34),
('Valeria Sofía', 'Paredes Luna', '11223355', 'valeria.paredes@hotmail.com', '966554433', 26),
('Sergio Alejandro', 'Núñez Campos', '44332266', 'sergio.nunez@outlook.com', '955443322', 41),
('Gabriela Fernanda', 'Quispe Mamani', '55667799', 'gabriela.quispe@gmail.com', '944332211', 32);

-- 3. CLIENTE (10 registros) – asociados a empresas
INSERT INTO Cliente (id_persona, id_empresa, fecha_registro) VALUES
(1, 1, '2025-01-15 10:30:00'),
(2, 2, '2025-01-20 14:45:00'),
(3, 1, '2025-02-01 09:00:00'),
(4, 3, '2025-02-10 11:20:00'),
(5, 2, '2025-02-15 16:10:00'),
(6, 1, '2025-03-01 08:30:00'),
(7, 2, '2025-03-05 13:15:00'),
(8, 1, '2025-03-10 10:00:00'),
(9, 3, '2025-03-12 12:00:00'),
(10, 2, '2025-03-15 15:30:00');

-- 4. EMPLEADO (5 registros) – algunos asignados a reclamos
INSERT INTO Empleado (id_persona, id_empresa, cargo, estado) VALUES
(11, 1, 'Analista de soporte', 'activo'),
(12, 2, 'Especialista en facturación', 'activo'),
(13, 1, 'Técnico de campo', 'activo'),
(14, 3, 'Atención al cliente', 'inactivo'),
(15, 2, 'Supervisor', 'activo'),
(16, 4, 'Asesor comercial', 'activo'),
(17, 5, 'Técnico de soporte', 'activo'),
(18, 6, 'Analista de calidad', 'activo'),
(19, 7, 'Supervisor de ventas', 'activo'),
(20, 8, 'Atención al cliente', 'inactivo');

-- 5. CATEGORIA RECLAMO (4 registros)
INSERT INTO CategoriaReclamo ( nombre, descripcion) VALUES
('Problema técnico', 'Fallas en equipos, software o servicios técnicos'),
('Facturación', 'Errores en facturas, cobros indebidos, etc.'),
('Atención al cliente', 'Quejas sobre el servicio o trato recibido'),
('Garantía', 'Solicitudes relacionadas con garantías de productos'),
('Devoluciones', 'Solicitudes de devolución de dinero o productos'),
('Instalación', 'Problemas con la instalación de productos o servicios'),
('Mantenimiento', 'Solicitudes de mantenimiento preventivo o correctivo'),
('Soporte postventa', 'Asistencia después de la compra'),
('Quejas por producto', 'Inconformidades con la calidad del producto'),
('Sugerencias', 'Comentarios y sugerencias para mejorar el servicio');

-- 6. ESTADO RECLAMO (3 registros)
INSERT INTO EstadoReclamo (nombre) VALUES
('Pendiente'),
('En proceso'),
('Resuelto'),
('En revisión'),
('Derivado'),
('Esperando respuesta'),
('Cancelado'),
('Rechazado'),
('En espera de repuestos'),
('Finalizado');

-- 7. RECLAMO (10 registros) – algunos asignados a empleados
INSERT INTO Reclamo (id_cliente, id_categoria, id_estado, id_empleado, descripcion, fecha_reclamo, prioridad) VALUES
(1, 1, 1, NULL, 'El equipo no enciende después de la actualización', '2025-02-16 09:00:00', 'alta'),
(2, 2, 2, 12, 'Me llegó factura con monto incorrecto', '2026-02-16 10:30:00', 'media'),
(3, 3, 3, 11, 'Atención inadecuada en la tienda', '2026-02-16 11:45:00', 'baja'),
(4, 1, 1, 13, 'La computadora se apaga sola', '2026-02-17 08:15:00', 'alta'),
(5, 4, 2, 15, 'Solicito cambio de producto defectuoso', '2026-02-17 09:30:00', 'media'),
(6, 2, 1, NULL, 'Cobro duplicado en la tarjeta', '2026-02-17 10:45:00', 'alta'),
(7, 1, 2, 11, 'Falla en el software de facturación', '2025-02-18 08:00:00', 'media'),
(8, 3, 3, NULL, 'Mala atención telefónica', '2026-02-18 09:20:00', 'baja'),
(9, 4, 1, 13, 'Producto no cumple especificaciones', '2026-02-18 11:10:00', 'media'),
(10, 2, 2, 12, 'Factura sin número de RUC', '2026-02-19 08:30:00', 'alta');

-- 8. SEGUIMIENTO (10 registros) – varios reclamos tienen seguimiento
INSERT INTO Seguimiento (id_reclamo, numero_seguimiento, comentario, fecha) VALUES
(1, 1, 'Se revisó el caso, se asignó técnico', '2026-03-10 10:00:00'),
(1, 2, 'Técnico visitó domicilio, diagnosticó falla', '2026-03-1 14:30:00'),
(2, 1, 'Se contactó al cliente para verificar datos', '2026-03-1 11:00:00'),
(2, 2, 'Se corrigió factura, se enviará nueva', '2026-02-25 09:15:00'),
(3, 1, 'Se disculpó con el cliente, se ofreció compensación', '2026-03-26 14:00:00'),
(4, 1, 'Se programó revisión técnica', '2026-03-20 09:00:00'),
(5, 1, 'Se verificó garantía, procede cambio', '2026-03-3 10:30:00'),
(6, 1, 'Se investiga duplicidad con el banco', '2026-03-1 11:30:00'),
(7, 1, 'Se actualizó software, pendiente de prueba', '2026-02-28 09:00:00'),
(8, 1, 'Se registró queja en el área correspondiente', '2026-02-26 10:00:00'),
(9, 1, 'Se solicitó documentación al cliente', '2026-03-05 12:00:00'),
(10, 1, 'Se generó nueva factura', '2026-03-19 09:00:00');

-- 9. ENCUESTA SATISFACCION (10 registros) – una por cada reclamo
INSERT INTO EncuestaSatisfaccion (id_reclamo, calificacion, comentario) VALUES
(1, 4, 'Buena atención, aunque tardó un poco'),
(2, 5, 'Rápida solución, excelente'),
(3, 2, 'No quedé conforme con la respuesta'),
(4, 3, 'Regular, esperaba más rapidez'),
(5, 5, 'Muy satisfecho con el cambio'),
(6, 4, 'Se resolvió el problema, gracias'),
(7, 3, 'Aún no se soluciona del todo'),
(8, 1, 'Pésima atención, no volveré'),
(9, 4, 'Bien, aunque demoraron'),
(10, 5, 'Perfecto, factura corregida en el día');

--creacion de indices 

--Índice para buscar reclamos por cliente
CREATE INDEX idx_reclamo_cliente ON Reclamo(id_cliente);
SELECT * FROM Reclamo WHERE id_cliente = 3;--permite encontrar los reclamos de un cliente

--Índice para estado del reclamo
CREATE INDEX idx_reclamo_estado ON Reclamo(id_estado);
SELECT * FROM Reclamo WHERE id_estado = 1;--Las empresas suelen consultar: reclamos abiertos

--Índice para categoría
CREATE INDEX idx_reclamo_categoria ON Reclamo(id_categoria);
SELECT * FROM Reclamo WHERE id_categoria = 2; --Por ejemplo: reclamos de facturación

--Índice para encuestas
CREATE INDEX idx_encuesta_reclamo ON EncuestaSatisfaccion(id_reclamo);

--

--consultas
--cliente, reclamo, empleado, empresa, categoria, estado reclamo
SELECT p.nombres as nombre_cliente, r.descripcion as reclamo, pe.nombres as nombre_empleado,
e.nombre as nombre_empresa, cr.nombre as categoria_reclamo, er. nombre as estado_reclamo
from cliente c 
INNER JOIN reclamo r  on c.id_persona = r.id_cliente 
INNER JOIN persona p ON c.id_persona = p.id_persona
INNER JOIN empresa e ON c.id_empresa = e.id_empresa
INNER JOIN categoriaReclamo cr ON r.id_categoria = cr.id_categoria
INNER JOIN estadoReclamo er ON r.id_estado = er.id_estado
INNER JOIN empleado em ON r.id_empleado = em.id_persona
INNER JOIN persona pe ON em.id_persona = pe.id_persona;

--empledos con su reclamo hasta lo que no tienen
SELECT  p.nombres as empleado, r.descripcion as reclamo, e.cargo
FROM empleado e LEFT JOIN reclamo r ON r.id_empleado = e.id_persona
INNER JOIN persona p ON p.id_persona = e.id_persona;

--cliente, estado reclamo, calificacion promedio, nota maxima, nota minima
SELECT 
    p.nombres AS cliente,
    er.nombre AS estado_reclamo,
    ROUND(AVG(es.calificacion),2) AS promedio_calificacion,
    MAX(es.calificacion) AS nota_maxima,
    MIN(es.calificacion) AS nota_minima
FROM EncuestaSatisfaccion es
INNER JOIN Reclamo r ON es.id_reclamo = r.id_reclamo
INNER JOIN Cliente c ON r.id_cliente = c.id_persona
INNER JOIN Persona p ON c.id_persona = p.id_persona
INNER JOIN EstadoReclamo er ON r.id_estado = er.id_estado
GROUP BY cliente, er.nombre
ORDER BY cliente;

--nombre y promedio del cliene
SELECT p.nombres AS cliente,
  ROUND(AVG(es.calificacion),2) AS promedio_cliente
FROM EncuestaSatisfaccion es
INNER JOIN Reclamo r ON es.id_reclamo = r.id_reclamo
INNER JOIN Cliente c ON r.id_cliente = c.id_persona
INNER JOIN Persona p ON c.id_persona = p.id_persona
GROUP BY cliente
HAVING AVG(es.calificacion) < (
        SELECT AVG(calificacion) 
        FROM EncuestaSatisfaccion
);

--si existe al menos una categoria cuyo calificacion sea menor a 2
SELECT cr.nombre AS categoria
FROM CategoriaReclamo cr
WHERE EXISTS (
    SELECT 1
    FROM Reclamo r
    INNER JOIN EncuestaSatisfaccion es 
        ON r.id_reclamo = es.id_reclamo
    WHERE r.id_categoria = cr.id_categoria
      AND es.calificacion <= 2
);

--nombre de los cliente cuyo promedio este por debajo del promedio general
SELECT p.nombres AS cliente,
    ROUND(AVG(es.calificacion),2) AS promedio_cliente,
    ( SELECT ROUND(AVG(calificacion),2)
        FROM EncuestaSatisfaccion) AS promedio_general
FROM EncuestaSatisfaccion es
INNER JOIN Reclamo r ON es.id_reclamo = r.id_reclamo
INNER JOIN Cliente c ON r.id_cliente = c.id_persona
INNER JOIN Persona p ON c.id_persona = p.id_persona
GROUP BY cliente
HAVING AVG(es.calificacion) < (
        SELECT AVG(calificacion)
        FROM EncuestaSatisfaccion
);

--5 Listar categorías de reclamo que tienen al menos un reclamo con baja calificación

SELECT cr.nombre AS categoria
FROM CategoriaReclamo cr
WHERE EXISTS (
    SELECT 1
    FROM Reclamo r
    INNER JOIN EncuestaSatisfaccion es 
        ON r.id_reclamo = es.id_reclamo
    WHERE r.id_categoria = cr.id_categoria
      AND es.calificacion <= 2
);

--union
SELECT p.nombres AS empleado, 'activo' AS estado
FROM Empleado em
INNER JOIN Persona p ON em.id_persona = p.id_persona
WHERE em.estado = 'activo'

UNION

SELECT p.nombres AS empleado, 'inactivo' AS estado
FROM Empleado em
INNER JOIN Persona p ON em.id_persona = p.id_persona
WHERE em.estado = 'inactivo';

--intersec
SELECT r.id_cliente FROM Reclamo r
WHERE r.id_categoria = 1

INTERSECT

SELECT r.id_cliente FROM Reclamo r
WHERE r.id_categoria = 2;

--funciones y triggers

CREATE OR REPLACE FUNCTION verificar_riesgo_cliente()
RETURNS TRIGGER AS $$
DECLARE
    promedio_cliente NUMERIC;
BEGIN

    SELECT AVG(es.calificacion)
    INTO promedio_cliente
    FROM EncuestaSatisfaccion es
    JOIN Reclamo r ON es.id_reclamo = r.id_reclamo
    WHERE r.id_cliente = NEW.id_reclamo;

    IF promedio_cliente < 3 THEN
        INSERT INTO Historico_Riesgo (id_cliente)
        VALUES (NEW.id_reclamo)
        ON CONFLICT DO NOTHING;
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_verificar_riesgo
AFTER INSERT ON EncuestaSatisfaccion
FOR EACH ROW
EXECUTE FUNCTION verificar_riesgo_cliente();

---auditotia
CREATE OR REPLACE FUNCTION fn_auditoria_general()
RETURNS TRIGGER AS $$
BEGIN

    IF TG_OP = 'DELETE' THEN
        INSERT INTO auditoria_general(
            tabla_afectada,
            accion,
            usuario,
            datos_anteriores
        )
        VALUES (
            TG_TABLE_NAME,
            TG_OP,
            current_user,
            to_jsonb(OLD)
        );
        RETURN OLD;

    ELSE
        INSERT INTO auditoria_general(
            tabla_afectada,
            accion,
            usuario,
            datos_anteriores,
            datos_nuevos
        )
        VALUES (
            TG_TABLE_NAME,
            TG_OP,
            current_user,
            to_jsonb(OLD),
            to_jsonb(NEW)
        );
        RETURN NEW;
    END IF;

END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_auditoria_reclamo
AFTER INSERT OR UPDATE OR DELETE
ON Reclamo
FOR EACH ROW
EXECUTE FUNCTION fun_auditoria_general();


CREATE TRIGGER trg_auditoria_persona
AFTER INSERT OR UPDATE OR DELETE
ON Persona
FOR EACH ROW
EXECUTE FUNCTION fun_auditoria_general();


CREATE TRIGGER trg_auditoria_empresa
AFTER INSERT OR UPDATE OR DELETE
ON Empresa
FOR EACH ROW
EXECUTE FUNCTION fun_auditoria_general();


CREATE TRIGGER trg_auditoria_empleado
AFTER INSERT OR UPDATE OR DELETE
ON Empleado
FOR EACH ROW
EXECUTE FUNCTION fun_auditoria_general();

CREATE TRIGGER trg_auditoria_seguimiento
AFTER INSERT OR UPDATE OR DELETE
ON Seguimiento
FOR EACH ROW
EXECUTE FUNCTION fun_auditoria_general();

--vista normal guarda una consulta

CREATE VIEW v_reporte_reclamos AS
SELECT
    r.id_reclamo,
    p.nombres || ' ' || p.apellidos AS cliente,
    cr.nombre AS categoria,
    er.nombre AS estado,
    r.descripcion,
    r.fecha_reclamo
FROM Reclamo r
JOIN Cliente c ON r.id_cliente = c.id_persona
JOIN Persona p ON c.id_persona = p.id_persona
JOIN CategoriaReclamo cr ON r.id_categoria = cr.id_categoria
JOIN EstadoReclamo er ON r.id_estado = er.id_estado;

SELECT * FROM v_reporte_reclamos;

--2da vista 
CREATE VIEW v_promedio_satisfaccion AS
SELECT
    c.id_persona AS cliente,
    p.nombres || ' ' || p.apellidos AS nombre_cliente,
    ROUND(AVG(es.calificacion),2) AS promedio
FROM EncuestaSatisfaccion es
JOIN Reclamo r ON es.id_reclamo = r.id_reclamo
JOIN Cliente c ON r.id_cliente = c.id_persona
JOIN Persona p ON p.id_persona = c.id_persona
GROUP BY c.id_persona, p.nombres, p.apellidos;

SELECT * FROM v_promedio_satisfaccion;

--Vista materializada

CREATE MATERIALIZED VIEW mv_estadisticas_reclamos AS
SELECT
    cr.nombre AS categoria,
    COUNT(r.id_reclamo) AS total_reclamos
FROM Reclamo r
JOIN CategoriaReclamo cr
ON r.id_categoria = cr.id_categoria
GROUP BY cr.nombre;

--refrescar la tabla
REFRESH MATERIALIZED VIEW mv_estadisticas_reclamos;

SELECT * FROM mv_estadisticas_reclamos;

--2da vista materializada
CREATE MATERIALIZED VIEW vm_promedio_satisfaccion_clientes AS
SELECT
    c.id_persona AS id_cliente,
    p.nombres || ' ' || p.apellidos AS nombre_cliente,
    COUNT(es.calificacion) AS total_encuestas,
    ROUND(AVG(es.calificacion),2) AS promedio_satisfaccion
FROM EncuestaSatisfaccion es
JOIN Reclamo r ON es.id_reclamo = r.id_reclamo
JOIN Cliente c ON r.id_cliente = c.id_persona
JOIN Persona p ON c.id_persona = p.id_persona
GROUP BY c.id_persona, p.nombres, p.apellidos;

--refrescar la tabla
REFRESH MATERIALIZED VIEW vm_promedio_satisfaccion_clientes;

SELECT * FROM vm_promedio_satisfaccion_clientes;
```
