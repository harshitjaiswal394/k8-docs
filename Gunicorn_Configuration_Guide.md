
# Gunicorn Configuration Guide

This guide explains common Gunicorn settings with examples, optimized for various real-world scenarios.

## Basic Settings

### `bind`
- **Purpose**: Specifies the network interface and port for the application.
- **Example**: 
  ```python
  bind = '0.0.0.0:8000'  # Listen on all network interfaces on port 8000
  ```
- **Usage**: Use `0.0.0.0` in production to make the application accessible externally, or `127.0.0.1` for local-only access (useful for development).

### `backlog`
- **Purpose**: Defines the maximum number of pending connections.
- **Example**:
  ```python
  backlog = 4096
  ```
- **Usage**: Increase if expecting high traffic. A setting like `backlog = 4096` helps prevent connection rejection during traffic spikes.

## Worker Settings

### `workers`
- **Purpose**: Sets the number of worker processes to handle requests.
- **Example**:
  ```python
  workers = 4
  ```
- **Usage**: Use `(2 * CPU cores) + 1` for basic applications. Increase workers for more concurrent requests, especially for CPU-bound tasks.

### `worker_class`
- **Purpose**: Determines the type of worker for request handling.
- **Example**:
  ```python
  worker_class = 'gevent'  # Asynchronous workers, ideal for high I/O
  ```
- **Usage**: Use `'sync'` for CPU-bound applications, `'gevent'` or `'eventlet'` for high concurrency (e.g., chat applications or streaming).

### `threads`
- **Purpose**: Sets the number of threads per worker.
- **Example**:
  ```python
  threads = 4
  ```
- **Usage**: Useful for handling multiple requests without adding more workers. Suitable for I/O-bound tasks, such as APIs with many external requests.

### `worker_connections`
- **Purpose**: Limits the maximum number of connections for async workers.
- **Example**:
  ```python
  worker_connections = 2000
  ```
- **Usage**: Increase in environments with high concurrency, especially with async workers for handling thousands of simultaneous connections.

### `max_requests` and `max_requests_jitter`
- **Purpose**: Limits requests per worker to manage memory leaks and reloads workers periodically.
- **Example**:
  ```python
  max_requests = 1000
  max_requests_jitter = 50
  ```
- **Usage**: Restart workers after handling a certain number of requests, ensuring memory stability over time.

## Timeout Settings

### `timeout`
- **Purpose**: Sets the maximum duration for handling a single request.
- **Example**:
  ```python
  timeout = 120  # 2 minutes
  ```
- **Usage**: Set lower values for applications with quick requests. Increase for long-running requests to avoid timeout errors.

### `graceful_timeout`
- **Purpose**: Allows workers to finish ongoing requests before stopping.
- **Example**:
  ```python
  graceful_timeout = 60
  ```
- **Usage**: Use slightly lower than `timeout` to give workers time to complete before restarting, ideal for zero-downtime deployments.

### `keepalive`
- **Purpose**: Defines how long to keep a connection open for a client.
- **Example**:
  ```python
  keepalive = 5  # 5 seconds
  ```
- **Usage**: Useful for reusing client connections, especially for APIs receiving sequential requests.

## Request Limiting

### `limit_request_line`
- **Purpose**: Sets the max length of a request line.
- **Example**:
  ```python
  limit_request_line = 8190
  ```
- **Usage**: Controls excessively long URLs. Increase only if necessary to handle large URL strings.

### `limit_request_fields` and `limit_request_field_size`
- **Purpose**: Limits the number and size of HTTP headers.
- **Example**:
  ```python
  limit_request_fields = 100
  limit_request_field_size = 8190
  ```
- **Usage**: Adjust to match your application’s header requirements while preventing abuse (e.g., large cookie headers).

## Reloading (Development Mode)

### `reload`
- **Purpose**: Enables automatic reloading of code changes.
- **Example**:
  ```python
  reload = True
  ```
- **Usage**: Use in development to apply code changes without restarting the server.

## Logging

### `loglevel`
- **Purpose**: Sets the verbosity of log output.
- **Example**:
  ```python
  loglevel = 'info'
  ```
- **Usage**: Use `'debug'` for development; `'info'` or `'warning'` for production.

### `accesslog` and `errorlog`
- **Purpose**: Specify locations for logging access and error messages.
- **Example**:
  ```python
  accesslog = '/var/log/gunicorn/access.log'
  errorlog = '/var/log/gunicorn/error.log'
  ```
- **Usage**: Logs requests and errors. For Docker setups, set to `'-'` to log to stderr for console output.

## SSL and Network Security

### `secure_scheme_headers`
- **Purpose**: Defines headers for HTTPS detection behind proxies.
- **Example**:
  ```python
  secure_scheme_headers = {'X-Forwarded-Proto': 'https'}
  ```
- **Usage**: Ensures secure header handling when running behind a reverse proxy like Nginx.

### `proxy_protocol` and `proxy_allow_ips`
- **Purpose**: Enables and restricts proxy protocol usage.
- **Example**:
  ```python
  proxy_protocol = True
  proxy_allow_ips = ['127.0.0.1']
  ```
- **Usage**: Restrict access to trusted proxies to prevent unauthorized access via proxy.

### `ssl_context`, `keyfile`, and `certfile`
- **Purpose**: Enable SSL/TLS if serving HTTPS directly from Gunicorn.
- **Example**:
  ```python
  keyfile = '/path/to/server.key'
  certfile = '/path/to/server.crt'
  ```
- **Usage**: Generally, SSL is handled by reverse proxies, but this config is useful for direct HTTPS serving in smaller setups.

## Lifecycle Hooks (Advanced)

### `on_starting` and `when_ready`
- **Purpose**: Execute custom code on server events.
- **Example**:
  ```python
  def on_starting(server):
      print("Server is starting up")
  ```
- **Usage**: Useful for initializing logging, metrics, or sending notifications at startup.

### `worker_exit`
- **Purpose**: Execute actions when a worker exits.
- **Example**:
  ```python
  def worker_exit(server, worker):
      print("Worker exited")
  ```
- **Usage**: Helps with cleanup operations or logging worker restarts.

## HTTP Header and Method Handling

### `permit_unconventional_http_method`
- **Purpose**: Allows or restricts unconventional HTTP methods.
- **Example**:
  ```python
  permit_unconventional_http_method = False
  ```
- **Usage**: Set to `False` unless your application requires custom HTTP methods.

### `header_map`
- **Purpose**: Controls header handling.
- **Example**:
  ```python
  header_map = 'drop'
  ```
- **Usage**: Use `'drop'` to discard specific headers or `'pass'` to forward all headers. This helps with header security when interacting with proxies.

---

