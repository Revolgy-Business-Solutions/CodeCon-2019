FROM golang:alpine as builder

RUN apk add --no-cache git mercurial \
    && go get github.com/go-sql-driver/mysql \
    && go get github.com/gorilla/mux

WORKDIR /app

ADD . .

RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o goapp .

#---

FROM alpine

WORKDIR /app

COPY --from=builder /app/goapp .

EXPOSE 8000

CMD ["./goapp"]
