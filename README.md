# Sistema Integral de Cartera y Facturación - JOSÉ A Y GERARDO E ZULUAGA S.A.S.

## Información General
- **Nombre del proyecto:** Sistema Integral de Cartera y Facturación
- **Descripción:** Aplicación web local desarrollada para automatizar la gestión de cartera, generación de cartas de cobro, importación de datos comerciales y emisión de certificados comerciales.
- **Propósito del sistema:** Agilizar el proceso de cobro y certificación comercial, reduciendo el trabajo manual mediante la generación automática de PDFs y el envío de correos electrónicos a través del servidor SMTP corporativo.
- **Problema que resuelve:** Elimina la necesidad de redactar cartas de cobro manualmente y adjuntar facturas una por una. Soluciona el envío masivo de notificaciones de mora, filtra bots de recepción de facturación (ej. SIESA) y consolida la información comercial en un formato unificado.
- **Usuarios o áreas que utilizan el sistema:** Área de Cartera, Crédito y Contabilidad.
- **Estado actual del proyecto:** FUNCIONAL Y ESTABLE.
- **Alcance actual:** El sistema es capaz de importar datos desde Excel, actualizar bases de datos, gestionar información de clientes, generar PDFs (cobros y certificados), enviar correos individuales y masivos, y exportar reportes.

## Stack Tecnológico
- **Lenguaje:** Python 3.10+
- **Framework Web:** FastAPI
- **Servidor ASGI:** Uvicorn
- **Base de Datos:** SQL Server (MSSQL)
- **ORM:** SQLAlchemy
- **Procesamiento de Datos:** Pandas
- **Generación de PDF:** FPDF2
- **Motor de Plantillas:** Jinja2
- **Frontend:** HTML5, CSS3, Bootstrap (vía clases CSS estandarizadas)
- **Servicios Externos / APIs:** SMTP Outlook (envío de correos)

## Arquitectura
El sistema posee una arquitectura monolítica orientada a servicios locales. 

```text
[Navegador del Usuario] <---(HTTP/HTML)---> [FastAPI + Jinja2 (app.py)]
                                                  |
                                                  v
[Archivos Excel] --(Pandas)--> [Controladores] <-----(SQLAlchemy)-----> [SQL Server (cartera_db)]
                                                  |
                                                  v
[FPDF2] --(Generación PDF)--> [Sistema de Archivos Local (cartas_generadas/)]
                                                  |
                                                  v
                                     [SMTP Outlook (Envío de correos)]
```

## Estructura del Proyecto
```text
/Cartera
├── app.py                      # Archivo principal de FastAPI, enrutamiento y controladores lógicos.
├── database.py                 # Configuración de conexión a SQL Server y SQLAlchemy SessionLocal.
├── models.py                   # Definición de modelos ORM (Cliente, Observacion).
├── crud.py                     # Operaciones de base de datos (Data Access Layer).
├── generador_cobros.py         # Lógica de creación de PDF de cobro y envío SMTP.
├── generador_certificado.py    # Lógica de creación de PDF para certificados comerciales.
├── templates/                  # Plantillas HTML procesadas por Jinja2 (index, cliente, cobro_masivo).
├── static/                     # Archivos estáticos (CSS, JS).
├── assets/                     # Recursos gráficos (logos, firmas).
├── cartas_generadas/           # Directorio donde se guardan temporalmente los PDF de cobro y archivos ZIP.
├── certificados_generados/     # Directorio de salida de los certificados comerciales.
├── requirements.txt            # Dependencias del proyecto Python.
├── start_server.bat            # Script autoejecutable para levantar el servidor localmente.
└── docs/                       # Documentación técnica avanzada (entrega del proyecto).
```

## 2. Instalación del Proyecto

### Requisitos Previos
- Python 3.10 o superior.
- Git.
- Acceso a la base de datos SQL Server corporativa.
- Credenciales del correo Outlook emisor.

### Configuración e Instalación
1. Clonar el repositorio:
   ```bash
   git clone https://github.com/JAAV-01/Cartera.git
   cd Cartera
   ```
2. Crear y activar el entorno virtual:
   ```bash
   python -m venv venv
   # En Windows:
   .\venv\Scripts\activate
   ```
3. Instalar dependencias:
   ```bash
   pip install -r requirements.txt
   ```
4. Crear el archivo `.env` en la raíz del proyecto. **NO incluir este archivo en el control de versiones**.

### Variables de Entorno (.env)
```env
# Base de datos (Custodiado por TI / Base de Datos)
DB_USER=desarrollojosea
DB_PASS=
DB_HOST=192.168.1.14
DB_PORT=1433
DB_NAME=cartera_db
DB_ENCRYPT=yes
DB_TRUSTSERVERCERT=yes

# Correo SMTP (Custodiado por TI / Cartera)
SMTP_USER=credito_cartera@josegera.com
SMTP_PASS=
SMTP_HOST=smtp-mail.outlook.com
SMTP_PORT=587
```

### Ejecución
Para iniciar el sistema en modo desarrollo o producción local:
```bash
.\start_server.bat
```
*(O alternativamente: `uvicorn app:app --host 0.0.0.0 --port 8000 --reload`)*

Para mayor detalle técnico, revisar la carpeta `/docs`.
