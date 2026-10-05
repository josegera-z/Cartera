# Troubleshooting e Historial del Proyecto

## Problemas Frecuentes y Soluciones

### 1. Sistema asume "Múltiples Clientes" o Falla al cruzar NITs desde Excel
- **Síntoma:** Al subir datos comerciales, clientes como "DUQUE GARCIA" no se actualizan o quedan con saldo vacío.
- **Posible Causa:** Pandas interpreta los NITs nulos/vacíos en Excel como tipos numéricos `float`, añadiendo un decimal `.0` (Ej: `3607497.0`). La regex de limpieza anterior extraía todos los números, transformándolo erróneamente en `36074970`.
- **Solución implementada:** Se incluyó el método `.split('.')[0]` antes del `re.sub(r'\D', '', ...)` para ignorar la porción decimal forzada.
- **Archivos:** `app.py` (Línea ~755).

### 2. Rebotes y Bloqueos de Envío de Correos (Filtro SIESA)
- **Síntoma:** Cartas de cobro se envían a buzones de software (recepcion de facturación electrónica) como `sisaferecepcion@siesafe.co`, resultando en rebotes o quejas.
- **Solución implementada:** En `generador_cobros.py`, se implementó una evaluación `if "siesa" not in e.lower():` al dividir los correos. Elimina exclusivamente el correo del bot, preservando los demás correos humanos del cliente.
- **Archivos:** `generador_cobros.py`.

### 3. Actualizaciones comerciales solo aplican a la última factura
- **Síntoma:** Al emitir un certificado de un cliente con 5 facturas en la base de datos, los datos salían en blanco o erróneos dependiendo desde cuál registro se emitiera.
- **Causa:** El mapeo en memoria usaba un diccionario que sobrescribía el objeto cliente (`clientes_dict[nit] = cliente`), guardando únicamente el último.
- **Solución implementada:** Se estructuró un diccionario de arreglos (`clientes_dict[nit] = []` -> `.append(cliente)`). El código luego itera y asigna los datos a todos los registros vinculados.

## Historial de Mejoras (Reconstrucción)
| Fecha | Módulo | Cambio | Motivo |
|---|---|---|---|
| Jun 2026 | API Comercial | Ajuste promedio comercial a Subtotal | Regla de negocio: ventas medidas antes de impuestos. |
| Sep 2026 | Base de Datos | Clientes vacíos | Permitir certificados para usuarios sin mora activa. |
| Sep 2026 | Interfaz & API | Soporte multicorreo (`;`) | Evitar bloqueos HTML5 y centralizar comunicaciones. |
| Sep 2026 | PDF Cobros | Inclusión de cuentas bancarias | Visibilidad de medios de pago en PDF final. |
| Sep 2026 | PDF Cobros | Filtro "SIESA" (Bisturí) | Prevenir envíos inútiles a bots del ERP. |

## Archivos Críticos a Entender
| Archivo | Función | Importancia |
|---|---|---|
| `app.py` | Motor de importación y limpieza (Pandas). | **CRÍTICA**. Modificar las reglas `regex` sin cuidado romperá los cruces entre base de datos y Excel. |
| `generador_cobros.py` | Envío de correos y renderizado FPDF. | **ALTA**. Maneja el SMTP. Un error de indentación bloqueará los cobros masivos. |
| `models.py` | Estructura SQL. | **MEDIA**. Entender que `Cliente` funge como *factura individual* es vital para no romper consultas. |

## Mantenimiento Recomendado
- **Caché/Almacenamiento:** Purgar periódicamente las carpetas `cartas_generadas/` y `certificados_generados/` mediante un script de sistema operativo o tarea cronográfica para evitar saturación del disco duro del servidor local.
