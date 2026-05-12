
# Sistema de Gestión de Hospital Inteligente - Oracle SQL

Este repositorio contiene la solución técnica para un sistema de base de datos hospitalario desarrollado en **Oracle SQL**. El proyecto se centra en la aplicación de reglas de integridad referencial avanzadas, específicamente el uso de eliminaciones en cascada para garantizar la consistencia de la información clínica.

---

##  Descripción del Escenario
El sistema modela la gestión de un hospital inteligente donde los **Pacientes** asisten a **Consultas** realizadas por **Médicos**. Cada médico pertenece a una **Especialidad** y cada consulta puede generar una **Receta** médica. 

La lógica del negocio implementada exige que, si un paciente es eliminado del sistema, todos sus registros asociados (consultas y recetas) se borren automáticamente mediante la propiedad **CASCADE** para evitar datos huérfanos y mantener la integridad de la base de datos.

---

## Parte 1: Modelado y Estructura (DDL)

### Explicación del Modelo Lógico
El diseño se basa en una arquitectura de cinco entidades principales con las siguientes restricciones:
*   **ESPECIALIDAD:** Tabla maestra con nombres únicos (`UNIQUE`).
*   **MEDICO:** Vinculado a una especialidad mediante integridad referencial.
*   **PACIENTE:** Contiene datos personales con validación de cédula única y un `CHECK` para asegurar que la edad sea mayor o igual a cero.
*   **CONSULTA:** Entidad central con `ON DELETE CASCADE` vinculado al Paciente.
*   **RECETA:** Detalla el tratamiento, vinculada a la consulta también con `ON DELETE CASCADE`.

### Implementación de Tablas en Oracle SQL
```sql
-- Creación de tablas con integridad referencial avanzada
CREATE TABLE ESPECIALIDAD (
    ID_ESP NUMBER PRIMARY KEY,
    NOM_ESP VARCHAR2(50) NOT NULL UNIQUE
);

CREATE TABLE MEDICO (
    ID_MED NUMBER PRIMARY KEY,
    NOM_MED VARCHAR2(50) NOT NULL,
    APE_MED VARCHAR2(50) NOT NULL,
    ID_ESP NUMBER NOT NULL,
    CONSTRAINT FK_MED_ESP FOREIGN KEY (ID_ESP) REFERENCES ESPECIALIDAD(ID_ESP)
);

CREATE TABLE PACIENTE (
    ID_PAC NUMBER PRIMARY KEY,
    NOM_PAC VARCHAR2(50) NOT NULL,
    APE_PAC VARCHAR2(50) NOT NULL,
    CED_PAC VARCHAR2(10) NOT NULL UNIQUE,
    EDA_PAC NUMBER CHECK (EDA_PAC >= 0)
);

CREATE TABLE CONSULTA (
    ID_CON NUMBER PRIMARY KEY,
    FEC_CON DATE NOT NULL,
    ID_PAC NUMBER NOT NULL,
    ID_MED NUMBER NOT NULL,
    CONSTRAINT FK_CON_PAC FOREIGN KEY (ID_PAC) REFERENCES PACIENTE(ID_PAC) ON DELETE CASCADE,
    CONSTRAINT FK_CON_MED FOREIGN KEY (ID_MED) REFERENCES MEDICO(ID_MED)
);

CREATE TABLE RECETA (
    ID_REC NUMBER PRIMARY KEY,
    DES_REC VARCHAR2(255) NOT NULL,
    ID_CON NUMBER NOT NULL,
    CONSTRAINT FK_REC_CON FOREIGN KEY (ID_CON) REFERENCES CONSULTA(ID_CON) ON DELETE CASCADE
);

```

---

## Parte 2: DML e Integridad de Datos

### Inserción de Datos (INSERT)

Carga de datos de prueba utilizando la estructura estándar de valores:

