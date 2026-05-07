# Essentia Music Tagger — Dockerfile
# Basado en: https://github.com/WB2024/Essentia-to-Metadata

FROM python:3.11-slim

# Dependencias del sistema necesarias para essentia-tensorflow
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    curl \
    ffmpeg \
    libsndfile1 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Clonar el repo de Essentia-to-Metadata
RUN git clone https://github.com/WB2024/Essentia-to-Metadata.git . 

# Instalar dependencias Python
# essentia-tensorflow puede tardar varios minutos la primera vez
RUN pip install --no-cache-dir \
    essentia-tensorflow \
    mutagen \
    numpy

# Descargar los modelos ML (~87MB) en tiempo de build
# Si falla aquí, se descargarán al primer run
RUN bash download_models.sh || echo "Modelos se descargarán en el primer run"

# El volumen /music se monta en runtime
# Los modelos se persisten en /app/models via volumen en compose
ENTRYPOINT ["python", "tag_music.py"]
CMD ["/music", "--auto"]
