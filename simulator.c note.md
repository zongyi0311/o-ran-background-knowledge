-  [Rfsimulator](#Rfsimulator)
    -  [simulator.c](#simulatorc)
    -  [device_init](#device_init)
    -  [rfsimulator_readconfig()](#rfsimulator_readconfig)
    -  [Set openair0_device](#set-the-function-pointers-and-members-in-the-openair0_device-structure)
       -  [startServer()](#startServer)
       -  [startClient()](#startClient)
       -  [stopServer()](#stopServer)
# OAI Project Directory Structure 

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


# Rfsimulator
## General
   RFSIM is a simulator designed to replace a physical RF board, allowing OAI to perform communication tests without requiring wireless hardware. It can simulate simple channels such as AWGN. RFSIM is not bound by real-time hardware sampling rates—its execution speed depends on CPU performance and can be faster or slower than real time.
## Initialization and RX Flow Diagram
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
1. **`rfsimulator_read`**  Read IQ samples from the remote side (instead of real RF reception).
2. **`rxAddInput`**  Apply the selected channel model (e.g., AWGN, SCM) to simulate real-world effects.
3. **`rx_rf`**  Forward the processed samples to the PHY layer.
4. **`ru_thread`**  Handles lower-level RF/PHY processing. Simulates southbound transmission.

## CL option
   RFSIM Command Line Options (CL options) allow users to configure the behavior of the RF simulator directly from the terminal when launching nr-softmodem or nr-uesoftmodem.
   
# simulator.c
   ## `device_init`    
   initializes a physical or simulated radio device (e.g., USRP or RFSIM) and configures its essential parameters, enabling the PHY layer in OAI to use the device for transmitting and receiving signals.
   ## `struct openair0_device`
   An abstract interface structure designed to support various physical radio devices and simulators. It contains function pointers for operations such as read, write, and start, along with internal device state. This allows different types of devices to interact with the OAI PHY layer in a unified and consistent manner.
   ## `struct openair0_config_t`
A configuration structure used to define how the RF device should be initialized and operated.  
It tells OAI how to configure the RF device based on parameters such as:
- `.conf` file settings
- Simulation mode
- IQ file paths
- Socket configurations
- Sampling rate, bandwidth, frequencies, gain
This structure is passed to `device_init()` during initialization to apply the desired device settings.

## `struct rfsimulator_state_t`

A state structure that maintains the internal configuration and runtime status of the RF simulator (`rfsimulator`) in OAI.

### Key Fields & Descriptions

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


###`rfsimulator_state_t *rfsimulator = calloc(sizeof(rfsimulator_state_t), 1);`

This line allocates memory for a new `rfsimulator_state_t` structure.  
All fields within the structure are initialized to zero.  
The pointer to the allocated memory is stored in the variable `rfsimulator`.
- `calloc(n, size)` allocates memory for `n` elements, each of size `size`, and zero-initializes them.
- In this case, `n = 1`, `size = sizeof(rfsimulator_state_t)`, so it creates one zeroed-out instance of the struct.

## `rfsimulator_readconfig()`

This function reads configuration settings for the `rfsimulator` module and initializes its internal parameters.
###  Key Steps:
1. **Define and initialize parameter descriptor array**  
   - Declares the list of expected parameters (e.g., IP, port, options).
     
2. **Read configuration from the `.conf` file**  
   - Uses the appropriate section (e.g., `[rfsimulator]`) to extract values.
     
3. **Parse the `options` parameter**  
   - Determines which simulation features are enabled (e.g., `chanmod`, `saviq`).
     
4. **Determine simulator role**  
   - Based on IP address or hostname, sets the role to `client` (UE) or `server` (gNB).
     
This function is typically called during `device_init()` to prepare the simulator before communication starts.
### Flow Overview

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

### Code syntax and comments
#### `paramdef_t rfsimu_params[] = RFSIMULATOR_PARAMS_DESC;`
This line defines a parameter descriptor array used by the configuration system in OAI.

- `paramdef_t` is a structure that describes a configuration parameter (e.g., name, type, default value, pointer to target variable).
- `RFSIMULATOR_PARAMS_DESC` is a macro or a static constant array that contains all supported RFSIM configuration options.
- This array is passed to functions like `config_get()` to load values from the `.conf` file or command-line.

This is how OAI knows **which parameters to read**, **what types they are**, and **where to store them**.

####  Finding the Index of a Parameter: `config_paramidx_fromname()`

This function searches a parameter descriptor array (like `rfsimu_params[]`) and returns the index of a parameter with the specified name.
**Defined in:** `config_userapi.c`

Used to locate options such as `"options"` so their values can be processed later.

Look up the index of the parameter named "options" in the rfsimu_params array.

The result is stored in variable p.

#define RFSIMU_OPTIONS_PARAMNAME "options"

#### `config_get()` — Load Configuration Values

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

####  Parsing `options` Parameter in RFSIMULATOR

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
#### rfsimulator_readconfig() Flowchart 

```text
┌────────────────────────────────────────────────────┐
│ Function Entry: rfsimulator_readconfig()           │
└────────────┬───────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────────────┐
│ Declare: saveF = NULL, modelname = NULL            │
│ Create param array: rfsimu_params[]                │
│ ← from RFSIMULATOR_PARAMS_DESC macro               │
└────────────┬───────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────────────┐
│ Lookup index `p` for "options" in rfsimu_params[]  │
│ using config_paramidx_fromname()                   │
└────────────┬───────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────────────┐
│ Call config_get() to load configuration            │
│ If ret < 0 → AssertFatal() to abort                │
└────────────┬───────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────────────┐
│ Initialize: rfsimulator->saveIQfile = -1           │
└────────────┬───────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────────────┐
│ Loop through rfsimu_params[p].strlistptr[]         │
│ for i = 0 to numelt                                │
└────────────┬───────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────────────┐
│ If option == "saviq":                              │
│ → Open file with open(saveF, flags, 0666)          │
│   If success: log message                          │
│   Else: print error and exit(-1)                   │
└────────────┬───────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────────────┐
│ If option == "chanmod":                            │
│ → Call init_channelmod()                           │
│ → Call load_channellist(...)                       │
│ → Set rfsimulator->channelmod = true               │
└────────────┬───────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────────────┐
│ Else (unknown option):                             │
│ → Print error and exit(-1)                         │
└────────────┬───────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────────────┐
│ Check if environment variable RFSIMULATOR exists   │
│ If so:                                             │
│ → Assign rfsimulator->ip from getenv("RFSIMULATOR")│
│ → Log warning (deprecated usage)                   │
│ → sleep(10)                                        │
└────────────┬───────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────────────┐
│ Determine simulator role based on IP prefix:       │
│ If IP starts with "enb" or "server" → SERVER role  │
│ Else → CLIENT role                                 │
└────────────────────────────────────────────────────┘
```
---

## Set the function pointers and members in the openair0_device structure
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
### startServer()
```c
static int startServer(openair0_device *device)
{
  int sock = -1;
  struct addrinfo *results = NULL;
  struct addrinfo *rp = NULL;

  rfsimulator_state_t *t = (rfsimulator_state_t *)device->priv;
  t->role = SIMU_ROLE_SERVER;

  char port[6];
  snprintf(port, sizeof(port), "%d", t->port);

  struct addrinfo hints = {
    .ai_family = AF_INET6,//IPv6 addresses.
    .ai_socktype = SOCK_STREAM,//want TCP sockets, UDP use SOCK_DGRAM
    .ai_flags = AI_PASSIVE,//a server, so you want to bind to a local address (not connect).
  };
```
---
This is the **server-side startup function** for the RF simulator in OAI. It prepares a TCP server socket using IPv6 and configures it to listen on the configured port.

---
| Code                          | Purpose |
|-------------------------------|---------|
| `int sock = -1;`              | File descriptor placeholder for socket |
| `struct addrinfo *results = NULL;` | Will store the list of possible addresses returned by `getaddrinfo()` |
| `struct addrinfo *rp = NULL;` | Used to iterate through address list |
| `t->role = SIMU_ROLE_SERVER;` | Set the role of the simulator to server |
| `char port[6];`               | Buffer to store port number string (5 digits + null terminator) |
| `snprintf(...)`              | Convert the numeric port (`t->port`) to a string |

---
#### getaddrinfo()
| Function Name     | Purpose                                           | Defined In   |
|-------------------|---------------------------------------------------|--------------|
| `getaddrinfo()`   | Query address information                         | `<netdb.h>`  |
| `gai_strerror()`  | Interpret error codes (specific to `getaddrinfo`) | `<netdb.h>`  |
| `freeaddrinfo()`  | Free the address information results              | `<netdb.h>`  |

**What Does `getaddrinfo()` Return?**

The `getaddrinfo()` function returns a **linked list of `struct addrinfo`**, where each node represents a possible socket connection configuration.

| Field Name   | Description |
|--------------|-------------|
| `ai_family`  | Address family (e.g., `AF_INET` for IPv4, `AF_INET6` for IPv6) AF_UNSPEC for Not specified |
| `ai_socktype`| Socket type (e.g., `SOCK_STREAM` for TCP, `SOCK_DGRAM` for UDP) |
| `ai_protocol`| Protocol (e.g., `IPPROTO_TCP`, `IPPROTO_UDP`) |
| `ai_addrlen` | Length of the `ai_addr` field |
| `ai_addr`    | Pointer to a `sockaddr` structure containing the actual address information |
| `ai_next`    | Pointer to the next `addrinfo` node in the list (used for iteration) |

Each node describes one possible way to connect to the target host and service.

#### socket()

int socket(int domain, int type, int protocol);
| Parameter   | Description                          | Common Values                      |
|-------------|--------------------------------------|------------------------------------|
| `domain`    | Address family (IPv4, IPv6, etc.)    | `AF_INET` (IPv4), `AF_INET6` (IPv6) |
| `type`      | Type of socket                       | `SOCK_STREAM` (TCP), `SOCK_DGRAM` (UDP) |
| `protocol`  | Transport protocol (usually 0)       | `0` (default), `IPPROTO_TCP`, `IPPROTO_UDP` |

 Set `protocol = 0` to automatically choose the default for the given type.

 #### setsockopt()

 ```c
int setsockopt(int sockfd, int level, int option_name,const void *option_value, socklen_t option_len);

```
| Parameter       | Meaning                                      |
|------------------|----------------------------------------------|
| `sockfd`         | The socket file descriptor                   |
| `level`          | Protocol level (e.g., `SOL_SOCKET`, `IPPROTO_TCP`) |
| `option_name`    | Name of the option to set                    |
| `option_value`   | Pointer to the value to set                  |
| `option_len`     | Length (in bytes) of the value               |

---

| Level           | Option Name       | Type     | Purpose                                                  |
|-----------------|-------------------|----------|----------------------------------------------------------|
| `SOL_SOCKET`    | `SO_REUSEADDR`    | `int`    | Allow reuse of local addresses (e.g., avoid "address in use") |
| `SOL_SOCKET`    | `SO_REUSEPORT`    | `int`    | Allow multiple sockets to bind to the same port (Linux)  |
| `SOL_SOCKET`    | `SO_RCVBUF`       | `int`    | Set receive buffer size                                  |
| `SOL_SOCKET`    | `SO_SNDBUF`       | `int`    | Set send buffer size                                     |
| `SOL_SOCKET`    | `SO_KEEPALIVE`    | `int`    | Enable periodic keepalive messages                       |
| `IPPROTO_TCP`   | `TCP_NODELAY`     | `int`    | Disable Nagle’s algorithm (send data immediately)        |
| `IPPROTO_IPV6`  | `IPV6_V6ONLY`     | `int`    | Limit IPv6 socket to IPv6 only (default = 1)             |

---

-  setsockopt(sock, IPPROTO_IPV6, IPV6_V6ONLY, &disable, sizeof(int));
-  Makes an IPv6 socket also accept IPv4 connections via IPv4-mapped addresses.

-  setsockopt(sock, SOL_SOCKET, SO_REUSEADDR, &enable, sizeof(int))
-  allow reusing a local address (IP + port)** even if it is still in the `TIME_WAIT` state.

Why Use `SO_REUSEADDR`?

- When a server restarts quickly, the port may still be reserved by the OS.
- Without this option, `bind()` would fail with:  
   `bind: Address already in use`
- With `SO_REUSEADDR`, the server can rebind the port **immediately**.
---

#### bind()

```c
if (bind(sock, rp->ai_addr, rp->ai_addrlen) == 0) {
    break;
}
```

**Purpose:**
This line attempts to **bind the socket to a local IP address and port**.

| Parameter         | Description                             |
|-------------------|-----------------------------------------|
| `sock`            | The socket file descriptor              |
| `rp->ai_addr`     | Pointer to the address to bind          |
| `rp->ai_addrlen`  | Size of the address structure           |

---

**Why `bind()` May Fail**

| Common Reason                  | Description                                 |
|--------------------------------|---------------------------------------------|
| Port already in use            | Another process is using the same port      |
| Permission denied              | Binding to a port < 1024 without root       |
| Address incompatible           | The address is not supported by socket type |

---

**Summary**

| Function | Purpose                          |
|----------|----------------------------------|
| `bind()` | Binds a socket to a local address |
| `== 0`   | Success, proceed to listen/accept |
| `!= 0`   | Failure, try next address         |

---

#### listen()

```c
int listen(int sockfd, int backlog);
```

- **Purpose**: Marks the socket as a listening socket for incoming TCP connections.
- **Precondition**: Must be `bind()`-ed before calling.
- **backlog**: Maximum number of pending connections in the queue.

| Return | Meaning        |
|--------|----------------|
| `0`    | Success        |
| `-1`   | Failure (check `errno`) |

**Without `listen()`, `accept()` will fail!**

---

| Code                             | Meaning |
|----------------------------------|---------|
| `struct epoll_event ev = {0};`   | Initializes the struct to all zeros |
| `ev.events = EPOLLIN;`           | Register interest in **readable input events** |
| `ev.data.ptr = NULL;`            | No custom data attached for this event (can store user-defined context later) |

#### epoll_ctl()

```c
if (epoll_ctl(t->epollfd, EPOLL_CTL_ADD, t->listen_sock, &ev) != 0) {
    LOG_E(HW, "epoll_ctl(EPOLL_CTL_ADD) failed, errno(%d)\n", errno);
    return -1;
}
```
---

**Purpose**

Registers the **listening socket** (`t->listen_sock`) to the `epoll` instance (`t->epollfd`) to monitor for **incoming connection events**.

---

**Function Syntax**

```c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```
---

| Field           | Value                    | Meaning                         |
|------------------|---------------------------|----------------------------------|
| `epfd`           | `t->epollfd`              | epoll instance for this server  |
| `op`             | `EPOLL_CTL_ADD`           | Add new socket to watch list    |
| `fd`             | `t->listen_sock`          | The server socket in listening state |
| `event`          | `&ev`                     | Watch for readable events (`EPOLLIN`) |

---

**Error Handling**

If `epoll_ctl()` fails:

- Logs an error with `errno`
- Returns `-1` to indicate server setup failed
---
This line is **crucial** for setting up an event-driven server using `epoll`.  
Without this, `epoll_wait()` will not receive any connection events from clients.

#### startServer() Flowchart

```text
┌────────────────────────────┐
│ Initialize variables       │
│  - sock = -1               │
│  - results, rp = NULL      │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ Set role to SERVER         │
│ Format port as string      │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ Set up hints for IPv6      │
│  - ai_family = AF_INET6    │
│  - ai_socktype = SOCK_STREAM │
│  - ai_flags = AI_PASSIVE   │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ Call getaddrinfo()         │
│ If fail → print error      │
│             ↳ return -1    │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ Loop through addr list     │
│ for (rp = results; ...)    │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ Create socket              │
│ If fail → continue         │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ setsockopt: Disable v6only │
│ If fail → continue         │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ setsockopt: Enable reuse   │
│ If fail → continue         │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ bind() to addr             │
│ If success → break         │
│ else → close socket        │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ freeaddrinfo(results)      │
│ Check if socket valid      │
│ If invalid → return -1     │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ listen(sock, backlog)      │
│ If fail → return -1        │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ Register socket in epoll   │
│ epoll_ctl(..., ADD, ...)   │
│ If fail → return -1        │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ Save sock to t->listen_sock│
│ Return 0 (Success)         │
└────────────────────────────┘
```
### startClient()

sock = client_try_connect(t->ip, t->port);

#### client_try_connect()

```c
s = getaddrinfo(host, dport, &hints, &result);
if (s != 0) {
  LOG_E(HW, "getaddrinfo: %s\n", gai_strerror(s));
  return -1;
}
```
**Purpose:**  
This code uses `getaddrinfo()` to resolve a hostname and service (port) into a list of socket address structures, and checks for failure.

---
**`getaddrinfo()` Parameters**

| Parameter   | Description                                        |
|-------------|----------------------------------------------------|
| `host`      | Hostname or IP address to resolve                  |
| `dport`     | Port number or service name (as string)            |
| `&hints`    | Pointer to `addrinfo` struct with filtering hints  |
| `&result`   | Output: linked list of resolved addresses          |

---

#### connect()
```c
if (connect(sock, rp->ai_addr, rp->ai_addrlen) != -1) {
  break;
}
```
**Purpose**  
This code attempts to connect the socket to the remote address provided by `rp` (from the `getaddrinfo()` results). If the connection is successful, it breaks out of the loop.

---
```c
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```
---
**Purpose**

The `connect()` function is used by a client to **initiate a connection to a remote server** using a socket. It is typically used with **TCP sockets** (`SOCK_STREAM`).

---
**Parameters**

| Parameter     | Type                    | Description |
|---------------|-------------------------|-------------|
| `sockfd`      | `int`                   | The file descriptor of the socket created via `socket()` |
| `addr`        | `const struct sockaddr *` | Pointer to a `sockaddr` structure containing the destination address |
| `addrlen`     | `socklen_t`             | Size of the `addr` structure |

---

#### setblocking()

**Purpose**

This function sets the specified socket `sock` to either:

- **Blocking mode**: I/O calls wait until completion (default)
- **Non-blocking mode**: I/O calls return immediately if they would block
---

### 🧾 Step-by-Step Breakdown

| Section        | Code                              | Explanation |
|----------------|-----------------------------------|-------------|
| Get flags      | `fcntl(sock, F_GETFL)`            | Retrieves current socket flags (e.g. `O_NONBLOCK`) |
| Choose mode    | `if (active == blocking)`         | Uses the `blocking_t` enum to decide desired mode |
| Modify flags   | `opts |= O_NONBLOCK` or `&= ~O_NONBLOCK` | Sets or clears the non-blocking flag |
| Apply flags    | `fcntl(sock, F_SETFL, opts)`      | Applies the modified flags to the socket |
| Error handling | `if (opts < 0)`                   | If `fcntl()` fails, logs error and returns `-1` |
| Success        | `return 0;`                       | Returns 0 if everything succeeded |

---

**Example**

```c
int sock = socket(AF_INET, SOCK_STREAM, 0);
setblocking(sock, notBlocking); // Set the socket to non-blocking mode
```

---
**blocking_t**
```c
enum blocking_t {
  notBlocking,
  blocking
};
```
---

**Purpose**

This `enum` defines two possible socket modes — blocking and non-blocking. It's typically used as a parameter to functions like `setblocking()` to control how a socket behaves during I/O operations.

---
| Name         | Value | Description                             |
|--------------|--------|-----------------------------------------|
| `notBlocking`| `0`    | Sets the socket to **non-blocking** mode |
| `blocking`   | `1`    | Sets the socket to **blocking** mode     |

> In C, enum members are automatically assigned integer values starting from `0`.

---

#### startClient() Flowchart

```text
┌──────────────────────────────────────────────┐
│ Enter startClient(openair0_device *device)   │
└────────────────────────┬─────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────┐
│ device->priv → rfsimulator_state_t *t        │
│ Set t->role = SIMU_ROLE_CLIENT               │
└────────────────────────┬─────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────┐
│ Loop: while (true)                           │
│ Attempt connection via client_try_connect()  │
└────────────────────────┬─────────────────────┘
                         │
                ┌────────▼────────┐
                │ sock > 0 ?      │───────→ Yes ───► Connected → break loop
                └──────┬──────────┘
                       │ No
                       ▼
       ┌──────────────────────────────────────┐
       │ Print "connect() failed, errno"      │
       │ sleep(1);                            │
       └──────────────────────────────────────┘
                         ▲
                         │ ←←←←←←←←←←←←←←←←←←←←←←←←←←←

┌──────────────────────────────────────────────┐
│ After loop: connection established           │
│ Call setblocking(sock, notBlocking)          │
└────────────────────────┬─────────────────────┘
                         │
                ┌────────▼────────┐
                │ return -1 ?     │──────→ Yes → exit with error
                └──────┬──────────┘
                       │ No
                       ▼
┌──────────────────────────────────────────────┐
│ Call allocCirBuf(t, sock)                    │
│ If success → return 0                        │
│ If fail    → return -1                       │
└──────────────────────────────────────────────┘
```
---

### stopServer()

Shutdown Server and Release Resources

```c
static void stopServer(openair0_device *device) {
  rfsimulator_state_t *t = (rfsimulator_state_t *) device->priv;
  DevAssert(t != NULL);
  close(t->listen_sock);
  rfsimulator_end(device);
}
```
---

Purpose

The `stopServer()` function is used to **shutdown the server-side socket** and **clean up resources** associated with the RF simulator. It's typically called when the simulation ends, restarts, or the process is exiting.

---
| Line | Code | Explanation |
|------|------|-------------|
| 1    | `rfsimulator_state_t *t = (rfsimulator_state_t *) device->priv;` | Retrieve the simulator state from the device object |
| 2    | `DevAssert(t != NULL);` | Ensure the simulator state pointer is not null (safety check) |
| 3    | `close(t->listen_sock);` | Close the listening socket to free OS-level resources |
| 4    | `rfsimulator_end(device);` | Perform additional cleanup (e.g., buffers, epoll removal, memory) |

---


**When to Use**

- When the server is shutting down (e.g., during `SIGINT`)
- Before restarting the simulator
- After tests finish to release sockets and memory

---
### rfsimulator_end()

```c
static void rfsimulator_end(openair0_device *device) {
  rfsimulator_state_t* s = device->priv;
  for (int i = 0; i < MAX_FD_RFSIMU; i++) {
    buffer_t *b = &s->buf[i];
    if (b->conn_sock >= 0 )
      removeCirBuf(s, b);
  }
  close(s->epollfd);
  free(s);
}
```
---

| Line | Code | Explanation |
|------|------|-------------|
| 1    | `buffer_t *b = &s->buf[i];`              | Access each buffer associated with a connection |
| 2    | `if (b->conn_sock >= 0)`                 | Check if the connection is active |
| 3    | `removeCirBuf(s, b);`                    | Remove circular buffer and clean up the connection |
| 4    | `close(s->epollfd);`                     | Close the epoll file descriptor used for polling events |
| 5    | `free(s);`                               | Free the dynamically allocated simulator state memory |

---

| Component         | Description |
|------------------|-------------|
| `MAX_FD_RFSIMU`   | Maximum number of socket connections (typically 250) |
| `buffer_t`        | Structure representing a single client buffer/connection |
| `removeCirBuf()`  | Function to remove epoll watch, close socket, free buffer |
| `s->epollfd`      | epoll instance used to monitor events |
| `s->buf[]`        | Array of buffer_t, one for each potential connection |

---

##### buffer_t

```c
typedef struct buffer_s {
  int conn_sock;                         // TCP socket descriptor for the connected client (UE or gNB)
  openair0_timestamp lastReceivedTS;     // Timestamp of the last received data (used for sync or ordering)
  bool headerMode;                       // True if currently parsing a packet header
  bool trashingPacket;                   // True if the current packet is being discarded (e.g., due to error)
  samplesBlockHeader_t th;               // Header information for the current incoming packet
  char *transferPtr;                     // Pointer to the current write position in the receive buffer
  uint64_t remainToTransfer;             // Number of bytes left to complete the current packet
  char *circularBufEnd;                  // Pointer to the end of the circular buffer (for wrap-around detection)
  sample_t *circularBuf;                 // Main circular buffer that stores IQ (complex) samples
  channel_desc_t *channel_model;         // Pointer to the associated channel model (fading, multipath, etc.)
} buffer_t;
```

---

`buffer_t` is used in the RF simulator to manage the **state, data, and channel conditions** for each active TCP connection. Each connection has its own instance of this structure.

---

| Field Name          | Type                  | Description |
|---------------------|-----------------------|-------------|
| `conn_sock`         | `int`                 | File descriptor for the socket associated with this connection |
| `lastReceivedTS`    | `openair0_timestamp`  | Timestamp of the last received packet |
| `headerMode`        | `bool`                | Indicates whether the simulator is currently parsing a packet header |
| `trashingPacket`    | `bool`                | Indicates whether the current packet is being discarded due to error |
| `th`                | `samplesBlockHeader_t`| Holds header information of the packet being processed |
| `transferPtr`       | `char *`              | Pointer to where the incoming data is being written |
| `remainToTransfer`  | `uint64_t`            | Amount of data (in bytes) left to receive for the current packet |
| `circularBufEnd`    | `char *`              | Marker to check if circular buffer has wrapped around |
| `circularBuf`       | `sample_t *`          | Buffer that stores complex IQ samples in a circular fashion |
| `channel_model`     | `channel_desc_t *`    | Pointer to the simulated wireless channel (e.g., fading, multipath) |

---

##### removeCirBuf()

```c
static void removeCirBuf(rfsimulator_state_t *bridge, buffer_t *buf)
{
  if (epoll_ctl(bridge->epollfd, EPOLL_CTL_DEL, buf->conn_sock, NULL) != 0) {
    LOG_E(HW, "epoll_ctl(EPOLL_CTL_DEL) failed\n");
  }
  close(buf->conn_sock);
  free(buf->circularBuf);
  // Fixme: no free_channel_desc_scm(bridge->buf[sock].channel_model) implemented
  // a lot of mem leaks
  // free(bridge->buf[sock].channel_model);
  memset(buf, 0, sizeof(buffer_t));
  buf->conn_sock = -1;
  nb_ue--;
}
```
---
This function is called when a connection (UE or gNB) is being closed. It performs the following:

- Unregisters the socket from epoll monitoring
- Closes the socket
- Frees the IQ circular buffer
- Clears the associated buffer state
---
| Line | Description |
|------|-------------|
| `epoll_ctl(...)` | Removes the socket from the epoll event loop |
| `close(buf->conn_sock);` | Closes the TCP socket for this buffer |
| `free(buf->circularBuf);` | Frees the memory allocated for the IQ sample buffer |
| `memset(...)` | Clears all fields of the `buffer_t` structure |
| `buf->conn_sock = -1;` | Marks the buffer as inactive |
| `nb_ue--;` | Decrements the global count of active UEs |

---
##### epoll_ctl()

`epoll_ctl()` is used to add, modify, or remove file descriptors (e.g., sockets) from an `epoll` instance. It controls what events are watched and how the kernel reacts to I/O readiness.

---
```c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

| Parameter | Description |
|-----------|-------------|
| `epfd`    | File descriptor of the epoll instance (returned by `epoll_create`) |
| `op`      | Operation to perform: add, modify, or delete |
| `fd`      | Target file descriptor to watch (e.g., a socket) |
| `event`   | Pointer to an `epoll_event` structure specifying the events to monitor (only for ADD and MOD) 

---
- **On success**: returns `0`
- **On failure**: returns `-1` and sets `errno`
---

| Operation        | Macro             | Description                             |
|------------------|-------------------|-----------------------------------------|
| Add to watchlist | `EPOLL_CTL_ADD`   | Adds a new file descriptor to be monitored |
| Modify settings  | `EPOLL_CTL_MOD`   | Changes the monitored events of an existing fd |
| Remove fd        | `EPOLL_CTL_DEL`   | Stops monitoring the specified file descriptor |

---

```c
epoll_ctl(bridge->epollfd, EPOLL_CTL_DEL, buf->conn_sock, NULL);
```
- Removes `buf->conn_sock` from the epoll instance `bridge->epollfd`.
- When using `EPOLL_CTL_DEL`, the `event` parameter **must be NULL**.

---

**close()**

Releases a **file descriptor** (e.g., for sockets, files, pipes).  
After calling `close()`, the file descriptor is no longer usable.
**Used for:**  
- Network sockets (`socket()`)
- File descriptors (`open()`, `accept()`)
- Pipes, device files

**free()**

Releases **heap memory** previously allocated by `malloc()`, `calloc()`, or `realloc()`.
**Used for:**  
- Dynamically allocated memory (like buffers, structs, arrays)
- Any pointer returned by `malloc()` family

**memset**

`memset()` is a standard C library function used to **fill a block of memory** with a specified byte value.

- Initialize buffers or arrays
- Clear `struct` variables to zero
- Reset heap memory before use

```c
void *memset(void *ptr, int value, size_t num);
```

| Parameter | Description |
|-----------|-------------|
| `ptr`     | Pointer to the memory block to fill |
| `value`   | Byte value to set (converted to `unsigned char`) |
| `num`     | Number of bytes to set |

### 🛑 `stopServer()` + `rfsimulator_end()` Integrated Flowchart

```text
┌────────────────────────────┐
│        stopServer()        │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ rfsimulator_state_t *t =   │
│   (rfsimulator_state_t*)   │
│   device->priv;            │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ DevAssert(t != NULL);      │
│ Ensure t is not NULL       │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ Close listening socket     │
│ close(t->listen_sock);     │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ Call rfsimulator_end(t);   │
└────────────┬───────────────┘
             │
             ▼
    ╔════════════════════════════════╗
    ║      rfsimulator_end() Begins   ║
    ╚════════════════════════════════╝
             │
             ▼
┌────────────────────────────┐
│ for i = 0 to MAX_FD_RFSIMU │
│ Loop through all buffers   │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ if (b->conn_sock >= 0)     │
│ → Call removeCirBuf()      │
└────────────┬───────────────┘
             │
             ▼
    ╔════════════════════════════════╗
    ║        removeCirBuf() Begins    ║
    ╚════════════════════════════════╝
             │
             ▼
┌────────────────────────────┐
│ Remove fd from epoll       │
│ epoll_ctl(..., DEL, ...)   │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ Close the socket            │
│ close(buf->conn_sock);      │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ Free circular buffer memory│
│ free(buf->circularBuf);    │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ Clear all buffer fields     │
│ memset(buf, 0, ...);        │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ Mark conn_sock = -1        │
│ Decrement nb_ue            │
└────────────┴───────────────┘
             ▲
             │ (next buffer)
             │
             ▼
┌────────────────────────────┐
│ After all buffers processed│
│ Close epoll fd             │
│ close(s->epollfd);         │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ Free the simulator state   │
│ free(s);                   │
└────────────┬───────────────┘
             │
             ▼
    ╔════════════════════════════════╗
    ║      rfsimulator_end() Ends     ║
    ╚════════════════════════════════╝
             │
             ▼
┌────────────────────────────┐
│    stopServer() Ends        │
└────────────────────────────┘
```


