[Initialization and RX Flow Diagram](#Initialization-and-RX-Flow-Diagram)
## Initialization and RX Flow Diagram

### General
RFSIM is a simulator designed to replace a physical RF board, allowing OAI to perform communication tests without requiring wireless hardware. It can simulate simple channels such as AWGN. RFSIM is not bound by real-time hardware sampling rates—its execution speed depends on CPU performance and can be faster or slower than real time.

### Initialization Flow
1. **`load_lib`**  Load the RFSIM shared library (`librfsimulator.so`) dynamically.
2. **`device_init`** Initialize the device and set function pointers like `trx_read_func`.
3. **`rfsimulator_readconfig`**  Read configuration values from `.conf` files (command line or config file).
4. **`rfsimu_params`**
   Based on the options provided, the flow splits:
   - If `chanmod` is selected:
     - `load_channellist`: Load a predefined channel model.
     - `new_channel_desc_scm`: Create a new SCM (Spatial Channel Model) descriptor.
   - If `saviq` is selected:
     - `saveIQfile`: Save transmitted IQ samples to a file.
### RX Data Flow

## rifsim simulator.c
