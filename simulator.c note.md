-  [Rfsimulator](#Rfsimulator)
## Rfsimulator
### General
   RFSIM is a simulator designed to replace a physical RF board, allowing OAI to perform communication tests without requiring wireless hardware. It can simulate simple channels such as AWGN. RFSIM is not bound by real-time hardware sampling rates—its execution speed depends on CPU performance and can be faster or slower than real time.
### Initialization and RX Flow Diagram
#### Initialization Flow
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
#### RX Data Flow
1. **`rfsimulator_read`**  Read IQ samples from the remote side (instead of real RF reception).
2. **`rxAddInput`**  Apply the selected channel model (e.g., AWGN, SCM) to simulate real-world effects.
3. **`rx_rf`**  Forward the processed samples to the PHY layer.
4. **`ru_thread`**  Handles lower-level RF/PHY processing. Simulates southbound transmission.

### CL option
   RFSIM Command Line Options (CL options) allow users to configure the behavior of the RF simulator directly from the terminal when launching nr-softmodem or nr-uesoftmodem.
   
## simulator.c
   ### `device_init`    
   initializes a physical or simulated radio device (e.g., USRP or RFSIM) and configures its essential parameters, enabling the PHY layer in OAI to use the device for transmitting and receiving signals.
   ### `struct openair0_device`
   An abstract interface structure designed to support various physical radio devices and simulators. It contains function pointers for operations such as read, write, and start, along with internal device state. This allows different types of devices to interact with the OAI PHY layer in a unified and consistent manner.
   ### `struct openair0_config_t`
A configuration structure used to define how the RF device should be initialized and operated.  
It tells OAI how to configure the RF device based on parameters such as:
- `.conf` file settings
- Simulation mode
- IQ file paths
- Socket configurations
- Sampling rate, bandwidth, frequencies, gain
This structure is passed to `device_init()` during initialization to apply the desired device settings.

### `struct rfsimulator_state_t`

A state structure that maintains the internal configuration and runtime status of the RF simulator (`rfsimulator`) in OAI.

#### 🧾 Key Fields & Descriptions

| Field | Description |
|-------|-------------|
| `listen_sock`, `epollfd` | File descriptors for TCP socket listening and epoll (event polling) |
| `nextRxTstamp`, `lastWroteTS` | Custom timestamp types for timing control of sample read/write |
| `role` | Simulator role: gNB or UE (`simuRole`) |
| `ip`, `port` | IP and port for TCP connection (client or server mode) |
| `saveIQfile` | Whether to save I/Q samples to file (`saviq` option) |
| `buf[MAX_FD_RFSIMU]` | Buffers for storing I/Q data blocks during simulation |
| `next_buf` | Index pointer for buffer selection |
| `rx_num_channels`, `tx_num_channels` | Number of RX/TX antenna channels |
| `sample_rate`, `rx_freq`, `tx_bw` | RF communication parameters (sampling rate, frequency, bandwidth) |
| `channelmod` | Whether to enable channel model simulation (e.g., fading, multipath) |
| `chan_pathloss`, `chan_forgetfact` | Pathloss and channel memory factor for SCM model |
| `chan_offset` | Offset applied to I/Q data for simulating delay |
| `telnetcmd_qid`, `poll_telnetcmdq` | Optional telnet CLI interface for commands (e.g., runtime control) |
| `wait_timeout` | Timeout (in seconds) when waiting for UE to connect |
| `prop_delay_ms` | Simulated propagation delay between gNB and UE |
| `hanging_workaround` | Flag to enable workaround for connection issues |