```sql
-- Inserciones para Especialidad
INSERT INTO ESPECIALIDAD VALUES (1, 'Cardiología');
INSERT INTO ESPECIALIDAD VALUES (2, 'Pediatría');

-- Inserciones para Medico
INSERT INTO MEDICO VALUES (10, 'Santiago', 'Mora', 1);
INSERT INTO MEDICO VALUES (20, 'Ana', 'Martínez', 2);

-- Inserciones para Paciente
INSERT INTO PACIENTE VALUES (100, 'Luis', 'Andrade', '1809988776', 22);
INSERT INTO PACIENTE VALUES (200, 'María', 'Castillo', '1705544332', 45);

-- Inserciones para Consulta
INSERT INTO CONSULTA VALUES (500, TO_DATE('2026-05-10', 'YYYY-MM-DD'), 100, 10);
INSERT INTO CONSULTA VALUES (501, TO_DATE('2026-05-12', 'YYYY-MM-DD'), 200, 20);

-- Inserciones para Receta
INSERT INTO RECETA VALUES (1000, 'Aspirina 100mg diaria', 500);
INSERT INTO RECETA VALUES (1001, 'Jarabe mucolítico cada 8h', 501);

COMMIT;

```

### Pruebas de Integridad Realizadas

1. **Provocación de Error ORA-02291:** Intento fallido de insertar una consulta con un `ID_PAC` inexistente.
2. **Update Seguro:** Actualización de registros específicos validando la persistencia de datos.
3. **Análisis de Delete en Cascada:** Al eliminar un paciente con `ID_PAC = 100`, el motor de Oracle elimina automáticamente sus registros en `CONSULTA` y `RECETA`.

---

## Parte 3: Consultas SQL de Valor

Extracción de información estratégica para la toma de decisiones:

```sql
-- Uso de IN: Médicos en especialidades de interés
SELECT * FROM MEDICO WHERE ID_ESP IN (1, 2);

-- Uso de LIKE: Búsqueda parcial de pacientes por apellido
SELECT * FROM PACIENTE WHERE APE_PAC LIKE 'C%';

-- Uso de BETWEEN: Reporte de actividad en fechas específicas
SELECT * FROM CONSULTA WHERE FEC_CON BETWEEN TO_DATE('2026-05-01','YYYY-MM-DD') AND TO_DATE('2026-05-15','YYYY-MM-DD');

-- Agregación (MAX/AVG): Análisis demográfico de pacientes
SELECT MAX(EDA_PAC) AS EDAD_MAX, AVG(EDA_PAC) AS EDAD_PROM FROM PACIENTE;

-- JOIN: Reporte clínico integral (Unión de 5 tablas)
SELECT P.NOM_PAC, M.NOM_MED, E.NOM_ESP, R.DES_REC
FROM CONSULTA C
JOIN PACIENTE P ON C.ID_PAC = P.ID_PAC
JOIN MEDICO M ON C.ID_MED = M.ID_MED
JOIN ESPECIALIDAD E ON M.ID_ESP = E.ID_ESP
JOIN RECETA R ON C.ID_CON = R.ID_CON;

```

---

## Parte 4: Análisis Profesional

1. **Atomicidad:** Se garantiza que las transacciones sean indivisibles. En el registro de un paciente, todas las operaciones de inserción deben tener éxito; de lo contrario, el sistema realiza un `ROLLBACK` automático.
2. **DELETE sin WHERE:** Análisis sobre el riesgo de eliminar toda la data de una tabla y cómo Oracle gestiona los registros en el área de Undo para recuperaciones de emergencia.
3. **ON DELETE CASCADE:** Justificación técnica de por qué este modelo utiliza cascada para optimizar el rendimiento y la limpieza de datos en un entorno hospitalario de alta concurrencia.

---

**Desarrollado por:** Santiago Mora

**Carrera:** Ingeniería de Software

**Institución:** Universidad Técnica de Ambato (UTA)

```

```
