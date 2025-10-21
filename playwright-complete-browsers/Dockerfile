# Dockerfile for custom Playwright server
FROM mcr.microsoft.com/playwright:v1.56.0-noble

# Environment best practices
ENV TZ=Europe/Budapest \
    NODE_ENV=production \
    PLAYWRIGHT_PORT=3000 \
    PLAYWRIGHT_HOST=0.0.0.0

# Install additional dependencies as root
USER root
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /home/pwuser

# Create custom start-up script
RUN cat > /home/pwuser/start-playwright-server.sh << 'EOF'
#!/usr/bin/env bash
set -euo pipefail

echo "🎭 Playwright Browser Server"
echo "=========================="
echo "User: $(whoami)"
echo "Directory: $(pwd)"
echo "Playwright CLI: $(npx playwright --version)"
echo "Port: ${PLAYWRIGHT_PORT:-3000}"
echo "Host: ${PLAYWRIGHT_HOST:-0.0.0.0}"
echo
echo "WebSocket Endpoint available at:"
echo "  ws://${PLAYWRIGHT_HOST:-0.0.0.0}:${PLAYWRIGHT_PORT:-3000}/"
echo
echo "Available browsers:"
ls -la /ms-playwright/ | grep -E "(chromium|firefox|webkit)" | head -3 || true
echo "=========================="
echo

echo "🚀 Starting Playwright server..."
# We use the version included in the base image (matching v1.56.0),
# avoiding runtime downloads and ensuring version match.
exec npx playwright run-server \
  --port "${PLAYWRIGHT_PORT:-3000}" \
  --host "${PLAYWRIGHT_HOST:-0.0.0.0}"
EOF

RUN chmod +x /home/pwuser/start-playwright-server.sh && \
    chown pwuser:pwuser /home/pwuser/start-playwright-server.sh

# Switch to pwuser (the user from the official image)
USER pwuser

# Expose port
EXPOSE 3000

# Healthcheck (uses curl, which was installed as root in the system)
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD curl -fsS http://localhost:3000/json/version -o /dev/null || exit 1

# Start command
CMD ["/home/pwuser/start-playwright-server.sh"]
