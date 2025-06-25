-  [Rfsimulator](#Rfsimulator)
    -  [simulator.c](#simulatorc)
    -  [device_init](#device_init)
    -  [rfsimulator_readconfig()](#rfsimulator_readconfig)
    -  [Set openair0_device](#set-the-function-pointers-and-members-in-the-openair0_device-structure)
       -  [startServer()](#startServer)
       -  [startClient()](#startClient)
       -  [stopServer()](#stopServer)
       -  [rfsimulator_write()](#rfsimulator_write)
       -  [rfsimulator_read()](#rfsimulator_read)
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

### rfsimulator_write()

Wrapper Function to Send IQ Samples to RF Simulator

```c
static int rfsimulator_write(openair0_device *device,
                             openair0_timestamp timestamp,
                             void **samplesVoid,
                             int nsamps,
                             int nbAnt,
                             int flags) {
  timestamp -= device->openair0_cfg->command_line_sample_advance;

  return rfsimulator_write_internal(device->priv,
                                    timestamp,
                                    samplesVoid,
                                    nsamps,
                                    nbAnt,
                                    flags,
                                    false);  // false = use lock
}
```
---

This function is a **wrapper** around the internal write logic used in the OAI RF simulator. It handles:

1. **Timestamp Adjustment:**
   - `timestamp -= command_line_sample_advance`  
     Ensures samples are transmitted earlier to compensate for system delays.

2. **Internal Write Call:**
   - Calls `rfsimulator_write_internal()` to actually process and store the samples.

3. **Thread Safety:**
   - The final argument is `false`, meaning the write operation uses a lock for thread safety.

---
| Parameter       | Type                | Description                                  |
|----------------|---------------------|----------------------------------------------|
| `device`        | `openair0_device*`   | RF device structure                          |
| `timestamp`     | `openair0_timestamp` | Timestamp at which samples should be sent    |
| `samplesVoid`   | `void **`            | Pointers to sample data per antenna          |
| `nsamps`        | `int`                | Number of samples per antenna                |
| `nbAnt`         | `int`                | Number of antennas (e.g., 1T, 2T, etc.)       |
| `flags`         | `int`                | Control flags (usually unused/reserved)      |
| `false`         | `bool`               | `false` means the function will acquire locks|

---

- This function is called by the PHY layer or device transmission pipeline in the simulator.
- It helps **inject IQ samples** into the RF simulation buffer, maintaining timing alignment using `sample_advance`.

---
#### rfsimulator_write_internal()

```c
for (int i = 0; i < MAX_FD_RFSIMU; i++) {
  buffer_t *b = &t->buf[i];

  if (b->conn_sock >= 0) {
    samplesBlockHeader_t header = {nsamps, nbAnt, timestamp};
    fullwrite(b->conn_sock, &header, sizeof(header), t);

    sample_t tmpSamples[nsamps][nbAnt];

    if (nbAnt == 1) {
      if (b->conn_sock >= 0) {
        fullwrite(b->conn_sock, samplesVoid[0], sampleToByte(nsamps, nbAnt), t);
      }
    } else {
      for (int a = 0; a < nbAnt; a++) {
        sample_t *in = (sample_t *)samplesVoid[a];
        for (int s = 0; s < nsamps; s++)
          tmpSamples[s][a] = in[s];
      }

      if (b->conn_sock >= 0) {
        fullwrite(b->conn_sock, (void *)tmpSamples, sampleToByte(nsamps, nbAnt), t);
      }
    }
  }
}
```

---
**Outer Loop**

```c
for (int i = 0; i < MAX_FD_RFSIMU; i++)
```
- Scans through all possible RF socket connections
- `MAX_FD_RFSIMU` is typically 250 (maximum supported connections)

```c
buffer_t *b = &t->buf[i];
if (b->conn_sock >= 0)
```
- Skips if the socket is unused or closed (`conn_sock == -1`)

---

**Send Block Header**
```c
samplesBlockHeader_t header = {nsamps, nbAnt, timestamp};
fullwrite(b->conn_sock, &header, sizeof(header), t);
```
- Sends metadata before actual IQ data
  - `nsamps`: number of samples per antenna
  - `nbAnt`: number of antennas
  - `timestamp`: time offset for synchronization

---

**Single-Antenna Path**

```c
if (nbAnt == 1) {
  fullwrite(b->conn_sock, samplesVoid[0], sampleToByte(nsamps, nbAnt), t);
}
```
- If only 1 antenna: transmit its sample buffer directly
- `samplesVoid[0]` is a `void*` to the first antenna's samples

---
**Multi-Antenna Path**

```c
for (int a = 0; a < nbAnt; a++) {
  sample_t *in = (sample_t *)samplesVoid[a];
  for (int s = 0; s < nsamps; s++)
    tmpSamples[s][a] = in[s];
}
```
- Outer loop `a`: iterates each antenna
- Inner loop `s`: reorders per-antenna data into `[sample][antenna]` layout
- Needed because socket transmission expects data grouped per sample time, not per antenna

```c
fullwrite(b->conn_sock, (void *)tmpSamples, sampleToByte(nsamps, nbAnt), t);
```
- Transmits the interleaved multi-antenna samples over the socket

---

**`fullwrite(sock, ptr, size, t)`**
- Ensures full write (no partial writes)
- Handles multiple calls to `write()` until `size` bytes are written

**`sampleToByte(nsamps, nbAnt)`**
- Calculates size in bytes: `sizeof(sample_t) * nsamps * nbAnt`
- Determines total IQ data payload length

---

| Level | Scope                | Purpose                                      |
|-------|----------------------|----------------------------------------------|
| Outer | All socket buffers   | Send to each connected UE/gNB                |
| Mid   | All antennas         | Extract antenna-specific sample streams      |
| Inner | All samples per ant. | Interleave for `[time][antenna]` data layout |

---

```c
if (t->lastWroteTS != 0 && fabs((double)t->lastWroteTS - timestamp) > (double)CirSize)
  LOG_W(HW, "Discontinuous TX gap too large Tx:%lu, %lu\n", t->lastWroteTS, timestamp);

if (t->lastWroteTS > timestamp)
  LOG_W(HW, "Not supported to send Tx out of order %lu, %lu\n", t->lastWroteTS, timestamp);

if ((flags != TX_BURST_START) && (flags != TX_BURST_START_AND_END) && (t->lastWroteTS < timestamp))
  LOG_W(HW,
        "Gap in writing to USRP: last written %lu, now %lu, gap %lu\n",
        t->lastWroteTS,
        timestamp,
        timestamp - t->lastWroteTS);

t->lastWroteTS = timestamp + nsamps;

if (!alreadyLocked)
  pthread_mutex_unlock(&Sockmutex);

LOG_D(HW,
      "Sent %d samples at time: %ld->%ld, energy in first antenna: %d\n",
      nsamps,
      timestamp,
      timestamp + nsamps,
      signal_energy(samplesVoid[0], nsamps));

return nsamps;
```

---

**Discontinuity Warning**

```c
if (t->lastWroteTS != 0 && fabs(...) > CirSize)
```
- If this is **not the first write**, and the time gap since the last write is **larger than the circular buffer** size, log a warning.
- Indicates a possible transmission gap or simulator misalignment.

---

**Out-of-Order Warning**

```c
if (t->lastWroteTS > timestamp)
```
- If the current sample’s timestamp is **before** the last one written, log a warning.
- Transmission must be strictly time-ordered. This condition is **not supported**.

---

**Gap Warning (Non-Burst Transmission)**

```c
if ((flags != TX_BURST_START) &&
    (flags != TX_BURST_START_AND_END) &&
    (t->lastWroteTS < timestamp))
```
- If this write is **not the start of a burst** and there's a time gap (missing timestamps), warn about a discontinuity.
- Helps detect partial drops or skipped samples.

---

**Update Timestamp**

```c
t->lastWroteTS = timestamp + nsamps;
```
- Record the new **ending timestamp** of this block of samples.
- Used in future checks to detect gaps or disorder.

---

**Debug Output**

```c
LOG_D(HW,
  "Sent %d samples at time: %ld->%ld, energy in first antenna: %d\n", ...);
```
- Print transmission summary for debugging:
  - Total number of samples
  - Time range
  - Signal energy of antenna 0 (helpful for detecting silence or transmission issues)

---

| Check Type       | Condition                                        | Purpose                        |
|------------------|--------------------------------------------------|--------------------------------|
| Time Gap Check   | `abs(lastWroteTS - timestamp) > CirSize`        | Detect excessive discontinuity |
| Out-of-Order     | `lastWroteTS > timestamp`                        | Block backward transmission    |
| Mid-burst Gaps   | `timestamp > lastWroteTS` without burst flags    | Warn about skipped blocks      |
| Timestamp Update | `lastWroteTS = timestamp + nsamps`              | Prepare for next transmission  |
| Unlocking        | `if (!alreadyLocked)`                           | Thread-safe access             |
| Debug Logging    | `LOG_D(...)`                                     | Print TX status and signal info |


# 📈 `rfsimulator_write()` → `rfsimulator_write_internal()` Flowchart

```text
┌────────────────────────────────────────────┐
│        rfsimulator_write()                 │
│  - Called by upper OAI PHY layer           │
│  - Inputs: device, timestamp, samples, etc.│
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ Adjust timestamp                           │
│ timestamp -= device->cfg->sample_advance   │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ Call rfsimulator_write_internal()          │
│ Pass:                                       │
│  - device->priv → rfsimulator_state_t *t   │
│  - adjusted timestamp                      │
│  - samplesVoid, nsamps, nbAnt, flags       │
│  - alreadyLocked = false                   │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ rfsimulator_write_internal()               │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ If not alreadyLocked → lock Sockmutex      │
│ pthread_mutex_lock(&Sockmutex)             │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ Print debug info: number of samples, time  │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ LOOP: for each connection buffer (250 max) │
│   buffer_t *b = &t->buf[i];                │
│   if (b->conn_sock >= 0) {                 │
│     → Send header                           │
│     → Send samples                          │
│   }                                         │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌──────────── Sample Send Logic ─────────────┐
│ IF nbAnt == 1                              │
│   → Send samplesVoid[0] directly           │
│ ELSE                                       │
│   → For each antenna a:                    │
│      for (s = 0; s < nsamps; s++)          │
│        tmpSamples[s][a] = in[s];           │
│   → Send tmpSamples                        │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ Check for gaps or out-of-order timestamps: │
│  - too large jump → LOG_W                  │
│  - out of order   → LOG_W                  │
│  - missing burst  → LOG_W                  │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ Update lastWroteTS = timestamp + nsamps    │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ If not alreadyLocked → unlock Sockmutex    │
│ pthread_mutex_unlock(&Sockmutex)           │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ Log signal energy for samplesVoid[0]       │
│ Return nsamps                              │
└────────────────────────────────────────────┘
```

---

## 📘 Key Functions Called

| Function                      | Purpose                                      |
|------------------------------|----------------------------------------------|
| `rfsimulator_write()`        | Entry point from PHY layer                   |
| `rfsimulator_write_internal()` | Main data transmission to all buffers       |
| `pthread_mutex_lock()`       | Thread-safety for socket writes              |
| `fullwrite()`                | Write all bytes over socket, with retries    |
| `sampleToByte()`             | Compute total byte size of IQ samples        |
| `signal_energy()`            | Calculate energy of first antenna's signal   |
| `pthread_mutex_unlock()`     | Unlock shared resources                      |

---

**`rfsimulator_write()` → `rfsimulator_write_internal()` Full Flowchart**

```text
┌────────────────────────────────────────────┐
│        rfsimulator_write()                 │
│  - Called by upper OAI PHY layer           │
│  - Inputs: device, timestamp, samples, etc.│
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ Adjust timestamp                           │
│ timestamp -= device->cfg->sample_advance   │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ Call rfsimulator_write_internal()          │
│ Pass:                                       │
│  - device->priv → rfsimulator_state_t *t   │
│  - adjusted timestamp                      │
│  - samplesVoid, nsamps, nbAnt, flags       │
│  - alreadyLocked = false                   │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ rfsimulator_write_internal()               │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ If not alreadyLocked → lock Sockmutex      │
│ pthread_mutex_lock(&Sockmutex)             │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ Print debug info: number of samples, time  │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ LOOP: for each connection buffer (250 max) │
│   buffer_t *b = &t->buf[i];                │
│   if (b->conn_sock >= 0) {                 │
│     → Send header                           │
│     → Send samples                          │
│   }                                         │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌──────────── Sample Send Logic ─────────────┐
│ IF nbAnt == 1                              │
│   → Send samplesVoid[0] directly           │
│ ELSE                                       │
│   → For each antenna a:                    │
│      for (s = 0; s < nsamps; s++)          │
│        tmpSamples[s][a] = in[s];           │
│   → Send tmpSamples                        │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ Check for gaps or out-of-order timestamps: │
│  - too large jump → LOG_W                  │
│  - out of order   → LOG_W                  │
│  - missing burst  → LOG_W                  │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ Update lastWroteTS = timestamp + nsamps    │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ If not alreadyLocked → unlock Sockmutex    │
│ pthread_mutex_unlock(&Sockmutex)           │
└──────────────────────┬─────────────────────┘
                       │
                       ▼
┌────────────────────────────────────────────┐
│ Log signal energy for samplesVoid[0]       │
│ Return nsamps                              │
└────────────────────────────────────────────┘
```

---

| Function                      | Purpose                                      |
|------------------------------|----------------------------------------------|
| `rfsimulator_write()`        | Entry point from PHY layer                   |
| `rfsimulator_write_internal()` | Main data transmission to all buffers       |
| `pthread_mutex_lock()`       | Thread-safety for socket writes              |
| `fullwrite()`                | Write all bytes over socket, with retries    |
| `sampleToByte()`             | Compute total byte size of IQ samples        |
| `signal_energy()`            | Calculate energy of first antenna's signal   |
| `pthread_mutex_unlock()`     | Unlock shared resources                      |

---

### rfsimulator_read()

```c
rfsimulator_state_t *t = device->priv;
LOG_D(HW,
      "Enter rfsimulator_read, expect %d samples, will release at TS: %ld, nbAnt %d\n",
      nsamps, t->nextRxTstamp + nsamps, nbAnt);
```
---

| 項目 | 說明 |
|------|------|
| `rfsimulator_state_t *t = device->priv;` | 取得 RFSIM 模擬器內部狀態。<br>此指標 `t` 包含目前所有 buffer、時間戳、連線 socket 狀態與模擬角色。 |
| `LOG_D(...)` | 使用 OAI 的除錯訊息系統印出本次讀取的資訊：<br>• 預期接收樣本數：`nsamps`<br>• 釋出時間戳：`t->nextRxTstamp + nsamps`<br>• 天線數量：`nbAnt` |

---
這段程式碼的目的是在 `rfsimulator_read()` 開始時：

- 取得模擬器內部狀態（context）
- 印出即將接收樣本的基本資訊
- 幫助後續除錯與同步分析使用

```c
int first_sock;

for (first_sock = 0; first_sock < MAX_FD_RFSIMU; first_sock++)
  if (t->buf[first_sock].circularBuf != NULL)
    break;
```
---

| 項目 | 說明 |
|------|------|
| `first_sock` | 用來記錄第一個有連線的 socket 編號（index）。初始化為 0。 |
| `t->buf[i].circularBuf` | 這是指向每個 socket 對應的接收緩衝區（circular buffer），若不為 `NULL` 表示該 socket 已成功連線並初始化。 |
| `break;` | 一旦找到有一個 socket 已連線，就跳出迴圈，不再繼續檢查。 |

---

這段程式碼的目的是：
> 在最多 `MAX_FD_RFSIMU`（通常為 250）個模擬 socket 中，檢查是否至少有一個對端（如 UE 或 gNB）已連上模擬器並建立 buffer。如果沒有任何一個連上，後續會進入「產生空樣本」的處理流程。


```c
if (first_sock == MAX_FD_RFSIMU) {
  if (t->nextRxTstamp == 0)
    LOG_I(HW, "No connected device, generating void samples...\n");

  if (!flushInput(t, t->wait_timeout, nsamps)) {
    for (int x = 0; x < nbAnt; x++)
      memset(samplesVoid[x], 0, sampleToByte(nsamps, 1));

    t->nextRxTstamp += nsamps;
```
---

| 程式段落 | 解釋 |
|----------|------|
| `if (first_sock == MAX_FD_RFSIMU)` | 若先前的搜尋沒有找到任何有效連線（即沒有一個 socket 建立 buffer），就表示目前沒有裝置（如 UE）連線到模擬器。 |
| `if (t->nextRxTstamp == 0)` | 代表這是模擬器的第一次接收，還未收到任何有效資料，因此印出提示：將產生空樣本。 |
| `flushInput(...)` | 嘗試從 socket 接收資料，給予一個 timeout（等候時間）。如果沒有收到樣本，會返回 false。 |
| `if (!flushInput(...))` | 如果在等待後仍沒有資料，系統會進入「產生空樣本」的處理流程。 |
| `for (...) memset(...)` | 對所有天線的接收樣本緩衝區，用 0 填滿。這代表產生的是「無訊號樣本」（void/空白樣本），也就是模擬器偽造一段靜默期。 |
| `t->nextRxTstamp += nsamps;` | 將下一個接收樣本的時間戳推進 `nsamps`，確保時間連續往前，即使沒收到資料也不會中斷時間軸。 |

---

當模擬器偵測到「完全沒有連線裝置（如 UE）」時：

> 它會先等待看看是否在 timeout 時間內能接收到任何樣本；若沒有，則自動為每根天線生成一段空白樣本（全為 0），以維持模擬時序與資料一致性。

這樣可避免模擬器因缺少實際資料而中斷收發流程。

#### flushInput()

```c
static bool flushInput(rfsimulator_state_t *t, int timeout, int nsamps_for_initial) {
  struct epoll_event events[MAX_FD_RFSIMU] = {{0}};
  int nfds = epoll_wait(t->epollfd, events, MAX_FD_RFSIMU, timeout);

  if (nfds == -1) {
    if (errno == EINTR || errno == EAGAIN) {
      return false;
    } else {
      LOG_W(HW, "epoll_wait() failed, errno(%d)\n", errno);
      return false;
    }
  }
```
---

| 區塊 | 功能說明 |
|------|-----------|
| `epoll_event events[]` | 準備一個陣列來接收 `epoll_wait()` 回傳的事件。每個事件代表一個有資料可讀的 socket。 |
| `epoll_wait(...)` | 在模擬器內部的 epoll 機制上等待 socket 事件（例如：有資料可讀）。<br>參數：<br>• `t->epollfd` 是 epoll 描述子<br>• `MAX_FD_RFSIMU` 是最大可監控的 socket 數量<br>• `timeout` 是最大等待時間（毫秒） |
| `nfds == -1` | 如果等待過程出現錯誤（如中斷、暫時無效、系統錯誤），就視情況印出警告並返回 false。 |
| `errno == EINTR` 或 `EAGAIN` | 這些是非致命錯誤，代表暫時中斷或無資料，因此直接結束並回傳 false。 |
| 其他錯誤 | 若非上述錯誤，則列印錯誤資訊並回傳 false。 |

---
> 利用 `epoll_wait()` 監控所有模擬器 socket 的輸入事件，確認是否有對端（UE/gNB）傳送資料進來；若出現錯誤或超時，則提前結束這次等待，避免程式卡住。
---

#### epoll_wait()

---
```c
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

---


| 參數       | 說明                                                                 |
|------------|----------------------------------------------------------------------|
| `epfd`     | 由 `epoll_create()` 或 `epoll_create1()` 建立的 epoll 描述子。      |
| `events`   | 指向 `epoll_event` 結構陣列的指標，用來儲存觸發的事件。             |
| `maxevents`| 可處理的最大事件數（即 `events[]` 陣列的大小）。                   |
| `timeout`  | 等待時間（毫秒）：<br>• `0` 表示立即返回<br>• `-1` 表示無限等待<br>• `>0` 指定等待的最大毫秒數 |

---

| 回傳值     | 說明                                                                 |
|------------|----------------------------------------------------------------------|
| `>= 0`     | 表示有幾個事件發生，`events[]` 中前 `n` 筆是有效的事件              |
| `0`        | 表示 timeout 到期，但沒有事件發生                                   |
| `-1`       | 發生錯誤，可查看 `errno` 得知原因                                    |

---

| 錯誤碼     | 說明                                                                 |
|------------|----------------------------------------------------------------------|
| `EINTR`    | 被 signal 中斷，屬於非致命錯誤，可忽略並重試                        |
| `EAGAIN`   | 無事件可處理，可能是非阻塞模式下的暫時無資料                        |
| 其他錯誤   | 如 `epfd` 無效、記憶體不足等，需進一步調查                          |

---

### 🧠 用途總結

> `epoll_wait()` 是 Linux 高效能的 I/O 事件監控機制，能監視大量檔案描述符（socket）是否可讀寫或有錯誤，適合用於伺服器或模擬器（如 RFSIM）中處理多個 client 裝置連線狀態。

---

```c
for (int nbEv = 0; nbEv < nfds; ++nbEv) {
  buffer_t *b = events[nbEv].data.ptr;

  if (events[nbEv].events & EPOLLIN && b == NULL) {
    int conn_sock;
    conn_sock = accept(t->listen_sock, NULL, NULL);
    if (conn_sock == -1) {
      LOG_E(HW, "accept() failed, errno(%d)\n", errno);
      return false;
    }
```
---
| 程式行 | 解釋 |
|--------|------|
| `for (int nbEv = 0; ...)` | 處理 `epoll_wait()` 回傳的每個事件，最多 `nfds` 個。 |
| `buffer_t *b = events[nbEv].data.ptr;` | 取出該事件對應的使用者資料指標。若為 NULL，表示是新連線請求。 |
| `if (events[nbEv].events & EPOLLIN && b == NULL)` | 如果該事件是「有資料可讀」且來自監聽 socket，表示有裝置（如 UE）想連線。 |
| `accept(t->listen_sock, NULL, NULL);` | 接受新連線，產生一個新的 socket 描述子用於與該裝置通訊。 |
| `if (conn_sock == -1)` | 若連線建立失敗，印出錯誤並返回 `false`。 |

---

- **監聽新的 UE/gNB 裝置嘗試連線到模擬器（server）**
- **使用 `accept()` 建立一個新的 socket**
- 若成功，後續會對這個新連線做初始化與加進 epoll 監控列表

**accept()**
```c
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

---

### 🔍 參數說明

| 參數 | 說明 |
|------|------|
| `sockfd` | 伺服器端用來監聽的 socket（通常經過 `socket()`、`bind()`、`listen()` 建立） |
| `addr` | （可選）指向 `sockaddr` 結構的指標，用來取得對方 client 的位址資訊 |
| `addrlen` | 指向 `socklen_t` 的變數，用來表示 `addr` 的結構長度，並在函式結束後會寫回實際的長度 |

如果不需要對端位址資訊，`addr` 和 `addrlen` 可傳入 `NULL`。

---

### 🔁 回傳值

| 回傳值 | 說明 |
|--------|------|
| `>= 0` | 成功，回傳一個新的 socket 檔案描述符，用來與 client 溝通 |
| `-1`   | 失敗，需透過 `errno` 取得錯誤原因（如連線已中斷等） |

在 `flushInput()` 函式中：
- 使用 `epoll_wait()` 監聽 `listen_sock` 是否有新連線
- 若事件發生（EPOLLIN）且對應 `buffer_t == NULL`
- 就代表是新裝置（UE/gNB）嘗試連線
- 此時使用 `accept()` 建立新 socket，並初始化對應的接收緩衝區

```c
if (setblocking(conn_sock, notBlocking)) {
  return false;
}
buffer_t *new_buf = allocCirBuf(t, conn_sock);
if (new_buf == NULL) {
  return false;
}
LOG_I(HW, "A client connects, sending the current time\n");

c16_t v = {0};
nb_ue++;
void *samplesVoid[t->tx_num_channels];
```
---

| 程式行 | 說明 |
|--------|------|
| `setblocking(conn_sock, notBlocking)` | 將 `conn_sock` 設定為非阻塞模式，這樣之後使用 `read()` 或 `write()` 時不會被阻塞在呼叫中。若設定失敗，直接 return `false`。 |
| `allocCirBuf(t, conn_sock)` | 為這個新 socket 分配一個 `buffer_t` 結構並初始化對應的環形緩衝區（circular buffer）。若記憶體不足或連線數已滿，會回傳 `NULL`。 |
| `LOG_I(...)` | 記錄有一個 UE 成功連進模擬器（gNB 端）。 |
| `c16_t v = {0};` | 宣告一個空的複數樣本（16 位元整數），後續可能用於初始化傳輸樣本。 |
| `nb_ue++` | 將連線中的 UE 數量加一。這對於後續同步或計時有幫助。 |
| `void *samplesVoid[t->tx_num_channels];` | 建立一個指標陣列，為後續傳送樣本準備記憶體區塊（依照有幾個天線通道 `tx_num_channels` 來定義）。 |

---
1. 設定新 socket 為非阻塞模式
2. 配置 buffer，並將其納入模擬器的管理陣列中
3. 更新連線狀態與 UE 數量
4. 準備好樣本資料指標區塊以利後續傳輸

**連線後立即送出虛擬樣本 — 用於時間同步**
```c
for (int i = 0; i < t->tx_num_channels; i++)
    samplesVoid[i] = (void *)&v;

rfsimulator_write_internal(
    t,
    t->lastWroteTS > 1 ? t->lastWroteTS - 1 : 0,
    samplesVoid,
    1,
    t->tx_num_channels,
    1,
    false
);
```
---
| 步驟 | 說明 |
|------|------|
| 建立虛擬樣本 | 使用 `samplesVoid[]` 指向靜態樣本 `v`（通常為零值樣本） |
| 傳送 1 筆樣本 | 呼叫 `rfsimulator_write_internal()` 傳送 1 筆樣本給每個天線 |
| 設定時間戳 | 使用 `lastWroteTS - 1` 略早於目前時間，避免與未來樣本衝突 |
| flags = 1 | 表示傳送起始（`TX_BURST_START`） |
| alreadyLocked = false | 代表這裡還沒上鎖，會由內部加鎖處理 |

---

> **讓新連線的 client（如 UE）在連線後立即收到一筆樣本以進行時間同步。**
此動作避免了以下問題：
- UE 收不到第一筆樣本而卡住
- UE 無法與模擬器同步時間戳（`timestamp`）
- 保證連線後立刻進入穩定傳輸流程

---

```c
if (new_buf->channel_model)
    new_buf->channel_model->start_TS = t->lastWroteTS;
```

| 功能 | 說明 |
|------|------|
| 同步 channel 模型時間 | 如果該 socket 的 `buffer_t` 有使用通道模型（如 fading 模擬），就將其時間起點設為模擬器目前已發送的最後時間 `lastWroteTS` |

---

```c
if (events[nbEv].events & (EPOLLHUP | EPOLLERR | EPOLLRDHUP))
    socketError(t, b);
```

| 錯誤處理條件 | 行為 |
|---------------|------|
| `EPOLLHUP` | 對方斷開 |
| `EPOLLERR` | socket 發生錯誤 |
| `EPOLLRDHUP` | 對方關閉了寫入端（常見於 TCP close） |
| ➡️ | 呼叫 `socketError()` 處理錯誤並跳過這個事件 |

---

**socketError()**

```c
static void socketError(rfsimulator_state_t *bridge, buffer_t *buf)
```

---

| 行為 | 說明 |
|------|------|
| 檢查是否有效 socket | 若 `buf->conn_sock != -1`，表示這個 socket 有綁定且尚未關閉 |
| 輸出警告 | 印出 "Lost socket" 表示通訊已中斷 |
| 清除 buffer | 使用 `removeCirBuf()` 清除該 socket 對應的緩衝與資源 |
| Client 特殊處理 | 若角色是 `SIMU_ROLE_CLIENT`，發生錯誤會立刻 `exit(1)` 強制結束程式 |

---

```c
if (events[nbEv].events & (EPOLLHUP | EPOLLERR | EPOLLRDHUP)) {
    socketError(t, b);
    continue;
}
```
---

```c
if (b->circularBuf == NULL)
  LOG_E(...);
```

| 檢查目的 | 說明 |
|----------|------|
| 是否為有效的連線 | 若該 socket 沒有綁定 buffer，就略過該事件 |

---

```c
if (b->headerMode)
    blockSz = b->remainToTransfer;
else
    blockSz = (b->transferPtr + b->remainToTransfer <= b->circularBufEnd)
              ? b->remainToTransfer
              : b->circularBufEnd - b->transferPtr;
```

| 模式 | blockSz 計算方式 |
|------|-------------------|
| Header 模式 | 接收 header 剩餘大小 |
| 資料模式 | 依據 circular buffer 位置動態計算避免越界 |

---

```c
ssize_t sz = recv(b->conn_sock, b->transferPtr, blockSz, MSG_DONTWAIT);
```

| 函式 | 說明 |
|-------|------|
| `recv()` | 從 TCP socket 讀取資料 |
| `MSG_DONTWAIT` | 使用非阻塞模式，不會卡住主迴圈 |

---

```c
if (sz < 0) {
  if (errno != EAGAIN) {
    LOG_E(...);
  }
}
```

| 條件 | 說明 |
|------|------|
| `sz < 0` | 表示 `recv()` 發生錯誤 |
| `errno == EAGAIN` | 資料尚未就緒（非阻塞模式下常見），屬於正常現象 |
| 其他 errno | 代表真正的 socket 錯誤，需記錄錯誤訊息 |

---

```c
else if (sz == 0)
  continue;
```

| 條件 | 說明 |
|------|------|
| `sz == 0` | 表示對端 socket 正常關閉連線 |
| 行為 | 略過此事件，等待後續 epoll 檢查來清理資源 |




**判斷條件**
```c
if (b->headerMode == true && b->remainToTransfer == 0)
```
| 條件 | 意義 |
|------|------|
| `headerMode == true` | 正在處理 header |
| `remainToTransfer == 0` | header 資料已接收完畢 |

---
**切換到資料接收模式**
```c
b->headerMode = false;
```
---
**首次時間同步（只在 `t->nextRxTstamp == 0` 時進行）**
```c
t->nextRxTstamp = ...;
b->lastReceivedTS = ...;
```
| 變數 | 用途 |
|------|------|
| `t->nextRxTstamp` | UE 下一次應接收的時間戳，與 gNB 對齊 |
| `b->lastReceivedTS` | buffer 中目前最後接收到的資料時間戳 |

---
**忽略第一包資料，只做同步用**
```c
b->trashingPacket = true;
```
---
**設定 Channel 模型起始時間（如果有）**
```c
b->channel_model->start_TS = t->nextRxTstamp;
```
---

**判斷條件**

```c
else if (b->lastReceivedTS < b->th.timestamp)
```

| 比較欄位 | 用途 |
|----------|------|
| `b->lastReceivedTS` | buffer 目前最後一筆樣本的時間戳 |
| `b->th.timestamp` | 新接收封包的起始時間戳 |

→ 中間出現空檔，需要補零樣本

---
**天線數量**

```c
int nbAnt = b->th.nbAnt;
```
---
**Gap < CirSize：逐筆填 0**
```c
for (index = lastReceivedTS; index < timestamp; index++)
  for (each antenna a)
    circularBuf[(index * nbAnt + a) % CirSize] = 0;
```
| 目的 | 保持每個樣本與時間對應一致性 |
|------|------------------------------|
| `% CirSize` | 保證緩衝區環形不溢位 |

---
**Gap 過大：整個緩衝區歸零**
```c
memset(circularBuf, 0, total_size);
```
---
**更新最後時間戳**
```c
b->lastReceivedTS = b->th.timestamp;
```
---

**根據時間戳決定是否接收或丟棄封包資料**

**情況 1：timestamp 倒退且封包 size = 1**
```c
if (b->lastReceivedTS > b->th.timestamp && b->th.size == 1)
```
| 條件                             | 意義                       |
|----------------------------------|----------------------------|
| 新封包 timestamp 過舊            | 可能是 Rx/Tx 同步封包     |
| 封包大小為 1                     | 是特殊同步封包             |
| 動作：`b->trashingPacket = true` | 不寫入，僅作同步參考        |

---
**情況 2：timestamp 正常接續**
```c
else if (b->lastReceivedTS == b->th.timestamp)
```
- 正常狀態，允許寫入資料
- 無需特殊處理
---
**情況 3：timestamp 倒退但 size ≠ 1**
```c
else
```
| 條件                             | 意義                     |
|----------------------------------|--------------------------|
| 新封包 timestamp 過舊            | 非同步封包但順序錯誤     |
| 動作：`b->trashingPacket = true` | 丟棄封包，避免資料錯亂    |

---

**IQ 樣本資料接收邏輯**

---
**鎖定並檢查 Tx/Rx 差距**
```c
pthread_mutex_lock(&Sockmutex);
...
pthread_mutex_unlock(&Sockmutex);
```
- 保護多執行緒對 timestamp 的存取
- 若 Tx 與 Rx 差距超過 `CirSize`，輸出警告
---
**設定資料寫入位置與大小**
```c
b->transferPtr = (char *)&b->circularBuf[(b->lastReceivedTS * b->th.nbAnt) % CirSize];
b->remainToTransfer = sampleToByte(b->th.size, b->th.nbAnt);
```
| 欄位 | 說明 |
|------|------|
| `transferPtr` | 指向 IQ 緩衝區正確位置 |
| `remainToTransfer` | 還要接收幾個 byte |

---
**若已在讀 IQ 資料（不是 header）**
```c
if (b->headerMode == false)
```
- 處理樣本資料並更新 timestamp
---
**資料完整接收完畢時**

```c
if (b->remainToTransfer == 0)
```
| 動作 | 說明 |
|------|------|
| `headerMode = true` | 下次收 header |
| `transferPtr = &th` | 指向 header buffer |
| `remainToTransfer = sizeof(header)` | 準備收下個封包 |
| `trashingPacket = false` | 重置狀態 |

---

## 🔁 `flushInput()` 詳細流程圖

```text
┌───────────────────────────────────────────┐
│ flushInput(t, timeout, nsamps_initial)   │
└───────────────────────────────────────────┘
                   │
                   ▼
┌───────────────────────────────────────────┐
│ 使用 epoll_wait() 監聽所有 socket 事件     │
└───────────────────────────────────────────┘
                   │
                   ▼
          ┌──────────────────────┐
          │ nfds == -1 ?         │
          └──────────────────────┘
              │          │
       errno = EINTR     │ nfds > 0
     or errno = EAGAIN   ▼
        return false   開始處理事件
                         │
                         ▼
        ┌────────────────────────────────────┐
        │ 逐個處理 events[nbEv]              │
        └────────────────────────────────────┘
                         │
                         ▼
        ┌────────────────────────────────────┐
        │ 如果是新連線事件 (b == NULL)       │
        │ → accept(), 建立 conn_sock         │
        │ → 設定 non-blocking                │
        │ → 建立 circular buffer 結構       │
        │ → 加入 epoll 監聽                  │
        └────────────────────────────────────┘
                         │
                         ▼
        ┌────────────────────────────────────┐
        │ 如果是已連線的 socket 收資料       │
        │ → recv() 資料進 buffer             │
        │   - 如果還在收 header               │
        │       → 更新 transferPtr / remain  │
        │       → 完成則轉入 payload 模式     │
        │   - 如果是 payload                  │
        │       → 寫入 circular buffer       │
        │       → 更新 lastReceivedTS        │
        │       → 完成則回到 headerMode      │
        └────────────────────────────────────┘
                         │
                         ▼
          ┌────────────────────────────┐
          │ 所有 event 處理完畢        │
          └────────────────────────────┘
                         │
                         ▼
               return nfds > 0
```

