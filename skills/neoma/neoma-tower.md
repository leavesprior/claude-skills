# Neoma Tower Management

Check and manage the Neoma Tower node (192.168.50.238) in the mesh network.

## Available Methods

### Status Check
Run mesh status to see all nodes:
```bash
~/.local/bin/neoma-mesh status
```

### Tower Health Check
Get detailed tower node information:
```bash
~/.local/bin/neoma-tower-check
```

### Quick Connectivity Test
```bash
ping -c 1 192.168.50.238 && ssh granny@neoma-tower "hostname && uptime"
```

### Ollama Status on Tower
```bash
curl -s http://192.168.50.238:11434/api/tags | jq '.models[].name'
```

### Run Inference on Tower
```bash
curl -s http://192.168.50.238:11434/api/generate -d '{"model":"llama3.2","prompt":"$ARGUMENTS","stream":true}'
```

## Node Information
- **Hostname:** neoma-tower
- **IP:** 192.168.50.238
- **Role:** Worker node
- **Capabilities:** ollama, compute, batch
- **Models:** llama3, llama3.2

## Instructions
When this skill is invoked, run both `neoma-mesh status` and `neoma-tower-check` to provide a complete overview of the tower node status. If the user provides arguments, use them as a prompt for tower inference.
