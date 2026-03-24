ARG GO_VERSION=1.25

# Copy Go from the official image — avoids apt-get entirely
FROM golang:${GO_VERSION} AS go

FROM dockersamples/labspace-workspace-base

COPY --from=go /usr/local/go /usr/local/go

ENV PATH=/usr/local/go/bin:$PATH
ENV GOPATH=/home/coder/go
ENV GOBIN=/home/coder/go/bin
ENV PATH=/home/coder/go/bin:$PATH

USER root

# Install gopls (Go language server for IDE support)
RUN /usr/local/go/bin/go install golang.org/x/tools/gopls@latest && \
    chown -R coder:coder /home/coder/go

USER coder
