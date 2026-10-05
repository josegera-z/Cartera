# DOCUMENTO DE HANDOVER (Entrega Técnica)

## 1. Resumen de la Entrega
Se hace entrega formal del **Sistema Integral de Cartera y Facturación**, un aplicativo web local construido en Python (FastAPI) que centraliza la administración de deudores, automatiza la emisión de cartas de cobro masivas, gestiona datos comerciales y emite certificados en PDF.

## 2. Estado Actual del Proyecto
El proyecto se entrega en estado **FUNCIONAL Y ESTABLE**. Todas las funcionalidades requeridas en el último ciclo de desarrollo han sido implementadas, probadas y subidas al repositorio principal de GitHub.

**Funcionalidades Operativas:**
- Importación de cartera desde Excel (`importar_excel`).
- Importación de datos comerciales y ventas (`importar_datos_comerciales`).
- Interfaz interactiva de gestión de clientes e historial de observaciones.
- Generación de Cartas de Cobro en PDF con medios de pago dinámicos.
- Envío automático de correos (soporte multi-correo separado por `;`).
- Filtro inteligente anti-rebote (bloqueo de envíos a direcciones `SIESA`).
- Generación Masiva de Cobros (procesamiento en lote y exportación en archivo `.zip`).
- Generación de Certificados Comerciales (creación de clientes sin mora para posibilitar certificación).

## 3. Componentes Entregados
- **Código Fuente:** Repositorio alojado en GitHub (`github.com/JAAV-01/Cartera`).
- **Scripts:** `start_server.bat` para inicialización rápida.
- **Documentación Técnica:** Carpeta `/docs` con información detallada de base de datos, módulos, solución de problemas (troubleshooting) y arquitectura.

## 4. Accesos Necesarios
Para la correcta operación del sistema, la empresa o el próximo desarrollador debe custodiar:
- Credenciales de la Base de Datos SQL Server (`192.168.1.14`).
- Credenciales SMTP de la cuenta Outlook (`credito_cartera@josegera.com`).
- Acceso de administrador al repositorio de GitHub para futuros despliegues o clones.

*(Nota: Las credenciales reales no se entregan en el código por seguridad. Deben ser configuradas en el archivo `.env` de cada equipo local).*

## 5. Backups y Restauración
- **Base de Datos:** El backup de `cartera_db` es responsabilidad del equipo de TI que administra el SQL Server.
- **Código Fuente:** GitHub funge como el backup del código. En caso de daño en una máquina local, basta con clonar el repositorio de nuevo.

## 6. Documentación Disponible
- `README.md`: Guía de instalación y stack tecnológico.
- `docs/DATABASE.md`: Estructura de la base de datos y tratamiento de modelos.
- `docs/MODULES.md`: Lógica detallada por módulo.
- `docs/TROUBLESHOOTING.md`: Guía de resolución de problemas comunes (decimales de pandas, bug de múltiples facturas, etc.).
- `docs/ARCHITECTURE.md`: Arquitectura, decisiones técnicas y flujos de negocio.

## 7. Riesgos y Recomendaciones
- **Dependencia de la estructura de Excel:** El sistema lee columnas específicas ("Valor neto local", "Valor subtotal local", "Código", etc.). Si el área contable cambia radicalmente los nombres de las columnas exportadas por su ERP, el sistema (pandas) podría fallar en la importación. *Recomendación:* Estandarizar la plantilla de Excel en la empresa.
- **Correos múltiples:** El sistema asume que los correos múltiples en Excel vienen separados por `;` o que el usuario los separará así en la UI.
- **Filtro SIESA:** Actualmente programado (hardcoded) para detectar el string `"siesa"`. Si hay otros bots, deberán agregarse a la condición en `generador_cobros.py`.

## 8. Puntos a Revisar por el Próximo Desarrollador
- La tabla `cartera` actualmente funciona tanto como entidad de *Cliente* como de *Factura* (hay múltiples filas por NIT). Se mitigó el problema de sobrescritura usando arreglos (`clientes_dict[nit] = []`), pero a futuro, si el sistema crece, se sugiere separar en una tabla `clientes` y otra `facturas`.
- Comprender la función `clean_email()` en `app.py`, la cual es vital para el proceso de validación y separación de correos.

---
**Firma de Entrega:** Antigravity / IA Developer
**Fecha:** Octubre 2026
