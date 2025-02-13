Here's a `docker-compose.yml` example that sets up an `httpd:2.4` container with CPU and memory limits:

```yaml
version: '3.8'

services:
  web:
    image: httpd:2.4
    container_name: httpd_server
    ports:
      - "8080:80"
    deploy:
      resources:
        limits:
          cpus: "0.5"      # Limits the container to half a CPU
          memory: "512M"    # Limits memory to 512 MB
        reservations:
          cpus: "0.2"       # Reserves 20% of a CPU
          memory: "256M"    # Reserves 256 MB of memory
    restart: always
```

This configuration does the following:

- **Image**: Uses the `httpd:2.4` image for an Apache HTTP server.
- **Ports**: Exposes port 80 on the container to port 8080 on the host.
- **Resources**: Sets limits on CPU and memory to manage the resources the container can consume.
- **Restart Policy**: Ensures the container restarts automatically if it stops.



To verify the CPU and memory limits applied to the running container, you can use the following methods:

### 1. Check Resource Limits with `docker stats`

Run `docker stats` to view the live resource usage and limits:

```bash
docker stats httpd_server
```

This command shows real-time CPU and memory usage for the `httpd_server` container, allowing you to monitor whether it adheres to the set limits. Look at the `CPU %` and `MEM USAGE / LIMIT` columns for verification.

### 2. Inspect Resource Limits with `docker inspect`

To view the set CPU and memory limits, run:

```bash
docker inspect httpd_server --format='{{json .HostConfig.Resources}}' | jq
```

You should see an output similar to this, which confirms the set limits:

```json
{
  "NanoCPUs": 500000000,     // Equivalent to 0.5 CPUs
  "Memory": 536870912        // Equivalent to 512 MB
}
```

This command allows you to verify the exact CPU and memory limits configured in `docker-compose.yml`. The values will be shown in nanoseconds for CPU (`NanoCPUs`) and bytes for memory (`Memory`).