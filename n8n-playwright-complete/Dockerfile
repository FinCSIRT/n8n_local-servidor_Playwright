FROM ubuntu:24.04

# Variables de entorno
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=America/Bogota
ENV NODE_ENV=production
ENV N8N_USER_FOLDER=/opt/n8n
ENV N8N_HOST=0.0.0.0
ENV N8N_PORT=5678
ENV N8N_PROTOCOL=http
# No descargar navegadores en esta imagen (usamos browser-server remoto)
ENV PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1
# (Opcional) asegura que Node pueda ver módulos globales fácilmente
ENV NODE_PATH=/usr/lib/node_modules

# Dependencias base
RUN apt-get update && \
    apt-get install -y --fix-missing --no-install-recommends \
        ca-certificates \
        curl \
        git \
        python3 \
        build-essential \
        iputils-ping \
        netcat-openbsd \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Node.js 22
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - && \
    apt-get install -y nodejs && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Verificar instalación
RUN node --version && npm --version

# Directorio de trabajo
WORKDIR /opt/n8n
RUN mkdir -p /opt/n8n/.n8n

# Instalar n8n
RUN npm install -g n8n@latest

# ⚠️ IMPORTANTE: usar SOLO playwright-core y fijarlo en 1.55.0
# (no instales 'playwright' full para evitar sombras de versiones)
ARG PW_VERSION=1.55.0
RUN npm rm -g playwright playwright-core || true \
 && npm cache clean --force \
 && npm i -g playwright-core@${PW_VERSION} \
 && node -e "console.log('playwright-core version:', require('playwright-core/package.json').version)"

# Módulos adicionales permitidos por n8n Code node
RUN npm install -g \
    axios \
    lodash \
    moment \
    cheerio \
    ws

# Limpiar caché npm
RUN npm cache clean --force

# Script de verificación (ajustado a hostname correcto)
RUN echo '#!/bin/bash\n\
echo "Node.js: $(node --version)"\n\
echo "N8N: $(n8n --version)"\n\
echo -n "playwright-core (require): " ; node -e "console.log(require(\"playwright-core/package.json\").version)"\n\
echo "Módulos globales:" ; npm ls -g --depth=0 | grep -E "playwright|n8n" || true\n\
echo "Probe puerto 3000 en playwright-server:"\n\
nc -zv playwright-server 3000 2>&1 || echo "No conectado a playwright-server:3000"\n\
' > /opt/check.sh && chmod +x /opt/check.sh

# Script de prueba de conexión WS (hostname correcto)
RUN echo 'const { chromium } = require("playwright-core");\n\
(async function test(){\n\
  try {\n\
    const browser = await chromium.connect("ws://playwright-server:3000/");\n\
    console.log("✅ Conectado a Playwright server");\n\
    await browser.close();\n\
  } catch (err) {\n\
    console.error("Error:", err.message);\n\
  }\n\
})();\n\
' > /opt/test.js

# Script de inicio
RUN echo '#!/bin/bash\n\
echo "N8N con Playwright Remote"\n\
echo "URL: http://0.0.0.0:5678"\n\
/opt/check.sh\n\
exec n8n start\n\
' > /opt/start.sh && chmod +x /opt/start.sh

EXPOSE 5678

HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD curl -f http://localhost:5678/healthz || exit 1

CMD ["/opt/start.sh"]
