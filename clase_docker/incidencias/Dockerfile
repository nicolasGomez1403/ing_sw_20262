# ===== ETAPA 1: CONSTRUCCIÓN =====
FROM python:3.14-slim AS builder

WORKDIR /app

# Copiar uv desde su imagen oficial
COPY --from=ghcr.io/astral-sh/uv:0.12.17 /uv /usr/local/bin/uv

# Crear venv e instalar dependencias
COPY requirements.txt .
RUN uv venv /opt/venv && \
    . /opt/venv/bin/activate && \
    uv pip install --no-cache -r requirements.txt

# ===== ETAPA 2: PRODUCCIÓN =====
FROM python:3.14-slim

# Crear usuario sin privilegios
RUN useradd --create-home --shell /bin/bash appuser

WORKDIR /app

# Copiar el venv ya resuelto
COPY --from=builder /opt/venv /opt/venv

# Copiar solo el código (sin __pycache__ por .dockerignore)
COPY api/ ./api/

# Dar permisos al usuario
RUN chown -R appuser:appuser /app

# Poner el venv en el PATH
ENV PATH="/opt/venv/bin:$PATH"

# Cambiar a usuario sin privilegios
USER appuser

EXPOSE 8080

# Healthcheck con Python (no hay curl en slim)
HEALTHCHECK --interval=10s --timeout=3s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8080/health')" || exit 1

# CMD en formato exec (importante para docker stop rápido)
CMD ["uvicorn", "api.main:app", "--host", "0.0.0.0", "--port", "8080"]