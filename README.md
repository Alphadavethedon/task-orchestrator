# Distributed Task Orchestrator

![Go Version](https://img.shields.io/badge/Go-1.21+-blue)
![Redis](https://img.shields.io/badge/Redis-7.0+-red)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.25+-326CE5)
[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

A scalable, distributed task queue system built with Redis, Go, and Kubernetes designed for fault-tolerant workflow orchestration across multiple workers.

## Features

- **Distributed Task Queue**: Horizontally scalable task processing
- **Fault Tolerance**: Automatic retries and failure handling
- **Workflow Orchestration**: Complex task dependencies and workflows
- **Priority Queues**: Support for multiple priority levels
- **Scheduled Tasks**: Future and recurring task execution
- **Observability**: Built-in metrics, logging, and tracing
- **Kubernetes Native**: Designed to run seamlessly in Kubernetes clusters
- **Redis Backed**: Leverages Redis for persistence and pub/sub

## Architecture

![System Architecture](docs/architecture.png)

The system consists of three main components:
1. **API Server**: Handles task submission and status queries
2. **Workers**: Process tasks from the queue
3. **Scheduler**: Manages scheduled and recurring tasks

All components are stateless and can be scaled independently.

## Quick Start

### Prerequisites

- Go 1.21+
- Redis 7.0+
- Kubernetes cluster (for production deployment)
- Helm (for Kubernetes deployment)

### Local Development

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/distributed-task-orchestrator.git
   cd distributed-task-orchestrator
   ```

2. Start Redis:
   ```bash
   docker run -p 6379:6379 redis:7-alpine
   ```

3. Build and run:
   ```bash
   make build
   ./distributed-task-orchestrator --config=config/local.yaml
   ```

### Kubernetes Deployment

1. Install with Helm:
   ```bash
   helm install task-orchestrator ./deploy/helm \
     --set redis.enabled=true \
     --set worker.replicas=3
   ```

2. Access the API:
   ```bash
   kubectl port-forward svc/task-orchestrator-api 8080:80
   ```

## Configuration

Configuration is managed via YAML files or environment variables:

```yaml
redis:
  addr: "redis:6379"
  password: ""
  db: 0

server:
  port: 8080
  graceful_shutdown_timeout: 30s

worker:
  concurrency: 10
  poll_interval: 1s
  retry_policy:
    initial_delay: 1s
    max_delay: 1m
    max_attempts: 3

metrics:
  enabled: true
  port: 9090
```

Environment variables override the YAML config using the `TASKORCH_` prefix (e.g., `TASKORCH_REDIS_ADDR`).

## API Usage

### Submit a Task

```bash
curl -X POST http://localhost:8080/v1/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "type": "process_image",
    "payload": {"image_url": "https://example.com/image.jpg"},
    "priority": "high",
    "retry_policy": {
      "max_attempts": 3,
      "delay": "5s"
    }
  }'
```

### Get Task Status

```bash
curl http://localhost:8080/v1/tasks/{task_id}
```

### Create a Workflow

```bash
curl -X POST http://localhost:8080/v1/workflows \
  -H "Content-Type: application/json" \
  -d '{
    "name": "image_processing_pipeline",
    "steps": [
      {
        "name": "download",
        "task_type": "download_image",
        "dependencies": []
      },
      {
        "name": "resize",
        "task_type": "resize_image",
        "dependencies": ["download"]
      },
      {
        "name": "upload",
        "task_type": "upload_image",
        "dependencies": ["resize"]
      }
    ]
  }'
```

## Monitoring

The system exposes Prometheus metrics at `/metrics` and supports OpenTelemetry tracing. Pre-configured Grafana dashboards are available in the `deploy/monitoring` directory.

## Development

### Building

```bash
make build
```

### Testing

```bash
make test
```

### Running with Docker Compose

```bash
docker-compose up --build
```

### Code Generation

The project uses Go generate for protobufs and mocks:

```bash
go generate ./...
```

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -am 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a pull request

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## Roadmap

- [ ] Add gRPC API support
- [ ] Implement task batching
- [ ] Add database persistence for audit logging
- [ ] Support workflow versioning
- [ ] Add web UI for monitoring and management

## Support

For support, please open an issue on GitHub or join our [Discord channel](#).

---

Built with ❤️ by DON DAVE
