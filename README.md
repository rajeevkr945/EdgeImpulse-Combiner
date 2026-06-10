# Multi-Impulse Deployment to ESP32

## Prerequisites

- Edge Impulse API keys for each project you want to merge
- Git
- Python 3 (>= 3.7)
- pip
- unzip
- Docker (optional, for containerized usage)
- ESP-IDF 5.1.1 installed and configured for your environment, or follow the example repo README
- An ESP32 board

## Generate a merged multi-impulse deployment (local)

1. Clone the deployment-block repo:

```bash
git clone https://github.com/edgeimpulse/multi-impulse-deployment-block.git
cd multi-impulse-deployment-block
```

2. Install Python requirements:

```bash
pip install -r requirements.txt
```

3. Prepare the API keys and quantization map.

- Get one Edge Impulse API key per project you want to merge.
- Example: two projects using `KEY1` and `KEY2`.
- The quantization map must have the same number of entries as API keys.
  - Example: `1,1` (both quantized)
  - Example: `1,0` (first quantized, second unquantized)

4. Run `generate.py`.

Basic command:

```bash
python3 generate.py --out-directory ./output --api-keys ei_KEY1,ei_KEY2 --quantization-map 1,1 --force-build
```

Useful flags:

- `--engine=tflite` — force regular TensorFlow Lite model output (default is EON-compiled model)
- `--force-build` — rebuild from scratch instead of using cached artifacts (USE this for MOGU ALWAYS!!)

Example with TFLite output and forced rebuild:

```bash
python3 generate.py --out-directory ./output --api-keys ei_KEY1,ei_KEY2 --quantization-map 1,1 --engine=tflite --force-build
```

5. Confirm the generated output.

Expected output structure:

- `edge-impulse-sdk/` — Edge Impulse SDK C++ porting and headers
- `model-parameters/` — model metadata and parameters
- `tflite-model/` — TensorFlow Lite model, or an EON artifact if using `engine=EON`

## Generate using Docker (optional)

1. Build the container from the repo root:

```bash
docker build -t multi-impulse .
```

2. Run the container and mount the current directory so output is written to the host:

```bash
docker run --rm -it -v "$PWD":/home multi-impulse --api-keys ei_KEY1,ei_KEY2 --quantization-map 1,1 --engine=tflite
```

The generated files will appear under `./output` on the host.

## Install the generated deployment into the ESP32 example repo

1. Clone the ESP32 example repo:

```bash
git clone https://github.com/edgeimpulse/example-standalone-inferencing-espressif-esp32.git
cd example-standalone-inferencing-espressif-esp32
```

2. Copy the generated folders from `multi-impulse-deployment-block/output` into the example repo root:

```bash
cp -r ../multi-impulse-deployment-block/output/edge-impulse-sdk ./
cp -r ../multi-impulse-deployment-block/output/model-parameters ./
cp -r ../multi-impulse-deployment-block/output/tflite-model ./
```

> If your output contains an EON artifact instead of `tflite-model`, place the EON model and headers into the locations expected by the ESP32 example.

3. Verify the repo layout matches the example project structure.

4. If needed, update `main/CMakeLists.txt` to match the latest example repository version:

https://github.com/edgeimpulse/example-standalone-inferencing-espressif-esp32/blob/main/main/CMakeLists.txt

> The Edge Impulse SDK is updated frequently, so keep the example repo in sync with the SDK release.

## Build and flash the ESP32 firmware

1. Prepare the ESP-IDF environment:

```bash
source $IDF_PATH/export.sh
```

Or, if the example repo includes it:

```bash
./get_idf
```

2. (Optional) Configure the project:

```bash
idf.py menuconfig
```

3. Build the project:

```bash
idf.py build
```

4. If you encounter missing include errors, verify that `edge-impulse-sdk` exists at the project root and contains the required porting headers. The example `main/CMakeLists.txt` expects `EI_SDK_FOLDER ../edge-impulse-sdk` relative paths.
