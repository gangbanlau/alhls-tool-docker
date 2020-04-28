FROM golang:alpine as build

RUN apk add --no-cache git

RUN mkdir /app
COPY . /app/
WORKDIR /app
RUN go get github.com/fsnotify/fsnotify
RUN go build -o main .

FROM alpine

COPY --from=build /app/main /app/main
ENTRYPOINT  ["/app/main", "-dir", "/var/www/html", "-certdir", "/data"]
