# Etapa de construcción
FROM python:3.11.9-slim-bullseye as builder

# Configuración del directorio de trabajo
WORKDIR /app

# Variables de entorno para evitar prompts interactivos
ARG DEBIAN_FRONTEND=noninteractive

# Actualización e instalación de dependencias necesarias
RUN apt-get update -qq \
  && apt-get -qqq install --no-install-recommends -y pkg-config gcc g++ \
  && apt-get upgrade --assume-yes \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# Crear un entorno virtual de Python
RUN python -mvenv venv && ./venv/bin/pip install --no-cache-dir --upgrade pip

# Copiar los archivos al contenedor
COPY . .

# Instalación de dependencias y configuración
RUN ./venv/bin/pip install Babel==2.12.1 \
  && ./venv/bin/python scripts/compile_locales.py \
  && ./venv/bin/pip install torch==2.0.1 --extra-index-url https://download.pytorch.org/whl/cpu \
  && ./venv/bin/pip install "numpy<2" \
  && ./venv/bin/pip install . \
  && ./venv/bin/pip cache purge

# Etapa final
FROM python:3.11.9-slim-bullseye

# Argumentos opcionales
ARG with_models=false
ARG models=""

# Crear un usuario sin privilegios
RUN addgroup --system --gid 1032 libretranslate \
  && adduser --system --uid 1032 libretranslate \
  && mkdir -p /home/libretranslate/.local \
  && chown -R libretranslate:libretranslate /home/libretranslate/.local

USER libretranslate

# Copiar los archivos desde la etapa de construcción
COPY --from=builder --chown=1032:1032 /app /app
WORKDIR /app

# Copiar el ejecutable ltmanage al path
COPY --from=builder --chown=1032:1032 /app/venv/bin/ltmanage /usr/bin/

# Inicializar los modelos si se solicita
RUN if [ "$with_models" = "true" ]; then \
  if [ ! -z "$models" ]; then \
    ./venv/bin/python scripts/install_models.py --load_only_lang_codes "$models"; \
  else \
    ./venv/bin/python scripts/install_models.py; \
  fi \
fi

# Exponer el puerto (para documentación)
EXPOSE 5000

# Comando de inicio, asegurándose de usar el puerto definido por Render
ENTRYPOINT ["./venv/bin/libretranslate", "--host", "0.0.0.0", "--port", "${PORT}"]
