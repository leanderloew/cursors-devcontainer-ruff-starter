# bring in the micromamba image so we can copy files from it
FROM mambaorg/micromamba:2.0.5 as micromamba

# This is your base image, choose whatever you wish
FROM lys95/seleniumbase:latest

USER root

ENV TZ=Etc/UTC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN apt-get update && apt-get install -y \
    curl \
    tar \
    wget \
    git \
    build-essential \
    ca-certificates \
    sudo \
    procps \
    vim \
    less \
    && rm -rf /var/lib/apt/lists/*

# Setting corrrect settings and copying files from the micromamba image
ARG MAMBA_USER=mambauser
ARG MAMBA_USER_ID=57439
ARG MAMBA_USER_GID=57439
ENV MAMBA_USER=$MAMBA_USER
ENV MAMBA_ROOT_PREFIX="/opt/conda"
ENV MAMBA_EXE="/bin/micromamba"

COPY --from=micromamba "$MAMBA_EXE" "$MAMBA_EXE"
COPY --from=micromamba /usr/local/bin/_activate_current_env.sh /usr/local/bin/_activate_current_env.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_shell.sh /usr/local/bin/_dockerfile_shell.sh
COPY --from=micromamba /usr/local/bin/_entrypoint.sh /usr/local/bin/_entrypoint.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_initialize_user_accounts.sh /usr/local/bin/_dockerfile_initialize_user_accounts.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_setup_root_prefix.sh /usr/local/bin/_dockerfile_setup_root_prefix.sh

RUN /usr/local/bin/_dockerfile_initialize_user_accounts.sh && \
    /usr/local/bin/_dockerfile_setup_root_prefix.sh

USER $MAMBA_USER
SHELL ["/usr/local/bin/_dockerfile_shell.sh"]
ENTRYPOINT ["/usr/local/bin/_entrypoint.sh"]

## Install python packages
COPY /env/env.yml /tmp/

RUN micromamba install -y -n base -f /tmp/env.yml --verbose \
    && micromamba clean --all --yes

## Set up the workspace
ENV PYTHONPATH=/workspace/src
WORKDIR /workspace

COPY src /workspace/src

CMD gunicorn -k uvicorn.workers.UvicornWorker \
    -t $TIMEOUT \
    -w $UVI_WORKERS \
    --max-requests 1000 \
    --max-requests-jitter 50 \
    --bind "localhost:$PORT" \
    --preload \
    --graceful-timeout 600 \
    $FASTAPI_APP