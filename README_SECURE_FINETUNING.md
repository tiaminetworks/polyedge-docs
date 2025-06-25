# Secure Model Finetuning Pipeline

This document describes the complete pipeline from data collection to secure finetuning, including ground truth collection with DroneTag Mini and the secure training process that protects model weights.

## Overview

The complete pipeline provides:

- **Ground Truth Collection**: Using DroneTag Mini for accurate positioning reference
- **Automated Data Collection**: Collect NR/5G signal data with precise timing
- **Data Processing**: Clean and normalize position data for training
- **Secure Finetuning**: Train models without exposing weights in plaintext
- **License-based Protection**: Model weights are never exposed in plaintext
- **Security**: Uses encryption and access controls throughout

## Complete Workflow

### 1. Ground Truth Setup (DroneTag Mini)
### 2. Data Collection (collect_nr_data.py)
### 3. Data Setup & Processing (setup_data.py)
### 4. Secure Model Finetuning (secure_finetune)

---

## 1. Ground Truth Setup - DroneTag Mini

### About DroneTag Mini

The **DroneTag Mini** is a precision positioning tracking device essential for collecting accurate ground truth data for training. It provides:

- **High-precision positioning**: Sub-meter accuracy for ground truth positioning
- **Real-time tracking**: Continuous position updates during data collection
- **Export capabilities**: JSON data export for integration with the training pipeline
- **Compact design**: Easily mountable on drones or mobile platforms

**Source**: [https://dronetag.cz/en/products/dronetag-mini/](https://dronetag.cz/en/products/dronetag-mini/)

### DroneTag Mini Setup

1. **Device Configuration**:

   - Configure DroneTag Mini via mobile app or web interface
   - Set update rate to 4Hz for data collection
   - Enable position logging
     
NOTE: Please note that DroneTag requires LTE to update and provide tracking logs, locations with no coverage would require live broadcast over BTE.

2. **Data Export Setup**:
   - Export position data as JSON format
   - Filename will be specified as argument to setup script
   - Ensure time synchronization with data collection system

### Why DroneTag Mini is Utilized

- **Timing**: Provides precise timestamps for synchronization with RF data
- **Reliability**: Professional-grade positioning with consistent performance
- **Integration**: Direct JSON export compatible with the processing pipeline

### Note: Please set the frequency of the RID broadcast to 4 Hz, found only on the drone-tag mobile app under Configuration->Direct/Broadcast Remote ID
---

## 2. Data Collection

Before running secure finetuning, you must collect training data using the NR data collection system.

### Prerequisites

- DroneTag Mini configured and ready for GPS logging
- Valid license file (`license.lic`)
- USRP B200 or compatible SDR hardware
- 5G NR signal source or test environment

### Running Data Collection

```bash
# Basic data collection
python3 organizer/collect_nr_data_launcher.py

# Custom duration and multiple samples
python3 organizer/collect_nr_data_launcher.py --duration 300 --samples 5

# Extended collection session
python3 organizer/collect_nr_data_launcher.py --duration 1200 --samples 10
```

### Collection Parameters

- `--duration`: Duration of each collection in seconds (default: 1200 = 20 minutes)
- `--samples`: Number of samples to collect (default: 1)

### During Collection

1. **Start DroneTag Mini position logging before starting PolyEdge, and land the device after PolyEdge has finished data collection**
2. **Run the collection script**
3. **Wait for system initialization**
4. **Begin movement/scenario execution**
5. **Collection completes automatically after specified duration**

### Collection Output

Data is automatically organized and stored in the system's data directories for subsequent processing.

---

## 3. Data Setup & Processing

After collecting raw data, process it for training using the setup pipeline.

### Prerequisites

- Completed data collection
- DroneTag Mini position export file (JSON format)
- Python environment with required dependencies

### Running Data Setup

```bash
# Automatic position file detection
python3 setup_data_launcher.py

# Specify position file explicitly
python3 setup_data_launcher.py --gps-file /path/to/position_export.json

# Custom reference point
python3 setup_data_launcher.py \
    --gps-file ./position_export.json \
    --reference-point "37.7749,-122.4194,10.0"
```

### Setup Parameters

- `--gps-file`: Path to DroneTag Mini position export (auto-detected if not specified)
- `--reference-point`: Reference point for position normalization in format "lat,lon,alt"

### Setup Process

The setup pipeline automatically:

1. **Position Data Processing**:
   - Cleans and validates position data from DroneTag Mini export
   - Processes timestamps and coordinates

2. **Position Normalization**:
   - Normalizes position coordinates relative to reference point
   - Generates visualization and statistics

3. **Data Verification**:
   - Verifies data completeness and integrity
   - Validates synchronization between RF and position data

### Setup Output

After successful setup, the system generates processed data files and logs for training.

### Validation

The setup process validates:
- DroneTag Mini position data format and completeness
- Time synchronization between position and RF data
- Coordinate normalization accuracy
- Data structure integrity

---

## 4. Secure Model Finetuning

After data collection and setup, run secure finetuning with the processed data. The system provides secure finetuning with license validation and encrypted model protection. All operations require valid licensing and maintain security throughout the training process.

## Usage

### Complete Workflow Example

```bash
# 1. Collect data (with DroneTag Mini running)
python3 collect_nr_data_launcher.py --duration 600 --samples 3

# 2. Setup and process data
python3 setup_data_launcher.py --reference-point "37.7749,-122.4194,10.0"

# 3. Run secure finetuning
./secure_finetune --reference-point "37.7749,-122.4194,10.0" --nr-params "nr_params.json"
```

### Basic Finetuning

**Note**: No paths are passed to secure_finetune - it automatically detects the processed data from the setup pipeline.

```bash
./secure_finetune \
    --reference-point "sensor_lat,sensor_lon,sensor_alt" \
    --nr-params "nr_params.json"
```

### Advanced Finetuning with Profiling

```bash
./secure_finetune \
    --reference-point "38.434402,-121.437406,-22.45" \
    --nr-params "nr_params.json" \
    --epochs 100 \
    --lr 1e-5 \
    --batch-size-train 32 \
    --report-enhanced-metrics \
    --accuracy-profile
```

## Command Line Arguments

### Required Arguments
- `--reference-point`: Reference point in format 'lat,lon,alt' (must match setup reference point)
- `--nr-params`: Path to NR parameters JSON file

### Training Arguments
- `--epochs`: Number of finetuning epochs (default: 50)
- `--lr`: Learning rate for finetuning (default: 1e-5, typically lower than initial training)
- `--batch-size-train`: Training batch size (default: 16)
- `--batch-size-test`: Testing batch size (default: 16)

### Analysis Options
- `--report-enhanced-metrics`: Report enhanced target positioning metrics
- `--accuracy-profile`: Generate detailed target positioning accuracy profile analysis
- `--sanitize`: Sanitize the dataset by removing invalid samples

### System Arguments
- `--run-id`: Unique identifier for this finetuning run (default: auto-generated)
- `--license-file`: Path to license file (default: `license.lic`)
- `--debug`: Enable debug mode with extra logging

## Integration with Deployment

The encrypted model produced by finetuning can be used directly with the streaming system. The system will automatically handle model deployment and integration.

## Troubleshooting

### DroneTag Mini Issues
```
