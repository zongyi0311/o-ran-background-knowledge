# `socket()`
## Function Declaration
```c
int socket(int domain, int type, int protocol);
```
---

### 1 `domain`

| Value                 | Description                            |
|----------------------|----------------------------------------|
| `AF_UNIX` / `AF_LOCAL`| Local (inter-process) communication    |
| `AF_INET`            | IPv4 Internet protocols                |
| `AF_INET6`           | IPv6 Internet protocols                |

---

### 2 `type`

| Value          | Protocol | Description                             |
|----------------|----------|-----------------------------------------|
| `SOCK_STREAM`  | TCP      | Stream-based, reliable connection       |
| `SOCK_DGRAM`   | UDP      | Datagram-based, connectionless          |

---
### 3 `protocol`

- Specifies a particular protocol to use with the socket.
- Usually set to `0`, letting the **kernel select the appropriate protocol** based on `domain` and `type`.

---
