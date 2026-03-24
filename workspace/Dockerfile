FROM dockersamples/labspace-workspace-base

ARG GO_VERSION=1.23.5
ARG TARGETARCH

USER root

# Install Go for the target architecture
RUN apt-get update -qq && apt-get install -y --no-install-recommends wget ca-certificates && \
    wget -q "https://go.dev/dl/go${GO_VERSION}.linux-${TARGETARCH}.tar.gz" && \
    tar -C /usr/local -xzf "go${GO_VERSION}.linux-${TARGETARCH}.tar.gz" && \
    rm "go${GO_VERSION}.linux-${TARGETARCH}.tar.gz" && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

ENV PATH=/usr/local/go/bin:$PATH
ENV GOPATH=/home/coder/go
ENV GOBIN=/home/coder/go/bin
ENV PATH=/home/coder/go/bin:$PATH

# Install gopls (Go language server for IDE support)
RUN /usr/local/go/bin/go install golang.org/x/tools/gopls@latest && \
    chown -R coder:coder /home/coder/go

USER coder
