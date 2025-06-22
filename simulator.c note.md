-  [Rfsimulator](#Rfsimulator)
    -  [simulator.c](#simulatorc)
    -  [device_init](#device_init)
    -  [rfsimulator_readconfig()](#rfsimulator_readconfig)
    -  [Set openair0_device](#set-the-function-pointers-and-members-in-the-openair0_device-structure)
## OAI Project Directory Structure 

| Directory Path         | Description |
|------------------------|-------------|
| `openair1/`            | Implementation of Layer 1 (PHY). Includes LDPC encoder/decoder source files such as `nrLDPC_encoder.c` and `ldpc_decoder.c`. |
| `radio/rfsimulator/`   | RFSIM simulator implementation. Handles simulated RF channel transmission between gNB and UE. Key file: `simulator.c`. |
| `cmake_targets/`       | Build scripts and configuration for compiling and launching softmodem. Main execution happens from this directory. |
| `executables/`         | Contains main entry points for gNB, UE, and other top-level executables. |
| `openair2/`            | Implementation of Layer 2 modules (MAC / RLC / PDCP / RRC). Can be skipped for LDPC-focused study. |
| `openair3/`            | Layer 3 modules (e.g., NGAP, GTP for core network). Not directly related to LDPC. |
| `CMakeLists.txt`       | Top-level CMake configuration file for building the entire project. |
| `doc/`                 | Documentation and architecture descriptions (e.g., system diagrams, flow explanations). |
| `tools/`               | Developer tools for code formatting, analysis, and maintenance. |
| `docker/`              | Docker build files and environment setup for container-based deployment. |
| `ci-scripts/`          | CI/CD scripts and configurations for automated testing and integration. |


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

#### Key Fields & Descriptions

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
| `channelmod` | Whether to enable channel model simulation |
| `chan_pathloss`, `chan_forgetfact` | Pathloss and channel memory factor for SCM model |
| `chan_offset` | Offset applied to I/Q data for simulating delay |
| `telnetcmd_qid`, `poll_telnetcmdq` | Optional telnet CLI interface for commands  |
| `wait_timeout` | Timeout (in seconds) when waiting for UE to connect |
| `prop_delay_ms` | Simulated propagation delay between gNB and UE |
| `hanging_workaround` | Flag to enable workaround for connection issues |


####`rfsimulator_state_t *rfsimulator = calloc(sizeof(rfsimulator_state_t), 1);`

This line allocates memory for a new `rfsimulator_state_t` structure.  
All fields within the structure are initialized to zero.  
The pointer to the allocated memory is stored in the variable `rfsimulator`.
- `calloc(n, size)` allocates memory for `n` elements, each of size `size`, and zero-initializes them.
- In this case, `n = 1`, `size = sizeof(rfsimulator_state_t)`, so it creates one zeroed-out instance of the struct.

### `rfsimulator_readconfig()`

This function reads configuration settings for the `rfsimulator` module and initializes its internal parameters.
####  Key Steps:
1. **Define and initialize parameter descriptor array**  
   - Declares the list of expected parameters (e.g., IP, port, options).
     
2. **Read configuration from the `.conf` file**  
   - Uses the appropriate section (e.g., `[rfsimulator]`) to extract values.
     
3. **Parse the `options` parameter**  
   - Determines which simulation features are enabled (e.g., `chanmod`, `saviq`).
     
4. **Determine simulator role**  
   - Based on IP address or hostname, sets the role to `client` (UE) or `server` (gNB).
     
This function is typically called during `device_init()` to prepare the simulator before communication starts.
#### Flow Overview

```text
main() or nr-softmodem
        ↓
Load RF device plugin (e.g., librfsimulator.so)
        ↓
device_init(openair0_device *device, openair0_config_t *cfg)
        ↓
       ┌────────────────────────────────────┐
       │   Called inside device_init()      │
       │   → rfsimulator_readconfig(cfg)    │
       └────────────────────────────────────┘
        ↓
Initialize rfsimulator_state_t based on cfg parameters (IP, port, options)
```

#### Code syntax and comments
##### `paramdef_t rfsimu_params[] = RFSIMULATOR_PARAMS_DESC;`
This line defines a parameter descriptor array used by the configuration system in OAI.

- `paramdef_t` is a structure that describes a configuration parameter (e.g., name, type, default value, pointer to target variable).
- `RFSIMULATOR_PARAMS_DESC` is a macro or a static constant array that contains all supported RFSIM configuration options.
- This array is passed to functions like `config_get()` to load values from the `.conf` file or command-line.

This is how OAI knows **which parameters to read**, **what types they are**, and **where to store them**.

##### 🔍 Finding the Index of a Parameter: `config_paramidx_fromname()`

This function searches a parameter descriptor array (like `rfsimu_params[]`) and returns the index of a parameter with the specified name.
**Defined in:** `config_userapi.c`

Used to locate options such as `"options"` so their values can be processed later.

Look up the index of the parameter named "options" in the rfsimu_params array.

The result is stored in variable p.

#define RFSIMU_OPTIONS_PARAMNAME "options"

##### `config_get()` — Load Configuration Values

This function loads parameter values from a configuration source (e.g., `.conf` file or command-line) and stores them in the provided descriptor array.

**Defined in:** `config_userapi.c`

- **`config_get_if()`**  
  Returns the active configuration interface (typically a macro for `uniqCfg`).

- **`RFSIMU_SECTION`**  
  A macro that defines the section name in the `.conf` file, typically `"rfsimulator"`.
---
- **`Function Behavior`**  
`config_get()` reads the `[rfsimulator]` section of the configuration file,  
matches parameter names to entries in `rfsimu_params`,  
and fills their values into the corresponding target variables.

This step is essential to prepare the RF simulator with the correct user-defined options  
such as **IP**, **port**, and `options=chanmod,saviq`, etc.

#####  Parsing `options` Parameter in RFSIMULATOR

This loop iterates over the list of `options` (passed via `.conf` or command line) and enables specific simulation features such as IQ saving (`saviq`) or channel modeling (`chanmod`).

```c
for (int i = 0; i < rfsimu_params[p].numelt; i++) {
    // Check if the current option is "saviq" → enable IQ saving
    if (strcmp(rfsimu_params[p].strlistptr[i], "saviq") == 0) {
        // Open the IQ save file in write mode (create if not exist, truncate if it does)
        rfsimulator->saveIQfile = open(saveF, O_APPEND | O_CREAT | O_TRUNC | O_WRONLY, 0666);

        if (rfsimulator->saveIQfile != -1) {
            // Successfully opened file for IQ saving
            LOG_I(HW, "Will save written IQ samples in %s\n", saveF);
        } else {
            // Failed to open file → print error and abort
            LOG_E(HW, "open(%s) failed for IQ saving, errno(%d)\n", saveF, errno);
            exit(-1);
        }

        // No need to check further options once "saviq" is handled
        break;

    } else if (strcmp(rfsimu_params[p].strlistptr[i], "chanmod") == 0) {
        // Enable RF channel model simulation
        init_channelmod();

        // Load predefined channel list with current RF parameters
        load_channellist(
            rfsimulator->tx_num_channels,
            rfsimulator->rx_num_channels,
            rfsimulator->sample_rate,
            rfsimulator->rx_freq,
            rfsimulator->tx_bw
        );

        // Activate channel modeling for future RX sample processing
        rfsimulator->channelmod = true;

    } else {
        // If the option is not recognized, print an error and terminate
        fprintf(stderr, "unknown rfsimulator option: %s\n", rfsimu_params[p].strlistptr[i]);
        exit(-1);
    }
}
```
When saving IQ samples (e.g., using the `"saviq"` option), the `open()` function is used with the following flags:

| Flag        | Meaning                                              |
|-------------|------------------------------------------------------|
| `O_APPEND`  | Append data to the end of the file                   |
| `O_CREAT`   | Create the file if it does not exist                 |
| `O_TRUNC`   | Truncate the file to 0 length if it exists           |
| `O_WRONLY`  | Open the file in write-only mode                     |

| Macro           | Function                                    |
| --------------- | ------------------------------------------- |
| `LOG_I(M, ...)` | Informational log, used for status messages |
| `LOG_E(M, ...)` | Error log, used when something goes wrong   |


`init_channelmod()`
Initializes channel modeling structures. Sets default parameters (e.g., path loss, fading model, memory factor) for RF simulation.

`load_channellist(tx, rx, rate, freq, bw)`
Creates and configures a list of channels between TX and RX paths based on system parameters:
- `tx` → Transmit channels
- `rx` → Receive channels
- `rate` → Sample rate (Hz)
- `freq` → Center frequency (Hz)
- `bw` → Bandwidth (Hz)

These functions are used in `rfsimulator_readconfig()` when the `"chanmod"` option is enabled.

---
This logic appears inside the `rfsimulator_readconfig()` function and determines whether the RFSIM instance should behave as a **server** (e.g., gNB) or **client** (e.g., UE) based on the configured IP string.

```c
if ( strncasecmp(rfsimulator->ip, "enb", 3) == 0 ||
     strncasecmp(rfsimulator->ip, "server", 3) == 0 ) {
    rfsimulator->role = SIMU_ROLE_SERVER;
} else {
    rfsimulator->role = SIMU_ROLE_CLIENT;
}
```
---

 Explanation

- `strncasecmp(a, b, n)`: Compares the first `n` characters of two strings case-insensitively.
- If the IP string starts with `"enb"` or `"server"`:
  - The simulator sets its role to `SIMU_ROLE_SERVER` → acts like a gNB.
- Otherwise:
  - The simulator sets its role to `SIMU_ROLE_CLIENT` → acts like a UE.

This classification is crucial for setting up the proper TCP communication direction (server listens, client connects).

---

### Set the function pointers and members in the openair0_device structure
```c
 device->trx_start_func = rfsimulator->role == SIMU_ROLE_SERVER ? startServer : startClient;
  device->trx_get_stats_func   = rfsimulator_get_stats;
  device->trx_reset_stats_func = rfsimulator_reset_stats;
  device->trx_end_func         = rfsimulator->role == SIMU_ROLE_SERVER ? stopServer : rfsimulator_end;
  device->trx_stop_func        = rfsimulator_stop;
  device->trx_set_freq_func    = rfsimulator_set_freq;
  device->trx_set_gains_func   = rfsimulator_set_gains;
  device->trx_write_func       = rfsimulator_write;
  device->trx_read_func      = rfsimulator_read;
  /* let's pretend to be a b2x0 */
  device->type = RFSIMULATOR;
  openair0_cfg[0].rx_gain[0] = 0;
  device->openair0_cfg=&openair0_cfg[0];
  device->priv = rfsimulator;
  device->trx_write_init = rfsimulator_write_init;
```

