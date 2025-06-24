FROM python:3.9-slim

ARG NAME
ARG NLP_CONF_FILE=presidio_analyzer/conf/transformers.yaml
ARG ANALYZER_CONF_FILE=presidio_analyzer/conf/default_analyzer.yaml
ARG RECOGNIZER_REGISTRY_CONF_FILE=presidio_analyzer/conf/default_recognizers.yaml
ENV PIP_NO_CACHE_DIR=1

ENV ANALYZER_CONF_FILE=${ANALYZER_CONF_FILE}
ENV RECOGNIZER_REGISTRY_CONF_FILE=${RECOGNIZER_REGISTRY_CONF_FILE}
ENV NLP_CONF_FILE=${NLP_CONF_FILE}

ENV PORT=3000
ENV WORKERS=1

WORKDIR /usr/bin/${NAME}

# Install essential build tools first (cached layer)
RUN apt-get update \
  && apt-get install -y build-essential \
  && rm -rf /var/lib/apt/lists/*

# Install poetry early (cached layer)
RUN pip install poetry

# Copy dependency files first for better caching
COPY ./pyproject.toml /usr/bin/${NAME}/
COPY ./README.md /usr/bin/${NAME}/

# Install dependencies (cached unless pyproject.toml changes)
RUN poetry install --no-root --only=main -E server -E transformers

# Copy configuration files needed for model installation
COPY ${ANALYZER_CONF_FILE} /usr/bin/${NAME}/${ANALYZER_CONF_FILE}
COPY ${RECOGNIZER_REGISTRY_CONF_FILE} /usr/bin/${NAME}/${RECOGNIZER_REGISTRY_CONF_FILE}
COPY ${NLP_CONF_FILE} /usr/bin/${NAME}/${NLP_CONF_FILE}

# Install nlp models (cached unless config changes)
COPY ./install_nlp_models.py /usr/bin/${NAME}/
RUN poetry run python install_nlp_models.py --conf_file ${NLP_CONF_FILE}

# Copy application code last (changes most frequently)
COPY . /usr/bin/${NAME}/

EXPOSE ${PORT}
CMD poetry run gunicorn -w $WORKERS -b 0.0.0.0:$PORT 'app:create_app()'
