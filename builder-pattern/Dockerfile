FROM golang:1.12.1

COPY app /app

WORKDIR /app

RUN go mod download
RUN go install ...

EXPOSE 8080

CMD ["/go/bin/webapp"]