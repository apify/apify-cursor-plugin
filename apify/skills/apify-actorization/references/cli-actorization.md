# CLI-Based Actorization

For languages without an SDK (Go, Rust, Java, etc.), create a wrapper script that uses the Apify CLI.

## Create Wrapper Script

Create `start.sh` in project root:

```bash
#!/bin/bash
set -e

# Get input from Apify key-value store
INPUT=$(apify actor:get-input)

# Parse input values (adjust based on your input schema)
MY_PARAM=$(echo "$INPUT" | jq -r '.myParam // "default"')

# Run your application with the input
./your-application --param "$MY_PARAM"

# If your app writes to a file, push it to key-value store
# apify actor:set-value OUTPUT --contentType application/json < output.json

# Or push structured data to dataset
# apify actor:push-data '{"result": "value"}'
```

## Update Dockerfile

Apify base images run on Alpine Linux, so install system packages with `apk`, not `apt-get`.

```dockerfile
FROM apify/actor-node:20

# Install system packages: bash runs start.sh, jq parses the input
RUN apk add --no-cache bash jq

# Install the Apify CLI from npm
RUN npm install -g apify-cli

# Copy your application
COPY . .

# Build your application if needed
# RUN ./build.sh

# Make start script executable
RUN chmod +x start.sh

# Run the wrapper script
CMD ["./start.sh"]
```

For a complete starting point, see the [cli-start template Dockerfile](https://github.com/apify/actor-templates/blob/master/templates/cli-start/Dockerfile). If your tool ships as a binary on GitHub Releases, pin the release version and verify its checksum before you run it.

## Testing CLI-Based Actors

For CLI-based actors (shell wrapper scripts), you may need to test the underlying application directly with mock input, as `apify run` requires a Node.js or Python entry point.

Test your wrapper script locally:

```bash
# Set up mock input
export INPUT='{"myParam": "test-value"}'

# Run wrapper script
./start.sh
```

## CLI Commands Reference

| Command | Description |
|---------|-------------|
| `apify actor:get-input` | Get input JSON from key-value store |
| `apify actor:set-value KEY` | Store value in key-value store |
| `apify actor:push-data JSON` | Push data to dataset |
| `apify actor:get-value KEY` | Retrieve value from key-value store |
