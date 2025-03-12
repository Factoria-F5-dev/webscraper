## Django Screper

## Segundo paso: Docker y firefox

- Entiendo que tienes instalado docker...

- En el directorio raiz del proyecto

- docker init

```bash
- ? What application platform does your project use? Python
- ? What version of Python do you want to use? 3.11.11 (¿python3 --version?)
- ? What port do you want your app to listen on? 8000
- ? What is the command you use to run your app? python3 webscraper_project/manage.py scraper 
```

- docker compose up --build # no funciona porque da problemas con el chrome, vamos autilizar el Firefox, instalarlo en el dockerfile y cambiar el comando de ejecución.

dockerfile
```bash
# syntax=docker/dockerfile:1

ARG PYTHON_VERSION=3.11.11
FROM python:${PYTHON_VERSION}-slim as base

# Prevents Python from writing pyc files and keeps Python from buffering stdout and stderr
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Set working directory
WORKDIR /app

# Create a non-privileged user that the app will run under
# ARG UID=10001
# RUN adduser \
#     --disabled-password \
#     --gecos "" \
#     --home "/nonexistent" \
#     --shell "/sbin/nologin" \
#     --no-create-home \
#     --uid "${UID}" \
#     appuser

# Switch to root to install system dependencies
USER root
RUN apt-get update && apt-get install -y \
    wget \
    unzip \
    curl \
    firefox-esr \
    libx11-xcb1 \
    libxtst6 \
    libxrender1 \
    libdbus-glib-1-2 \
    libgtk-3-0 \
    libasound2 \
    fonts-liberation \
    libgl1-mesa-dri \
    libpci3 \
    && rm -rf /var/lib/apt/lists/*

# Configure Fontconfig to avoid cache errors
RUN mkdir -p /tmp/cache/fontconfig && chmod 777 /tmp/cache/fontconfig
ENV FONTCONFIG_PATH=/tmp/cache/fontconfig

# Assign a valid home directory to appuser
# RUN usermod -d /home/appuser appuser && mkdir -p /home/appuser && chown appuser:appuser /home/appuser

# Download and install GeckoDriver
RUN case $(dpkg --print-architecture) in \
    amd64) ARCH=linux64 ;; \
    arm64) ARCH=linux-aarch64 ;; \
    *) echo "Unsupported architecture" && exit 1 ;; \
    esac && \
    wget -O /tmp/geckodriver.tar.gz https://github.com/mozilla/geckodriver/releases/download/v0.35.0/geckodriver-v0.35.0-$ARCH.tar.gz && \
    tar -xvzf /tmp/geckodriver.tar.gz -C /usr/local/bin/ && \
    chmod +x /usr/local/bin/geckodriver && \
    rm /tmp/geckodriver.tar.gz

# Install Python dependencies
COPY requirements.txt /app/
RUN python -m pip install --no-cache-dir -r requirements.txt

# Ensure permissions for SQLite database and source code
RUN mkdir -p /app/webscraper_project && \
    touch /app/webscraper_project/db.sqlite3 && \
    chmod -R 777 /app/webscraper_project && \
    chmod -R 777 /app

# Copy application source code
COPY . .

# Switch to non-privileged user
#USER appuser

# Expose the port used by the application
EXPOSE 8000

# Default command to keep the container running
CMD ["tail", "-f", "/dev/null"]
```

y el scraper

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

from selenium.webdriver.firefox.service import Service
from selenium.webdriver.firefox.options import Options

def scrape_website():
    # Configurar Selenium
    options = Options()
    options.add_argument('--headless')  # Ejecutar en modo headless
    options.add_argument('--no-sandbox')  # Requerido para algunos servidores
    options.add_argument('--disable-dev-shm-usage')  # Para evitar errores de memoria

    # Configurar el servicio de GeckoDriver
    service = Service("/usr/local/bin/geckodriver")
    # Crear el WebDriver de Firefox
    driver = webdriver.Firefox(service=service, options=options)

    # Navegar al sitio web
    url = "https://jorgebenitezlopez.com"
    driver.get(url)
    print(driver.title)  
# Esperar a que los elementos estén presentes
    try:
        WebDriverWait(driver, 10).until(
            EC.presence_of_all_elements_located((By.CSS_SELECTOR, "h1"))
        )
        titles = driver.find_elements(By.CSS_SELECTOR, "h1")
        urls = driver.find_elements(By.CSS_SELECTOR, "a")
    except Exception as e:
        print("Error al encontrar los elementos:", e)
        driver.quit()
        return []

    scraped_data = []
    for title, link in zip(titles, urls):
        scraped_data.append({
            "title": title.text,
            "url": link.get_attribute("href"),
        })

    print("Scraped data:", scraped_data)  # Para depuración
    driver.quit()
    return scraped_data
```

- Ahora el contenedor se queda activado
- Instala diferentes versiones del navegador dependiendo del tipo de imagen
- Compruebo
```bash
docker ps
docker exec -it name bash
docker exec -it webscraper-server-1 /usr/local/bin/python webscraper_project/manage.py scraper

👋 Shall we take a look around?
Scraped data: [{'title': '👋 Hello!', 'url': 'https://factoriaf5.org/somos/#equipo'}, {'title': '', 'url': 'https://cristinamaser.com/'}]
Scraped Data: [{'title': '👋 Hello!', 'url': 'https://factoriaf5.org/somos/#equipo'}, {'title': '', 'url': 'https://cristinamaser.com/'}]
Scraping completed!

docker exec -it webscraper-server-1 bash
ls -l db.sqlite3
apt-get update && apt-get install -y sqlite3
sqlite3 db.sqlite3
.tables
SELECT * FROM scraper_scrapeddata;
```

En el caso de que falle firefox. Consejos para depurar:

```bash
docker ps (cogemos name del contenedor)
docker exec -it name bash (para acceder y analizar la estructura de carpetas y que existe geckodriver y firefox)
appuser@4f2e7276d3cc:/app$ which geckodriver
appuser@4f2e7276d3cc:/app$ which firefox
tienen que exisitir
docker exec -it name /usr/local/bin/geckodriver --log debug (Dejar abierta esta ventana para ver errores)
```

