FROM quay.io/operator-framework/helm-operator:v0.10.0
MAINTAINER Pure Storage, Inc.
LABEL name="pure-flex" vendor="Pure Storage" version="2.5.4" release="1.0" summary="Pure Storage FlexDriver Operator" description="Pure Service Orchestrator FlexDriver Operator"
COPY helm-charts/ ${HOME}/helm-charts/
COPY watches.yaml ${HOME}/watches.yaml
COPY licenses /licenses
