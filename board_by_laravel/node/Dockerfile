FROM node:24-bookworm-slim

RUN apt-get update \
	&& apt-get install -y --no-install-recommends procps \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /var/git/www.wderfer6790.kr

EXPOSE 5173