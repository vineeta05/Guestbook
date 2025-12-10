FROM golang:1.24-alpine AS builder
WORKDIR /build
COPY go.mod main.go ./
RUN go mod download
RUN go mod tidy
RUN CGO_ENABLED=0 GOOS=linux go build -o guestbook-api .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /app
COPY --from=builder /build/guestbook-api .
EXPOSE 8080
CMD ["./guestbook-api"]
