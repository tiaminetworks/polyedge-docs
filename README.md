# PolyEdge User Guide

## Overview

PolyEdge is a real-time ISAC localization system that uses 5G NR Downlink Synchronization Signals to track and locate targets. The system provides real-time tracking capabilities with TAK (Team Awareness Kit) integration for tactical situational awareness.

## System Components

- **Real-time Non-cooperative Target Localization**: Converts 5G NR frames to absolute target positions using ISAC
- **TAK Integration**: Broadcasts target locations to tactical systems
- **5G NR Interface**: Interfaces with 5G networks for signal data collection
- **License Validation**: Secure operation with hardware-bound licensing
- **Multi-session Management**: Concurrent operation of multiple system components

### The Streamer (`./streamer`)

The **streamer** is the core real-time processing engine that bridges 5G NR signal collection with ISAC localization. It continuously processes Downlink Synchronization Signals data and produce absolute coordinates with a combination of Passive Radar based processing and ML.

#### How the Streamer Works:

1. **5G NR Frame Ingestion**: Receives raw downlink 5G NR frames PolyEdge replicates complete L1-PHY stack optimized for ISAC to demodulate and utilize the downlink telemetry as the resource for sensing.
2. **Real-time Processing**: Applies signal processing and normalization
3. **Model Inference**: Uses the trained model to predict absolute target positions
4. **Output Generation**: Streams results to CSV logs, WebSocket clients, and TAK systems

#### Configuration Dependencies:

The streamer requires both configuration files to operate:

**From `nr_params.json`:**
- `band`: Determines signal processing parameters for the specific 5G band
- `frequency`: Sets center frequency for 5G NR frame processing
- `subcarrier_spacing`: Configures numerology for signal analysis
- `number_of_prbs`: Defines resource block processing scope
- `gnb_location`: Used as reference point for coordinate transformations

**From `config.json`:**
- `model_inference.enabled`: Controls whether target positioning is active
- `model_inference.device`: Sets processing device (CPU/GPU)
- `model_inference.reference_point`: **Must match `gnb_location`** for accurate positioning
- `model_inference.gps_boundaries`: Normalization parameters for model input
- `output_directory`: Where processed data and logs are written
- `websocket_port`: Real-time streaming configuration
- `processing.max_queue_size`: 5G NR frame queue management
- `processing.file_ready_timeout`: File processing timing

#### Streamer Operation Flow:

```
          5G NR Frame
               │
               ▼
      ┌────────────────────────┐
      │   Signal Processing    │
      └────────────────────────┘
               │
               ▼
      ┌────────────────────────┐
      │     CAF Generation     │  
      └────────────────────────┘
               │
               ▼
      ┌────────────────────────┐
      │ Ambiguity Match Filter │
      └────────────────────────┘
               │
               ▼
      ┌────────────────────────┐
      │     Model Inference    │
      └────────────────────────┘
               │
               ▼
     Absolute Target Position

```

#### Real-time Outputs:

- **CSV Logging**: Continuous target position tracking log (`tracking_log.csv`)
- **WebSocket Stream**: Real-time coordinate broadcasting (port 8765 default)
- **Console Output**: Processing statistics and system status
- **TAK Integration**: Formatted messages for tactical displays

#### Performance Indicators:

**Normal Operation:**
- Consistent 5G NR frame processing (no queue buildup)
- Target coordinates generated at regular intervals
- Model confidence scores > 0.3 (typical range 0.3-0.8)
- No timeout errors in processing queue

**Potential Issues:**
- Queue buildup indicates processing bottleneck
- Low confidence scores suggest poor RF conditions or model mismatch
- Missing target position output indicates configuration or model issues

#### Streamer Startup Sequence:

1. **License Validation**: Verifies license file before operation
2. **Configuration Loading**: Reads `nr_params.json` and `config.json`
3. **Model Loading**: Loads the trained target positioning model
4. **Processing Pipeline**: Initializes 5G NR frame processing and inference pipeline
5. **Output Setup**: Creates directories and initializes logging
6. **Real-time Operation**: Begins continuous 5G NR frame processing

## Prerequisites

- System must be compiled and all binaries available
- Valid license file (`license.lic`)
- Proper 5G NR hardware setup
- TAK server access (optional, for tactical integration)

