# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Build and Development
- `npm install` - Install all dependencies
- `npm run build` - Build TypeScript to JavaScript (outputs to `dist/`)
- `npm run dev` - Run in development mode with ts-node
- `npm start` - Run the built application from `dist/`
- `npm run clean` - Clean the dist directory

### Docker
- Build: `docker build -t hame-relay .`
- Run: `docker run -v $(pwd)/config:/app/config ghcr.io/tomquist/hame-relay:main`

## Architecture Overview

This is an MQTT relay service that bridges Marstek storage systems between local and cloud MQTT brokers. The system supports two operational modes:

### Core Components

1. **MQTT Forwarder** (`src/forwarder.ts`)
   - Main application logic for bidirectional MQTT message forwarding
   - Manages connections to both local and remote (Hame cloud) brokers
   - Handles device-specific topic transformations and routing
   - Supports two modes via `inverse_forwarding` flag:
     - `false`: Device connects to local broker, relay forwards to cloud (Saturn/B2500 only)
     - `true`: Device connects to cloud broker, relay forwards to local (all devices)

2. **Device Management**
   - Supports multiple Marstek device types (HMA-1, HMA-2, HMA-3, etc.)
   - Device identification uses device_id (24-digit), MAC address, and type
   - Automatic device discovery via Hame API (`src/hame_api.ts`)

3. **Broker Configuration** 
   - Remote brokers defined in `brokers.json`
   - Supports multiple cloud broker endpoints with different encryption keys
   - Automatic broker selection based on device firmware version

4. **Topic Encryption** (`src/encryption.ts`)
   - Handles encrypted topic ID generation for newer firmware versions
   - Uses device-specific encryption based on MAC address

5. **Health Monitoring** (`src/health.ts`)
   - HTTP health endpoint on port 8080
   - Docker health checks integrated

### Configuration Structure

The system uses a two-file configuration:
- `config/config.json` - User configuration with local broker and devices
- `brokers.json` - Remote broker definitions with certificates and encryption keys

### Key Design Decisions

- TypeScript with strict typing for reliability
- Pino logger for structured logging with configurable levels
- Separate local and remote MQTT client instances per device
- Certificate-based authentication for cloud brokers
- Health checks for container orchestration