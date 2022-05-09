FROM golang:1.18.1 as builder
WORKDIR /hwsw/tekton-workshop/

COPY picalc.go picalc_test.go ./

RUN GO111MODULE=auto CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go test && \
    GO111MODULE=auto CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -v -o picalc

FROM alpine:3.15
COPY --from=builder /hwsw/tekton-workshop/picalc /picalc

ENV PORT 8080
CMD ["/picalc"]
