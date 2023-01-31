---
title: otelcol.receiver.opencensus
---

# otelcol.receiver.opencensus

`otelcol.receiver.opencensus` accepts telemetry data via gRPC or HTTP 
using the [OpenCensus](https://opencensus.io/) format and
forwards it to other `otelcol.*` components.

> **NOTE**: `otelcol.receiver.opencensus` is a wrapper over the upstream
> OpenTelemetry Collector `opencensus` receiver from the `otelcol-contrib`
> distribution. Bug reports or feature requests will be redirected to the
> upstream repository, if necessary.

Multiple `otelcol.receiver.opencensus` components can be specified by giving them
different labels.

## Usage

```river
otelcol.receiver.opencensus "LABEL" {
  output {
    metrics = [...]
    logs    = [...]
    traces  = [...]
  }
}
```

## Arguments

`otelcol.receiver.opencensus` supports the following arguments:


Name | Type | Description | Default | Required
---- | ---- | ----------- | ------- | --------
`cors_allowed_origins`     | `list(string)` | A list of allowed Cross-Origin Resource Sharing (CORS) origins. |  | no

`cors_allowed_origins` are the allowed [CORS](https://github.com/rs/cors) origins for HTTP/JSON requests.
An empty list means that CORS is not enabled at all. A wildcard (*) can be
used to match any origin or one or more characters of an origin.

## Blocks

The following blocks are supported inside the definition of
`otelcol.receiver.opencensus`:

Hierarchy | Block | Description | Required
--------- | ----- | ----------- | --------
grpc | [grpc][] | Configures the gRPC/HTTP server to receive telemetry data. | no
grpc > tls | [tls][] | Configures TLS for the gRPC server. | no
grpc > keepalive | [keepalive][] | Configures keepalive settings for the configured server. | no
grpc > keepalive > server_parameters | [server_parameters][] | Server parameters used to configure keepalive settings. | no
grpc > keepalive > enforcement_policy | [enforcement_policy][] | Enforcement policy for keepalive settings. | no
output | [output][] | Configures where to send received telemetry data. | yes

The `>` symbol indicates deeper levels of nesting. For example, `grpc > tls`
refers to a `tls` block defined inside a `grpc` block.

[grpc]: #grpc-block
[tls]: #tls-block
[keepalive]: #keepalive-block
[server_parameters]: #server_parameters-block
[enforcement_policy]: #enforcement_policy-block
[output]: #output-block

### grpc block

The `grpc` block configures the gRPC/HTTP server used by the component. If the
`grpc` block isn't provided, a server using default parameters is started.

The following arguments are supported:

Name | Type | Description | Default | Required
---- | ---- | ----------- | ------- | --------
`endpoint` | `string` | `host:port` to listen for traffic on. | `"0.0.0.0:4317"` | no
`transport` | `string` | Transport to use for the gRPC server. | `"tcp"` | no
`max_recv_msg_size` | `string` | Maximum size of messages the server will accept. 0 disables a limit. | | no
`max_concurrent_streams` | `number` | Limit the number of concurrent streaming RPC calls. | | no
`read_buffer_size` | `string` | Size of the read buffer the gRPC server will use for reading from clients. | `"512KiB"` | no
`write_buffer_size` | `string` | Size of the write buffer the gRPC server will use for writing to clients. | | no
`include_metadata` | `boolean` | Propagate incoming connection metadata to downstream consumers. | | no

In order to use plain HTTP/JSON, specify the `endpoint` attribute. There is no HTTP-specific block.
The HTTP/JSON address is the same as gRPC, as the protocol is recognized and processed accordingly.

To write traces with HTTP/JSON, `POST` to `[address]/v1/trace`. The JSON message format parallels the gRPC protobuf format. For details, refer to its [OpenApi specification](https://github.com/census-instrumentation/opencensus-proto/blob/master/gen-openapi/opencensus/proto/agent/trace/v1/trace_service.swagger.json).

Note that `max_recv_msg_size`, `read_buffer_size` and `write_buffer_size` are formatted in a special way, 
so that the units are included in the string, e.g., "512KiB" or "1024KB".

### tls block

The `tls` block configures TLS settings used for a server. If the `tls` block
isn't provided, TLS won't be used for connections to the server.

The following arguments are supported:

Name | Type | Description | Default | Required
---- | ---- | ----------- | ------- | --------
`ca_file` | `string` | Path to the CA file. | | no
`cert_file` | `string` | Path to the TLS certificate. | | no
`key_file` | `string` | Path to the TLS certificate key. | | no
`min_version` | `string` | Minimum acceptable TLS version for connections. | `"TLS 1.2"` | no
`max_version` | `string` | Maximum acceptable TLS version for connections. | `"TLS 1.3"` | no
`reload_interval` | `duration` | Frequency to reload the certificates. | | no
`client_ca_file` | `string` | Path to the CA file used to authenticate client certificates. | | no

### keepalive block

The `keepalive` block configures keepalive settings for connections to a gRPC
server.

`keepalive` doesn't support any arguments and is configured fully through inner
blocks.

### server_parameters block

The `server_parameters` block controls keepalive and maximum age settings for gRPC
servers.

The following arguments are supported:

Name | Type | Description | Default | Required
---- | ---- | ----------- | ------- | --------
`max_connection_idle` | `duration` | Maximum age for idle connections. | `"infinity"` | no
`max_connection_age` | `duration` | Maximum age for non-idle connections. | `"infinity"` | no
`max_connection_age_grace` | `duration` | Time to wait before forcibly closing connections. | `"infinity"` | no
`time` | `duration` | How often to ping inactive clients to check for liveness. | `"2h"` | no
`timeout` | `duration` | Time to wait before closing inactive clients that do not respond to liveness checks. | `"20s"` | no

### enforcement_policy block

The `enforcement_policy` block configures the keepalive enforcement policy for
gRPC servers. The server will close connections from clients that violate the
configured policy.

The following arguments are supported:

Name | Type | Description | Default | Required
---- | ---- | ----------- | ------- | --------
`min_time` | `duration` | Minimum time clients should wait before sending a keepalive ping. | `"5m"` | no
`permit_without_stream` | `boolean` | Allow clients to send keepalive pings when there are no active streams. | `false` | no

### output block

{{< docs/shared lookup="flow/reference/components/output-block.md" source="agent" >}}

## Exported fields

`otelcol.receiver.opencensus` does not export any fields.

## Component health

`otelcol.receiver.opencensus` is only reported as unhealthy if given an invalid
configuration.

## Debug information

`otelcol.receiver.opencensus` does not expose any component-specific debug
information.

## Example

This example forwards received telemetry data through a batch processor before
finally sending it to an OTLP-capable endpoint:

```river
otelcol.receiver.opencensus "default" {
	cors_allowed_origins = ["https://*.test.com", "https://test.com"]

	grpc {
		endpoint  = "0.0.0.0:9090"
		transport = "tcp"

		max_recv_msg_size      = "32KB"
		max_concurrent_streams = "16"
		read_buffer_size       = "1024KB"
		write_buffer_size      = "1024KB"
		include_metadata       = true

		tls {
			cert_file = "test.crt"
			key_file  = "test.key"
		}

		keepalive {
			server_parameters {
				max_connection_idle      = "11s"
				max_connection_age       = "12s"
				max_connection_age_grace = "13s"
				time                     = "30s"
				timeout                  = "5s"
			}

			enforcement_policy {
				min_time              = "10s"
				permit_without_stream = true
			}
		}
	}

	output {
		metrics = [otelcol.processor.batch.default.input]
		logs    = [otelcol.processor.batch.default.input]
		traces  = [otelcol.processor.batch.default.input]
	}
}

otelcol.processor.batch "default" {
	output {
		metrics = [otelcol.exporter.otlp.default.input]
		logs    = [otelcol.exporter.otlp.default.input]
		traces  = [otelcol.exporter.otlp.default.input]
	}
}

otelcol.exporter.otlp "default" {
	client {
		endpoint = env("OTLP_ENDPOINT")
	}
}
```