---

## Configuration Files

### 1. NR Parameters Configuration (`nr_params.json`)

This file defines the 5G NR network parameters for your deployment.

#### Template:
```json
{
    "operator": "OPERATOR_NAME",
    "band": 66,
    "frequency": 2.15268e9,
    "subcarrier_spacing": 15e3,
    "number_of_prbs": 106,
    "gnb_location": {
        "lat": 38.433531,
        "lon": -121.437309,
        "alt": -4.8
    }
}
```

#### Parameter Descriptions:

**Network Parameters:**
- `operator`: Name of your network operator (e.g., "ATT", "Verizon", "Custom")
- `band`: 5G NR frequency band number (e.g., 66, 71, 78)
- `frequency`: Center frequency in Hz (e.g., 2.15268e9 for 2.15 GHz)
- `subcarrier_spacing`: Subcarrier spacing in Hz (15e3 = 15 kHz, 30e3 = 30 kHz)
- `number_of_prbs`: Number of Physical Resource Blocks

**Location Parameters:**
- `gnb_location`: GPS coordinates of your gNodeB (base station)
  - `lat`: Latitude in decimal degrees
  - `lon`: Longitude in decimal degrees  
  - `alt`: Altitude in meters

#### Common Band Configurations:

**Band 66 (AWS-3):**
```json
{
    "band": 66,
    "frequency": 2.15268e9,
    "subcarrier_spacing": 15e3,
    "number_of_prbs": 106
}
```

**Band 71 (600 MHz):**
```json
{
    "band": 71,
    "frequency": 622850000,
    "subcarrier_spacing": 15e3,
    "number_of_prbs": 106
}
```

**Band 78 (3.5 GHz):**
```json
{
    "band": 78,
    "frequency": 3.68e9,
    "subcarrier_spacing": 30e3,
    "number_of_prbs": 75
}
```

### 2. Main System Configuration (`config.json`)

This file configures the PolyEdge localization system and TAK integration.

#### Template:
```json
{
  "output_directory": "./tmp/output",
  "csv_file_path": "./tmp/output/tracking_log.csv",
  "license_file": "./license.lic",
  "log_level": "INFO",
  "websocket_port": 8765,
  "model_inference": {
      "enabled": true,
      "model_type": "isac_tracking",
      "device": "cpu",
      "gps_boundaries": {
        "x": {
            "min": -41.50738856501624,
            "max": 5.499877579475834,
            "mean": -15.188734690437816
        },
        "y": {
            "min": -108.29829492214316,
            "max": -4.78020849600199,
            "mean": -44.838131795633096
        },
        "z": {
            "min": -5.48,
            "max": 11.620000000000001,
            "mean": 3.6787458100558603
        },
        "alt": {
            "min": 0,
            "max": 100
        }
      },
      "reference_point": {
          "latitude": 38.43575,
          "longitude": -121.43635,
          "altitude": -16.5
      },
      "normalization_type": "zerocenter"
  },
  "simulator_enabled": false,
  "simulator_source": "./tmp/source_data",
  "simulator_destination": "./tmp/csi_data",
  "processing": {
      "max_queue_size": 100,
      "file_ready_timeout": 5.0,
      "check_interval": 0.1
  },
  "tak": {
      "enabled": false,
      "sensor_heading": 0.0,
      "cot_url": "10.39.62.102:8089",
      "client_cert": "/path/to/client-cert.pem",
      "client_key": "/path/to/client-key.pem",
      "ca_file": "/path/to/ca-cert.pem"
  }
}
```

#### Section Descriptions:

**Basic Settings:**
- `output_directory`: Where output files are stored
- `csv_file_path`: Location of GPS tracking log file
- `license_file`: Path to your license file
- `log_level`: Logging verbosity (DEBUG, INFO, WARNING, ERROR)
- `websocket_port`: Port for real-time data streaming

**Model Inference:**
- `enabled`: Enable/disable GPS inference (true/false)
- `model_type`: Model architecture type
- `device`: Processing device ("cpu" or "cuda")
- `reference_point`: **CRITICAL** - Must match your actual gNodeB location
  - Should match `gnb_location` in `nr_params.json`
