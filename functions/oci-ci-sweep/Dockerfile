FROM golang:1.26-alpine@sha256:ce864e7223ac17b1775e6fd0b4c0db580c2eb50e7953a427916379e4b92a1628 AS build
WORKDIR /go/src/func
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o func .

# Plain alpine, not an Fn/OCI-specific runtime image: the compiled binary is
# static (CGO_ENABLED=0) and fdk-go embeds the OCI Functions invocation
# protocol directly, so no Go toolchain is needed at runtime. fnproject/go
# (the officially documented Fn Go runtime image) tops out at Go 1.24, which
# would force downgrading go.mod; this trades that official-but-stale image
# for staying on a current Go release.
FROM alpine@sha256:28bd5fe8b56d1bd048e5babf5b10710ebe0bae67db86916198a6eec434943f8b
RUN adduser -D -u 10001 func
COPY --from=build /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
WORKDIR /function
COPY --from=build /go/src/func/func /function/func
USER func
ENTRYPOINT ["./func"]
