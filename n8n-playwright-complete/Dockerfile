FROM ubuntu:24.04

# Environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Europe/Budapest
ENV NODE_ENV=production
ENV N8N_USER_FOLDER=/opt/n8n
ENV N8N_HOST=0.0.0.0
ENV N8N_PORT=5678
ENV N8N_PROTOCOL=http
# Do not download browsers in this image (we use a remote browser-server)
ENV PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1
# (Optional) ensures Node can easily see global modules
ENV NODE_PATH=/usr/lib/node_modules

# Base dependencies
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

# Verify installation
RUN node --version && npm --version

# Working directory
WORKDIR /opt/n8n
RUN mkdir -p /opt/n8n/.n8n

# Install n8n
RUN npm install -g n8n@latest

# ⚠️ IMPORTANT: use ONLY playwright-core and pin it to 1.55.0
# (do not install the full 'playwright' to avoid version shadows)
ARG PW_VERSION=1.55.0
RUN npm rm -g playwright playwright-core || true \
 && npm cache clean --force \
 && npm i -g playwright-core@${PW_VERSION} \
 && node -e "console.log('playwright-core version:', require('playwright-core/package.json').version)"

# Additional modules allowed by n8n Code node
RUN npm install -g \
    axios \
    lodash \
    moment \
    cheerio \
    ws

# Clear npm cache
RUN npm cache clean --force

# Verification script (adjusted to correct hostname)
RUN echo '#!/bin/bash\n\
echo "Node.js: $(node --version)"\n\
echo "N8N: $(n8n --version)"\n\
echo -n "playwright-core (require): " ; node -e "console.log(require(\"playwright-core/package.json\").version)"\n\
echo "Global modules:" ; npm ls -g --depth=0 | grep -E "playwright|n8n" || true\n\
echo "Probing port 3000 on playwright-server:"\n\
nc -zv playwright-server 3000 2>&1 || echo "Not connected to playwright-server:3000"\n\
' > /opt/check.sh && chmod +x /opt/check.sh

# WS connection test script (correct hostname)
RUN echo 'const { chromium } = require("playwright-core");\n\
(async function test(){\n\
  try {\n\
    const browser = await chromium.connect("ws://playwright-server:3000/");\n\
    console.log("✅ Connected to Playwright server");\n\
    await browser.close();\n\
  } catch (err) {\n\
    console.error("Error:", err.message);\n\
  }\n\
})();\n\
' > /opt/test.js

# Start script
RUN echo '#!/bin/bash\n\
echo "N8N with Remote Playwright"\n\
echo "URL: http://0.0.0.0:5678"\n\
/opt/check.sh\n\
exec n8n start\n\
' > /opt/start.sh && chmod +x /opt/start.sh

EXPOSE 5678

HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD curl -f http://localhost:5678/healthz || exit 1

CMD ["/opt/start.sh"]
