
[rifsim simulator.](#rifsim-simulator.c)

##Foundation Understanding RF simulator 
RFSIM is a simulator designed to replace a physical RF board, allowing OAI to perform communication tests without requiring wireless hardware. It can simulate simple channels such as AWGN. RFSIM is not bound by real-time hardware sampling rates—its execution speed depends on CPU performance and can be faster or slower than real time.

Architecture
load_lib(載入 RFSIM 的 shared library)->

1
1
32
32
3
3232

323
5
42
12
1




85
56

4

5
65
##rifsim simulator.c
