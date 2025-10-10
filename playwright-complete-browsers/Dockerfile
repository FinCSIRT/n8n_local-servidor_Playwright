# Dockerfile para servidor Playwright personalizado
FROM mcr.microsoft.com/playwright:v1.56.0-noble

# Buenas prácticas de entorno
ENV TZ=America/Bogota \
    NODE_ENV=production \
    PLAYWRIGHT_PORT=3000 \
    PLAYWRIGHT_HOST=0.0.0.0

# Instalar dependencias adicionales como root
USER root
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Establecer directorio de trabajo
WORKDIR /home/pwuser

# Crear script de inicio personalizado
RUN cat > /home/pwuser/start-playwright-server.sh << 'EOF'
#!/usr/bin/env bash
set -euo pipefail

echo "🎭 Playwright Browser Server"
echo "=========================="
echo "Usuario: $(whoami)"
echo "Directorio: $(pwd)"
echo "Playwright CLI: $(npx playwright --version)"
echo "Puerto: ${PLAYWRIGHT_PORT:-3000}"
echo "Host: ${PLAYWRIGHT_HOST:-0.0.0.0}"
echo
echo "WebSocket Endpoint disponible en:"
echo "  ws://${PLAYWRIGHT_HOST:-0.0.0.0}:${PLAYWRIGHT_PORT:-3000}/"
echo
echo "Navegadores disponibles:"
ls -la /ms-playwright/ | grep -E "(chromium|firefox|webkit)" | head -3 || true
echo "=========================="
echo

echo "🚀 Iniciando servidor Playwright..."
# Usamos la versión incluida en la imagen base (1.54.0),
# evitando descargas en runtime y garantizando el match de versiones.
exec npx playwright run-server \
  --port "${PLAYWRIGHT_PORT:-3000}" \
  --host "${PLAYWRIGHT_HOST:-0.0.0.0}"
EOF

RUN chmod +x /home/pwuser/start-playwright-server.sh && \
    chown pwuser:pwuser /home/pwuser/start-playwright-server.sh

# Cambiar a usuario pwuser (el que trae la imagen oficial)
USER pwuser

# Exponer puerto
EXPOSE 3000

# Healthcheck (usa curl, que quedó instalado como root en el sistema)
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD curl -fsS http://localhost:3000/json/version -o /dev/null || exit 1

# Comando de inicio
CMD ["/home/pwuser/start-playwright-server.sh"]
