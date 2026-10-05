# Módulos del Sistema

## 1. Módulo: Importación de Cartera (`/importar_excel`)
### Objetivo
Cargar el archivo matriz de deudores y facturas al sistema.
### Flujo del usuario
1. El usuario selecciona un archivo `.xlsx` desde la vista principal.
2. El sistema lo sube, limpia columnas y nombres (Ej: convierte "E-mail" a "correo").
3. Hace un proceso de inserción y actualización (UPSERT) en SQL Server comparando NIT y nro_docto_cruce.
### Reglas de Negocio
- Si un documento ya existe, actualiza saldos y días de mora.
- Si no existe, crea una nueva fila.
- Los correos pasan por `clean_email()` que estandariza y une con `;`.

## 2. Módulo: Importación Comercial (`/api/importar_datos_comerciales`)
### Objetivo
Actualizar los metadatos comerciales de los clientes para la generación de Certificados.
### Flujo del usuario
1. Se sube el archivo con hojas "BASE DE DATOS" y "VENTAS".
2. El sistema cruza los NITs limpios (ignorando `.0`).
3. Para la "BASE DE DATOS": asocia cupos, fechas de ingreso, y traduce la condición de pago mediante el diccionario interno `MAPEO_CONDICIONES`.
   **Regla Especial:** Si el NIT no existe en la base de datos (cliente sin mora), crea un perfil vacío (`total_cop = 0`) para retener la información.
4. Para "VENTAS": Calcula el total vendido dividido por los meses únicos de actividad para obtener el promedio mensual, y lo asigna a todas las filas del cliente.

## 3. Módulo: Generador de Cartas de Cobro (`generador_cobros.py`)
### Objetivo
Generar PDF formales exigiendo el pago de facturas y enviarlos por correo.
### Archivos principales
- `generador_cobros.py`
### Flujo y Lógica
1. Filtra registros con `total_cop > 0` (o por NIT si es individual).
2. Genera un PDF usando FPDF2, incluyendo logo, texto de la Ley 1266 de Habeas Data, tabla de facturas adeudadas y **Medios de Pago Autorizados** (Cuentas Davivienda, Popular).
3. **Manejo de Correo:**
   - Lee el campo correo. Si tiene múltiples (`;`), los divide.
   - **Filtro SIESA:** Descarta correos que incluyan el substring `"siesa"` (bots de facturación).
   - Toma el primer correo válido como `To:` y los demás como `Cc:`.
   - Se conecta vía `smtplib` a Outlook y dispara el PDF adjunto. Si no hay correo válido, abre el explorador de Windows para gestión manual.

## 4. Módulo: Cobro Masivo (`/cobro_masivo`)
### Objetivo
Automatizar el envío de cobros a toda la base de deudores críticos.
### Flujo del usuario
1. Ingresa a la interfaz de Cobro Masivo (`cobro_masivo.html`).
2. El backend envía solo los clientes con `dias_vencidos > 14` y `total_cop > 0`.
3. El usuario marca las casillas y presiona enviar. Se despliega un "spinner" de carga (UI no bloqueante).
4. El backend recorre el arreglo de clientes, llama al módulo de cobros por cada uno, y empaqueta en un `.zip` los PDFs de aquellos que fallaron o no tenían correo.
5. El sistema entrega el ZIP para descarga y actualiza la `fecha_gestion` de los correos exitosos.

## 5. Módulo: Certificado Comercial (`generador_certificado.py`)
### Objetivo
Generar el PDF de referencia comercial para terceros.
### Lógica
Extrae los campos comerciales (`fecha_ingreso`, `promedio_mensual`, `cupo_credito`, etc.) de la base de datos y ensambla el documento. Permite emitirlo incluso para clientes con saldo $0 gracias a la modificación del módulo comercial.
