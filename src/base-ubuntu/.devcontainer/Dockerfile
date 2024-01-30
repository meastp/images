# [Choice] Ubuntu version (use jammy on local arm64/Apple Silicon): jammy, focal
ARG VARIANT="lunar"
FROM buildpack-deps:${VARIANT}-curl

RUN cat /etc/passwd

LABEL dev.containers.features="common"