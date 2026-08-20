# rift

A CLI for tailing and filtering AWS CloudWatch and SQS logs, without
squinting at raw JSON blobs in the AWS console. Point it at a log group
or queue, optionally live-tail it, and grep with a pattern flag instead
of piping through `grep` yourself and losing structure.

## Requirements

- Go 1.18+
- AWS credentials configured (uses your `~/.aws` profile)

## Build

No published releases yet, build from source:

```sh
git clone https://github.com/go-batteries/rift.git
cd rift
go build -o rift .
```

`release.sh` cross-compiles for darwin/amd64, darwin/arm64, and
linux/amd64 into `bin/` if you need binaries for other machines.

## Usage

### CloudWatch

```sh
rift cloudwatch \
  -region us-west-2 \
  -profile default \
  -group /my/log-group \
  -stream my-log-stream \
  -proto proto/cloudwatch/log.proto \
  -live \
  -grep "ERROR"
```

CloudWatch log parsing expects a proto definition for the log body. A
sample is at `proto/cloudwatch/log.proto`, covering the fields rift
looks for:

- `time`: request timestamp
- `request_id`: a trace ID per request
- `msg`: the actual log message
- `level`: log level

### SQS

```sh
rift sqs \
  -region us-west-2 \
  -profile default \
  -queue my-queue-name \
  -live \
  -grep "checkout"
```

SQS messages are currently displayed as-is from the message body; proto
parsing for SQS is on the TODO list below.

### Flags (both subcommands)

| flag | meaning |
|---|---|
| `-region` | AWS region (default `us-west-2`) |
| `-profile` | AWS profile (default `default`) |
| `-proto` | proto file to parse the log/message body |
| `-json` | print output as JSON |
| `-live` | live-tail instead of a one-shot fetch |
| `-grep` | filter by pattern |

## Completeness

CloudWatch and SQS streaming, formatting, and `-grep` filtering are
real and wired end to end, not stubs. What's genuinely incomplete
(verified by reading the source, not guessing):

- `-only` is defined as a flag on both subcommands but never read
  anywhere past the flag parser. It's currently a silent no-op, left
  out of the usage examples above until it's implemented.
- SQS message bodies are displayed as-is; proto-based structured
  parsing (like CloudWatch already has) doesn't exist for SQS yet.
- No published releases, build from source (see above).

## TODO

- [ ] implement `-only` or remove it
- [ ] proto parsing for SQS message bodies
- [ ] support more backends beyond CloudWatch and SQS
- [ ] published releases / binary downloads
