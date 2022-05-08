FROM golang:1.18.1 as builder
WORKDIR /hwsw/tekton-workshop/
COPY picalc.go picalc_test.go .

RUN CGO_ENABLED=0 GOOS=linux go test
RUN CGO_ENABLED=0 GOOS=linux go build -v -o picalc

FROM alpine
COPY --from=builder /hwsw/tekton-workshop/picalc /picalc

ENV PORT 8080
CMD ["/picalc"]
