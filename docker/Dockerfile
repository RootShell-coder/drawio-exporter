FROM debian:bullseye

ENV DEBIAN_FRONTEND=noninteractive \
    DRAWIO_VERSION=28.0.6 \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8 \
    LANGUAGE=en_US:ru_RU

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    ca-certificates wget gnupg2 locales fontconfig \
    libgtk-3-0 libnotify4 libnss3 libxss1 libxtst6 xdg-utils libatspi2.0-0 libsecret-1-0 \
    libasound2 libx11-xcb1 libxcomposite1 libxdamage1 libxrandr2 libxext6 libxfixes3 libxi6 libxcb1 \
    libgbm1 libxkbfile1 libglib2.0-0 \
    xvfb xauth libxrender1 libxshmfence1 libdrm2 libpango-1.0-0 libcairo2 \
    fonts-dejavu-core fonts-noto-core fonts-liberation2 fonts-noto-extra \
    librsvg2-bin && \
    rm -rf /var/lib/apt/lists/*

# Локали тутЪ
RUN sed -i 's/# \?ru_RU.UTF-8 UTF-8/ru_RU.UTF-8 UTF-8/' /etc/locale.gen && \
    sed -i 's/# \?en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    locale-gen ru_RU.UTF-8 en_US.UTF-8 && \
    update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 LANGUAGE=en_US:ru_RU && \
    fc-cache -f

# Загрузка и установка draw.io
RUN wget -O /tmp/drawio.deb \
    https://github.com/jgraph/drawio-desktop/releases/download/v${DRAWIO_VERSION}/drawio-amd64-${DRAWIO_VERSION}.deb && \
    apt-get update && apt-get install -y /tmp/drawio.deb || (apt-get -f install -y && apt-get install -y /tmp/drawio.deb) && \
    rm -f /tmp/drawio.deb && \
    rm -rf /var/lib/apt/lists/*

# entrypoint
COPY entrypoint /usr/local/bin/entrypoint
RUN chmod +x /usr/local/bin/entrypoint

WORKDIR /data

ENTRYPOINT ["entrypoint"]