- `gps_boundaries`: Model normalization parameters (keep as provided)
- `normalization_type`: Keep as "zerocenter"

**TAK Integration (Optional):**
- `enabled`: Enable TAK broadcasting (true/false)
- `sensor_heading`: Sensor orientation in degrees (0° = North, 90° = East)
- `cot_url`: TAK server address (format: "IP:PORT")
- `client_cert`: Path to client certificate for TLS
- `client_key`: Path to client private key
- `ca_file`: Path to certificate authority file

**Processing Settings:**
- `max_queue_size`: Maximum files in processing queue
- `file_ready_timeout`: Wait time for file completion (seconds)
- `check_interval`: File monitoring frequency (seconds)

#### Critical Configuration Notes:

1. **Reference Point Alignment**: The `reference_point` in `config.json` must match the `gnb_location` in `nr_params.json`

2. **TAK Certificate Setup**: If using TAK integration, ensure certificate files exist and have proper permissions

---

## Starting the System

### Using the Automated Session Manager

The system provides an automated tmux session manager that starts all components:

```bash
./start_polyedge_session.sh
```

#### Custom Session Name:
```bash
./start_polyedge_session.sh -s my_session_name
```

### What the Session Manager Does:

The script creates a tmux session with three panes:

1. **Pane 0 (Top-left)**: NR UE Interface
   - Loads: `./init_nrUE.sh`
   - Purpose: Initializes 5G NR user equipment interface

2. **Pane 1 (Top-right)**: Data Streamer  
   - Loads: `./streamer`
   - Purpose: Real-time CSI data collection and processing

3. **Pane 2 (Bottom)**: TAK Integration
   - Loads: `python3 stream_pytak.py`
   - Purpose: GPS localization and TAK broadcasting

### Manual Session Management:

If you need to start components individually:

```bash
# Create new session
tmux new-session -d -s polyedge

# Split into three panes
tmux split-window -h -t polyedge
tmux select-pane -t polyedge:0.0
tmux split-window -v -t polyedge

# Start each component
tmux send-keys -t polyedge:0.0 "./init_nrUE.sh" Enter
tmux send-keys -t polyedge:0.1 "./streamer" Enter  
tmux send-keys -t polyedge:0.2 "python3 stream_pytak.py" Enter

# Attach to session
tmux attach -t polyedge
```

### Session Navigation:

Within the tmux session:
- `Ctrl+b` then arrow keys: Navigate between panes
- `Ctrl+b` then `d`: Detach from session
- `tmux attach -t polyedge`: Reattach to session
- `Ctrl+b` then `x`: Kill current pane
- `Ctrl+c`: Stop current process in pane

---

## System Operation

### Startup Sequence:

1. **Configure Files**: Set up `nr_params.json` and `config.json`
2. **Start Session**: Run `./start_polyedge_session.sh`
3. **Initialize NR UE**: Start the NR interface (Pane 0)
4. **Start Streamer**: Begin CSI data collection (Pane 1)
5. **Start TAK**: Begin GPS localization and broadcasting (Pane 2)

### Monitoring Output:

**NR UE Pane**: Shows 5G connection status and signal quality
**Streamer Pane**: Displays CSI data processing statistics
**TAK Pane**: Shows GPS coordinates and TAK broadcast status

### Normal Operation Indicators:

- Consistent CSI data flow in streamer
- GPS coordinates being generated and logged
- TAK messages being sent (if enabled)
- No error messages in logs

### Common Issues:

**No GPS Output**: Check reference point alignment between config files

**TAK Connection Failed**: Verify certificate paths and TAK server connectivity

**License Errors**: Ensure license file is valid and not expired

---

## Model Finetuning

The system supports secure model finetuning to adapt to new environments or improve accuracy. 

### Complete Finetuning Documentation

For comprehensive information on the complete finetuning pipeline including data collection, setup, and secure training, see:

** [README_SECURE_FINETUNING.md](README_SECURE_FINETUNING.md)**

This guide covers:
- **Ground Truth Collection** with DroneTag Mini
- **Data Collection** using the NR data collection system  
- **Data Setup & Processing** for training preparation
- **Secure Model Finetuning** with comprehensive options
- **Complete workflow examples** from data collection to deployment
