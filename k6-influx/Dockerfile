# From https://github.com/grafana/xk6-output-influxdb/blob/5411750fc094bf48198b6b1e15a528732a1d1e20/Dockerfile
# Licensed under Apache 2.0 License. See https://github.com/grafana/xk6-output-influxdb/blob/5411750fc094bf48198b6b1e15a528732a1d1e20/LICENSE

FROM golang:1.20-alpine3.17 as builder
WORKDIR $GOPATH/src/go.k6.io/k6
RUN apk --no-cache add git
RUN go install go.k6.io/xk6/cmd/xk6@v0.9.2
RUN xk6 build --with github.com/grafana/xk6-output-influxdb@v0.4.1 --output /tmp/k6

FROM alpine:3.17
RUN apk add --no-cache ca-certificates && \
    adduser -D -u 12345 -g 12345 k6
COPY --from=builder /tmp/k6 /usr/bin/k6

USER 12345
WORKDIR /home/k6
ENTRYPOINT ["k6"]
