FROM python:3-slim as builder

# Alternatively use ADD https:// (which will not be cached by Docker builder)
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl \
    && echo "Pulling watchdog binary from Github." \
    && curl -sSLf https://github.com/openfaas-incubator/of-watchdog/releases/download/0.5.1/of-watchdog > /usr/bin/fwatchdog \
    && chmod +x /usr/bin/fwatchdog \
    && apt-get remove -y curl \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && rm -rf /var/lib/apt/lists/*

ENV LIBRARY_PATH=/lib:/usr/lib
WORKDIR /wheels
COPY ./requirements.txt .
RUN pip wheel -r requirements.txt

# Multi-stage build starts
FROM python:3-slim
ARG ADDITIONAL_PACKAGE

RUN printf "deb http://cdn-fastly.deb.debian.org/debian stretch-backports main non-free\ndeb-src http://cdn-fastly.deb.debian.org/debian stretch-backports main non-free" > /etc/apt/sources.list.d/backports.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends -t stretch-backports \
        tesseract-ocr \
        tesseract-ocr-osd \
        tesseract-ocr-equ \
        tesseract-ocr-eng \
        tesseract-ocr-chi-sim \
        tesseract-ocr-chi-sim-vert \
        tesseract-ocr-chi-tra \
        tesseract-ocr-chi-tra-vert \
        tesseract-ocr-jpn \
        tesseract-ocr-jpn-vert \
        tesseract-ocr-kor \
        tesseract-ocr-kor-vert \
        tesseract-ocr-epo \
        tesseract-ocr-enm \
        tesseract-ocr-grc \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /usr/bin/fwatchdog /usr/bin/fwatchdog
COPY --from=builder /wheels /wheels
COPY ./requirements.txt .
RUN pip install -r requirements.txt -f /wheels

WORKDIR /root/
COPY ./index.py .

COPY function   function
RUN touch ./function/__init__.py

ENV fprocess="python3 index.py"

HEALTHCHECK --interval=1s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]
