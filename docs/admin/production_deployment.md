# Production deployment

## Compress responses

If transmitting large payloads in response to requests sent to the context broker, it is possible to enable Gzip compression of responses.

For this, the following property has to be passed (and set to `true`) as an environment variable to the corresponding Docker container: `SERVER_COMPRESSION_ENABLED`.
