
# 1.2.2 The Bit

## 1. Computers Operate in Binary
- All computer data is stored and processed as **bits** (0 or 1).
- A **bit** (binary digit) is the **smallest unit of data**.

## 2. Bit = Two Physical States
- A bit is represented by **two discrete physical states**, e.g.:
  - Voltage levels
  - Magnetization directions
  - Light intensity
  - On/Off (e.g., light switch: 1 = On, 0 = Off)

## 3. Input/Output Devices Use Binary
- **Input devices** (keyboard, mouse, mic) → convert human actions to binary.
- **Output devices** (monitor, printer, speaker) → convert binary to human-readable form.

## 4. ASCII: Representing Text in Binary
- **ASCII** is a common binary code for characters.
- Each character = **8 bits** = **1 byte**.

  | Character | ASCII Binary |
  |-----------|--------------|
  | A         | 01000001     |
  | 9         | 00111001     |
  | #         | 00100011     |


#  1.2.3 Common Methods of Data Transmission

## 1. From Bits to Signals
- After data is converted to **binary bits**, it must be transformed into **signals** to travel over a network.
- A **signal** is a pattern (electrical, optical, or radio) that carries data from **source to destination**.

---

## 2. What is Network Media?
- **Media** is the physical path used for transmitting signals.
- Common types:
  - Copper wire (Ethernet cables)
  - Fiber-optic cables
  - Wireless (air-based, using radio/microwave)

---

## 3. Signal Types

| Type            | Medium Used         | Description                                      |
|-----------------|---------------------|--------------------------------------------------|
| Electrical      | Copper wires        | Data represented as **pulses of electricity**    |
| Optical         | Fiber-optic cables  | Data transmitted as **pulses of light**          |
| Wireless        | Air (no cables)     | Data transmitted as **radio/microwave/infrared** |

 A signal may be **converted multiple times** as it passes through different types of media.

---

## 4. Real-World Applications

- **Home/Small Business**:
  - Usually use **copper cables (Ethernet)** or **Wi-Fi (wireless)**

- **Large-scale Networks**:
  - Often use **fiber-optic cables** for **high-speed, long-distance** communication

---

# 1.3 Bandwidth and Throughput

---

## 1. 📡 What is Bandwidth?

- **Bandwidth** is the **theoretical maximum capacity** of a medium to carry data.
- Measured in **bits per second (bps)** – how many bits can be transmitted in one second.

### 🔢 Common Bandwidth Units:

| Unit                  | Abbreviation | Equivalent in bps  |
|-----------------------|--------------|--------------------|
| Bits per second       | bps          | Base unit (1 bps)  |
| Kilobits per second   | Kbps         | 1 Kbps =  10³      |
| Megabits per second   | Mbps         | 1 Mbps =  10⁶      |
| Gigabits per second   | Gbps         | 1 Gbps = 10⁹       |
| Terabits per second   | Tbps         | 1 Tbps = 10¹²      |

---

## 2.What is Throughput?

- **Throughput** is the **actual rate** at which data is successfully transferred over the network.
- Always **less than or equal to bandwidth** due to various real-world limitations.

### Factors that affect throughput:
- Amount of traffic on the network
- Type of data being transferred (e.g., video, files)
- **Latency** – delay caused by devices, distance, or network congestion
- Slowest link in a multi-segment path determines overall throughput

Even if most of the network is fast, **one slow segment** will reduce the total throughput.

---

## What is Latency?

- **Latency** is the total time it takes for data to travel from source to destination.
- Includes transmission, propagation, queuing, and processing delays.

---

## Summary: Bandwidth vs. Throughput

| Aspect        | Bandwidth                        | Throughput                       |
|---------------|----------------------------------|----------------------------------|
| Definition    | Maximum theoretical data rate    | Actual data transfer rate        |
| Units         | bps, Kbps, Mbps, Gbps, Tbps      | Same                             |
| Influenced by | Hardware, media, physics         | Traffic, latency, slowest link   |
| Measurement   | Device specs, link capacity      | Speed tests, real transfer logs  |





