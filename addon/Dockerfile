ARG BUILD_FROM
FROM ${BUILD_FROM}

# System dependencies. Python 3.14 + pip + venv come from the base image.
RUN apk add --no-cache \
    bash \
    ffmpeg \
    alsa-lib-dev \
    portaudio-dev \
    git \
    curl \
    jq

# Clone Hermes Agent and this integration bundle
WORKDIR /opt
ARG HERMES_REPO=https://github.com/NousResearch/hermes-agent.git
ARG HERMES_REF=main
ARG INTEGRATION_REPO=https://github.com/rusty4444/hermes-voice-ha-integration.git
ARG INTEGRATION_REF=main
RUN git clone --depth 1 --branch ${HERMES_REF} ${HERMES_REPO} hermes-agent && \
    git clone --depth 1 --branch ${INTEGRATION_REF} ${INTEGRATION_REPO} hermes-voice-ha-integration

# Python virtual environment with Hermes + voice stack deps (Python 3.14 => core deps install)
WORKDIR /opt/hermes-agent
RUN python3 -m venv .venv && \
    .venv/bin/pip install --upgrade pip setuptools wheel && \
    .venv/bin/pip install -e . && \
    .venv/bin/pip install \
    "aiohttp>=3.9" \
    "pydantic>=2.0" \
    "edge-tts>=6.1" \
    "sounddevice>=0.4" \
    "numpy>=1.26"

# Install Hermes HA integration plugins
RUN cp -R /opt/hermes-voice-ha-integration/plugins/home_assistant /opt/hermes-agent/plugins/home_assistant && \
    cp -R /opt/hermes-voice-ha-integration/plugins/voice_stack /opt/hermes-agent/plugins/voice_stack && \
    cp -R /opt/hermes-voice-ha-integration/custom_components /opt/hermes-agent/custom_components

# Configuration
COPY run.sh /opt/run.sh
RUN chmod +x /opt/run.sh

RUN mkdir -p /data/hermes /data/hermes/logs /data/hermes/voice_cache
ENV HOME=/data/hermes
ENV HERMES_HOME=/data/hermes
ENV HERMES_VOICE_CACHE=/data/hermes/voice_cache

EXPOSE 7860
ENTRYPOINT ["/opt/run.sh"]
