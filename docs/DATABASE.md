# Documentación de Base de Datos

## Motor de Base de Datos
- **Tipo:** SQL Server (Microsoft SQL Server)
- **Conexión:** Vía `pyodbc` / `pymssql` manejado mediante SQLAlchemy en `database.py`.

## Estructura de Tablas

### 1. Tabla: `cartera` (Lógico: Cliente / Factura)
**Propósito:** Es la tabla principal del sistema. Funciona con una estructura desnormalizada donde *cada fila representa una factura o documento de cobro*, pero contiene duplicada la información demográfica del cliente. Si un cliente tiene 3 facturas vencidas, existirán 3 filas con el mismo `nit_cliente`.

**Campos Relevantes:**
- `id` (Integer, Primary Key)
- `nit_cliente` (String): Identificador único del deudor (LIMPIO, solo números).
- `razon_social` (String): Nombre de la empresa o persona.
- `nro_docto_cruce` (String): Número de factura.
- `dias_vencidos` (Integer): Días de mora.
- `total_cop` (Numeric): Saldo pendiente (si es 0, no entra a cobro masivo).
- `valor_docto` (Numeric): Valor inicial.
- `correo` (String): Correos electrónicos (Si son múltiples, separados por `;`).
- **Campos Comerciales:**
  - `fecha_ingreso` (Date): Para certificados.
  - `condicion_pago` (String): Texto descriptivo (ej. "CONTADO 1 DIA").
  - `cupo_credito` (Numeric).
  - `fecha_ultima_compra` (Date).
  - `promedio_mensual` (Numeric).

**Nota de Diseño Reciente:**
Para permitir que un cliente sin deudas pueda obtener un Certificado Comercial, el sistema inserta una fila en esta tabla con `total_cop = 0` y `nro_docto_cruce = "N/A"`.

### 2. Tabla: `observaciones`
**Propósito:** Guarda el historial de gestiones de cobro (notas, llamadas, acuerdos) asociadas a un cliente.

**Campos Relevantes:**
- `id` (Integer, Primary Key).
- `cliente_id` (Integer, Foreign Key -> `cartera.id`): Vinculación al registro principal.
- `texto` (Text): El contenido de la gestión.
- `fecha_creacion` (DateTime): Auto-estampado con `func.now()`.

## Relaciones
- Relación **One-to-Many** entre `cartera` (Cliente) y `observaciones` (Historial).
- Mapeado en SQLAlchemy usando `relationship("Observacion", back_populates="cliente")`.

## Decisiones Técnicas y Procesos Automáticos
1. **Limpieza de NITs:** Pandas a veces lee los NITs como flotantes (ej. `12345.0`). Antes de buscar en la BD, la aplicación hace `re.sub(r'\D', '', str(nit).split('.')[0])` para extraer el NIT numérico real.
2. **Asignación Múltiple:** Al importar datos comerciales, la aplicación agrupa todas las filas de la base de datos bajo un mismo NIT en un diccionario de arreglos (`clientes_dict[nit] = [fila1, fila2]`). Los cambios comerciales se inyectan mediante un ciclo `for` a *todas* las filas simultáneamente para evitar inconsistencias.
