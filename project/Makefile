FRONT_END_BINARY=frontApp
BROKER_BINARY=brokerApp
AUTH_BINARY=authApp
LOGGER_BINARY=loggerServiceApp
MAIL_BINARY=mailerApp
LISTENER_BINARY=listenerApp
FRONT_BINARY=frontEndApp

# Source file dependencies
LOGGER_PROTO_FILES=../logger-service/logs/logs.proto
BROKER_PROTO_FILES=../broker-service/logs/logs.proto
LOGGER_GO_FILES=$(wildcard ../logger-service/cmd/api/*.go ../logger-service/data/*.go ../logger-service/*.go)
BROKER_GO_FILES=$(wildcard ../broker-service/cmd/api/*.go ../broker-service/event/*.go)
AUTH_GO_FILES=$(wildcard ../authentication-service/cmd/api/*.go ../authentication-service/data/*.go)
MAIL_GO_FILES=$(wildcard ../mail-service/cmd/api/*.go)
LISTENER_GO_FILES=$(wildcard ../listener-service/*.go ../listener-service/event/*.go)
FRONT_GO_FILES=$(wildcard ../front-end/cmd/web/*.go)

## up: starts all containers in the background without forcing build
up:
	@echo "Starting Docker images..."
	docker compose up -d
	@echo "Docker images started!"

## up_build: stops docker-compose (if running), builds all projects and starts docker compose
up_build: build_all
	@echo "Stopping docker images (if running...)"
	docker compose down
	@echo "Building (when required) and starting docker images..."
	docker compose up --build -d
	@echo "Docker images built and started!"

## build_all: builds all services in correct dependency order
build_all: protoc_generate build_logger build_auth build_mail build_listener build_broker

## down: stop docker compose
down:
	@echo "Stopping docker compose..."
	docker compose down
	@echo "Done!"

## protoc_generate: generates protobuf files
protoc_generate:
	@echo "Generating protobuf files..."
	@if command -v protoc > /dev/null; then \
		cd ../logger-service && protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative logs/logs.proto; \
		cd ../broker-service && protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative logs/logs.proto; \
		echo "Protobuf files generated!"; \
	else \
		echo "Warning: protoc not found. Install Protocol Buffers compiler to regenerate .proto files"; \
	fi

## clean_proto: clean generated protobuf files
clean_proto:
	@echo "Cleaning generated protobuf files..."
	@rm -f ../logger-service/logs/*.pb.go
	@rm -f ../broker-service/logs/*.pb.go
	@echo "Done!"

## build_front_linux: builds the front end binary as a linux executable
build_front_linux: $(FRONT_GO_FILES)
	@echo "Building front end linux binary..."
	cd ../front-end && env GOOS=linux CGO_ENABLED=0 go build -o ${FRONT_BINARY} ./cmd/web
	@echo "Done!"

## build_broker: builds the broker binary as a linux executable (depends on protobuf files)
build_broker: protoc_generate $(BROKER_GO_FILES) ../broker-service/logs/logs.pb.go ../broker-service/logs/logs_grpc.pb.go
	@echo "Building broker binary..."
	cd ../broker-service && env GOOS=linux CGO_ENABLED=0 go build -o ${BROKER_BINARY} ./cmd/api
	@echo "Done!"

## build_listener: builds the listener binary as a linux executable
build_listener: $(LISTENER_GO_FILES)
	@echo "Building listener binary..."
	cd ../listener-service && env GOOS=linux CGO_ENABLED=0 go build -o ${LISTENER_BINARY} .
	@echo "Done!"

## build_logger: builds the logger binary as a linux executable (depends on protobuf files)
build_logger: protoc_generate $(LOGGER_GO_FILES) ../logger-service/logs/logs.pb.go ../logger-service/logs/logs_grpc.pb.go
	@echo "Building logger binary..."
	cd ../logger-service && env GOOS=linux CGO_ENABLED=0 go build -o ${LOGGER_BINARY} ./cmd/api
	@echo "Done!"

## build_auth: builds the auth binary as a linux executable
build_auth: $(AUTH_GO_FILES)
	@echo "Building auth binary..."
	cd ../authentication-service && env GOOS=linux CGO_ENABLED=0 go build -o ${AUTH_BINARY} ./cmd/api
	@echo "Done!"

## build_mail: builds the mail binary as a linux executable
build_mail: $(MAIL_GO_FILES)
	@echo "Building mail binary..."
	cd ../mail-service && env GOOS=linux CGO_ENABLED=0 go build -o ${MAIL_BINARY} ./cmd/api
	@echo "Done!"

## build_front: builds the frone end binary
build_front:
	@echo "Building front end binary..."
	cd ../front-end && env CGO_ENABLED=0 go build -o ${FRONT_END_BINARY} ./cmd/web
	@echo "Done!"

## start: starts the front end
start: build_front
	@echo "Starting front end"
	cd ../front-end && ./${FRONT_END_BINARY} &

## stop: stop the front end
stop:
	@echo "Stopping front end..."
	@-pkill -SIGTERM -f "./${FRONT_END_BINARY}"
	@echo "Stopped front end!"

## clean: clean all built binaries
clean:
	@echo "Cleaning built binaries..."
	@rm -f ../front-end/${FRONT_END_BINARY}
	@rm -f ../front-end/${FRONT_BINARY}
	@rm -f ../broker-service/${BROKER_BINARY}
	@rm -f ../authentication-service/${AUTH_BINARY}
	@rm -f ../logger-service/${LOGGER_BINARY}
	@rm -f ../mail-service/${MAIL_BINARY}
	@rm -f ../listener-service/${LISTENER_BINARY}
	@echo "Done!"

## clean_all: clean binaries and protobuf files
clean_all: clean clean_proto
	@echo "All clean!"

## check_deps: check if required tools are installed
check_deps:
	@echo "Checking dependencies..."
	@which go >/dev/null || (echo "ERROR: Go is not installed" && exit 1)
	@which docker >/dev/null || (echo "ERROR: Docker is not installed" && exit 1)
	@docker compose version >/dev/null 2>&1 || (echo "ERROR: Docker Compose is not available" && exit 1)
	@which protoc >/dev/null || echo "WARNING: protoc not found. Install Protocol Buffers compiler to regenerate .proto files"
	@echo "Dependencies check completed!"

## test: run tests for all services
test:
	@echo "Running tests..."
	@cd ../authentication-service && go test -v ./...
	@cd ../broker-service && go test -v ./...
	@cd ../logger-service && go test -v ./...
	@cd ../mail-service && go test -v ./...
	@cd ../listener-service && go test -v ./...
	@cd ../front-end && go test -v ./...
	@echo "Tests completed!"

## mod_tidy: run go mod tidy for all services
mod_tidy:
	@echo "Running go mod tidy for all services..."
	@cd ../authentication-service && go mod tidy
	@cd ../broker-service && go mod tidy
	@cd ../logger-service && go mod tidy
	@cd ../mail-service && go mod tidy
	@cd ../listener-service && go mod tidy
	@cd ../front-end && go mod tidy
	@echo "Go mod tidy completed!"

## help: show this help message
help:
	@echo "Available targets:"
	@echo "  up              - Start all containers in the background"
	@echo "  up_build        - Build all services and start containers"
	@echo "  down            - Stop all containers"
	@echo "  build_all       - Build all services"
	@echo "  build_broker    - Build broker service"
	@echo "  build_auth      - Build authentication service"
	@echo "  build_logger    - Build logger service"
	@echo "  build_mail      - Build mail service"
	@echo "  build_listener  - Build listener service"
	@echo "  build_front     - Build frontend service"
	@echo "  build_front_linux - Build frontend for Linux"
	@echo "  protoc_generate - Generate protobuf files"
	@echo "  clean_proto     - Clean generated protobuf files"
	@echo "  clean           - Clean all built binaries"
	@echo "  clean_all       - Clean binaries and protobuf files"
	@echo "  check_deps      - Check if required tools are installed"
	@echo "  test            - Run tests for all services"
	@echo "  mod_tidy        - Run go mod tidy for all services"
	@echo "  start           - Start frontend service"
	@echo "  stop            - Stop frontend service"
	@echo "  help            - Show this help message"

.PHONY: up up_build down build_all build_broker build_auth build_logger build_mail build_listener build_front build_front_linux protoc_generate clean_proto clean clean_all check_deps test mod_tidy start stop help