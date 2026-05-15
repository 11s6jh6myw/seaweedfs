# Multi-stage build for SeaweedFS
# Stage 1: Build the binary
FROM golang:1.21-alpine AS builder

RUN apk add --no-cache git make gcc musl-dev

WORKDIR /go/src/github.com/seaweedfs/seaweedfs

# Copy go mod files first for better layer caching
COPY go.mod go.sum ./
RUN go mod download

# Copy the rest of the source code
COPY . .

# Build the weed binary
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags "-s -w -X github.com/seaweedfs/seaweedfs/weed/util.VERSION=$(git describe --tags --always --dirty)" \
    -o /usr/local/bin/weed \
    ./weed

# Stage 2: Create minimal runtime image
FROM alpine:3.18

# Install runtime dependencies
RUN apk add --no-cache \
    ca-certificates \
    tzdata \
    curl \
    && rm -rf /var/cache/apk/*

# Create non-root user for running seaweedfs
RUN addgroup -S seaweedfs && adduser -S seaweedfs -G seaweedfs

# Copy the binary from builder stage
COPY --from=builder /usr/local/bin/weed /usr/local/bin/weed

# Create data directories
RUN mkdir -p /data/master /data/volume /data/filer \
    && chown -R seaweedfs:seaweedfs /data

# Switch to non-root user
USER seaweedfs

# Expose common SeaweedFS ports
# Master server HTTP
EXPOSE 9333
# Volume server HTTP
EXPOSE 8080
# Filer HTTP
EXPOSE 8888
# S3 API
EXPOSE 8333
# gRPC ports
EXPOSE 19333 18080 18888

VOLUME ["/data"]

WORKDIR /data

# Default command shows help; override in docker-compose or k8s
ENTRYPOINT ["/usr/local/bin/weed"]
CMD ["--help"]
