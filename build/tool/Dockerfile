FROM alpine:3.12

# Its Based on https://github.com/bskim45/docker-helm-kubectl-jq

# Metadata
LABEL org.label-schema.vcs-ref="" \
    org.label-schema.name="helm-kubectl-oc-wrap-curl-git-jq-yq-kustomize" \
    org.label-schema.url="https://quay.io/repository/soharaki/helm-kubectl-oc-wrap-curl-git-jq-yq-kustomize" \
    org.label-schema.vcs-url="" \
    org.label-schema.build-date="2020/08/27"

# Note: Latest version of kubectl may be found at:
# https://github.com/kubernetes/kubernetes/releases
ENV KUBE_LATEST_VERSION="v1.18.7"
# Note: Latest version of helm may be found at:
# https://github.com/kubernetes/helm/releases
ENV HELM_VERSION="v3.3.0"
# Note: Latest version of helm may be found at:
# https://github.com/mikefarah/yq
ENV YQ_VERSION="3.3.2"
# Note: Latest version of helm may be found at:
# https://github.com/mikefarah/yq
ENV KUSTOMIZE_VERSION="3.8.1"

RUN apk add --no-cache ca-certificates bash git openssh curl jq \
     && wget -q https://storage.googleapis.com/kubernetes-release/release/${KUBE_LATEST_VERSION}/bin/linux/amd64/kubectl -O /usr/local/bin/kubectl \
          && chmod +x /usr/local/bin/kubectl \
          && wget -q https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz -O - | tar -xzO linux-amd64/helm > /usr/local/bin/helm \    
          && chmod +x /usr/local/bin/helm \
          && wget -O /usr/local/bin/yq https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_amd64 \
          && chmod +x /usr/local/bin/yq \
          && wget -q https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv${KUSTOMIZE_VERSION}/kustomize_v${KUSTOMIZE_VERSION}_linux_amd64.tar.gz -O - | tar -xzO kustomize > /usr/local/bin/kustomize \
          && chmod +x /usr/local/bin/kustomize 

WORKDIR /workspace/source

CMD bash