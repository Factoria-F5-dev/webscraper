## Django Screper

## Cuarto paso: Capturas de pantalla

En esta rama se ha implementado la funcionalidad para tomar capturas de pantalla durante el proceso de web scraping. Los cambios incluyen:

### 1. Configuración del Sistema de Capturas

Se modificó el archivo `scrape.py` para incluir la funcionalidad de capturas de pantalla:
- Se agregaron las importaciones necesarias (`os` y `datetime`)
- Se implementó la captura de pantalla después de cargar la página
- Las capturas se guardan con timestamp único en el nombre

### 2. Configuración de Docker

Se realizaron modificaciones en los archivos de Docker para manejar las capturas:

#### Dockerfile
```dockerfile
# Create screenshots directory and set permissions
RUN mkdir -p /app/screenshots && chmod 777 /app/screenshots

# Create volume for screenshots para acceder a ellas
VOLUME ["/app/screenshots"]
```

#### compose.yaml
```yaml
services:
  server:
    volumes:
      - ./screenshots:/app/screenshots
```

### 3. Gestión de Archivos

- Se creó un directorio `screenshots/` en la raíz del proyecto
- Se agregó `screenshots/` al `.gitignore` para evitar versionar las capturas
- Las capturas se guardan con el formato: `captura_YYYYMMDD_HHMMSS.png`

### 4. Estructura de Directorios

```
webscraper_project/
    ├── screenshots/           # Directorio para las capturas
    │   └── captura_*.png     # Capturas generadas
    ├── Dockerfile
    ├── compose.yaml
    └── webscraper_project/
        └── scraper/
            └── services/
                └── scrape.py  # Script modificado
```

### 5. Funcionamiento

1. Durante el scraping, se toma una captura después de cargar la página
2. La captura se guarda en el directorio `screenshots/`
3. El nombre del archivo incluye timestamp para evitar sobrescrituras
4. Las capturas son accesibles tanto desde el contenedor como desde el sistema local

### 6. Acceso a las Capturas

- **Desde el contenedor:** `/app/screenshots/`
- **Desde el sistema local:** `./screenshots/`

### 7. Comandos Útiles

```bash
# Ejecutar el scraper
python webscraper_project/manage.py scraper

# Ver las capturas guardadas
ls screenshots/

# Acceder a las capturas desde el contenedor
ls /app/screenshots/
```

Esta implementación permite un flujo de trabajo eficiente donde las capturas de pantalla se almacenan de forma organizada y son fácilmente accesibles tanto desde el contenedor como desde el sistema host.

### 8. Más

Anti-bots
```python
import undetected_chromedriver as uc
pip install undetected-chromedriver
# Se actualiza constantemente para adaptarse a los cambios en los mecanismos anti-bots de sitios como Google, LinkedIn, Instagram, etc.
```

ScraperAPI. Una API que gestiona rotación de IPs, resolución de CAPTCHAs, renderizado de JavaScript y manejo de headers automáticamente para que tu scraper no sea detectado.

Scrapear sin permiso puede ser considerado ilegal si viola términos de uso o derechos de propiedad intelectual.
Muchas webs incluyen términos de servicio que prohíben explícitamente el scraping. Scrapear sin permiso puede ser considerado ilegal si viola términos de uso o derechos de propiedad intelectual. Muchas webs usan este archivo para definir si permiten o no el scraping automatizado: robots.txt.
