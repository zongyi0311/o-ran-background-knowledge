# nr-softmodem.c — Function-by-Function Reference（函式逐一說明）

> Entry point and runtime orchestration for OAI NR softmodem.


## pdcp_run()

**Signature**: `void pdcp_run(const protocol_ctxt_t *const ctxt_pP)`

**Purpose**: Legacy stub (4G scheduler) — not used in NR path; aborts if called.  
**用途**：相容 4G 排程器的保留函式；NR 路徑不使用，呼叫即中止。

**Parameters / 參數**
- `ctxt_pP`: `const protocol_ctxt_t *const`

**Returns / 回傳**
- `void`

---

## clock_difftime()

**Signature**: `struct timespec clock_difftime(struct timespec start, struct timespec end)`

**Purpose**: Compute `end - start` as a timespec with normalized nsec.  
**用途**：計算兩時間點差值（奈秒位元正規化）。

**Parameters / 參數**
- `start`: `struct timespec`
- `end`: `struct timespec`

**Returns / 回傳**
- `struct timespec`

---

## print_difftimes()

**Signature**: `void print_difftimes(void)`

**Purpose**: Log the current min/max measured difftime (ns).  
**用途**：輸出目前量測到的最小/最大時間差（奈秒）。

**Parameters / 參數**
- (none)

**Returns / 回傳**
- `void`

---

## update_difftimes()

**Signature**: `void update_difftimes(struct timespec start, struct timespec end)`

**Purpose**: Update min/max difftime using `clock_difftime()` and optionally print.  
**用途**：以 `clock_difftime()` 更新最小/最大時間差，必要時列印。

**Parameters / 參數**
- `start`: `struct timespec`
- `end`: `struct timespec`

**Returns / 回傳**
- `void`

---

## build_rflocal()

**Signature**: `unsigned int build_rflocal(int txi, int txq, int rxi, int rxq)`

**Purpose**: Pack RF I/Q route selection into a bitfield word.  
**用途**：將 RF I/Q 路徑選擇打包成位元欄位。

**Parameters / 參數**
- `txi`: `int`
- `txq`: `int`
- `rxi`: `int`
- `rxq`: `int`

**Returns / 回傳**
- `unsigned int`

---

## build_rfdc()

**Signature**: `unsigned int build_rfdc(int dcoff_i_rxfe, int dcoff_q_rxfe)`

**Purpose**: Pack RXFE DC offset (I/Q) into a single word.  
**用途**：將接收前端 I/Q 的 DC 偏移量打包成整數。

**Parameters / 參數**
- `dcoff_i_rxfe`: `int`
- `dcoff_q_rxfe`: `int`

**Returns / 回傳**
- `unsigned int`

---

## exit_function()

**Signature**: `void exit_function(const char *file, const char *function, const int line, const char *s, const int assert)`

**Purpose**: Global exit path: set `oai_exit`, call RF/IF `trx_end_func`, abort or clean exit.  
**用途**：全域結束流程：設 `oai_exit`、呼叫 RF/IF 關閉函式，視情況中止或正常結束。

**Parameters / 參數**
- `*file`: `const char`
- `*function`: `const char`
- `line`: `const int`
- `*s`: `const char`
- `assert`: `const int`

**Returns / 回傳**
- `void`

---

## create_gNB_tasks()

**Signature**: `int create_gNB_tasks(ngran_node_t node_type, configmodule_interface_t *cfg)`

**Purpose**: Create/configure ITTI tasks and NR stacks (RRC/PDCP/RLC/MAC, NGAP/X2AP); start SCTP/NFAPI; register to AMF/X2 as needed.  
**用途**：建立並設定 ITTI 任務與 NR 協定堆疊（RRC/PDCP/RLC/MAC、NGAP/X2AP）；啟動 SCTP/NFAPI；依需求向 AMF/X2 註冊。

**Parameters / 參數**
- `node_type`: `ngran_node_t`
- `*cfg`: `configmodule_interface_t`

**Returns / 回傳**
- `int`

---

## get_options()

**Signature**: `void get_options(configmodule_interface_t *cfg)`

**Purpose**: Parse command-line options and set softmodem flags.  
**用途**：解析命令列參數並設定軟體數據機旗標。

**Parameters / 參數**
- `*cfg`: `configmodule_interface_t`

**Returns / 回傳**
- `void`

---

## wait_RUs()

**Signature**: `void wait_RUs(void)`

**Purpose**: Block until all RUs complete initialization/synchronization.  
**用途**：等待所有 RU 完成初始化/同步。

**Parameters / 參數**
- (none)

**Returns / 回傳**
- `void`

---

## wait_gNBs()

**Signature**: `void wait_gNBs(void)`

**Purpose**: Wait until every gNB instance is configured.  
**用途**：等待所有 gNB 實例完成設定。

**Parameters / 參數**
- (none)

**Returns / 回傳**
- `void`

---

## terminate_task()

**Signature**: `void terminate_task(task_id_t task_id, module_id_t mod_id)`

**Purpose**: Send ITTI `TERMINATE_MESSAGE` to a target task.  
**用途**：向指定 ITTI 任務送出結束訊息。

**Parameters / 參數**
- `task_id`: `task_id_t`
- `mod_id`: `module_id_t`

**Returns / 回傳**
- `void`

---

## stop_L1()

**Signature**: `int stop_L1(module_id_t gnb_id)`

**Purpose**: Stop NR L1 & RU/gNB devices; free PHY/RU resources; mark gNB unconfigured.  
**用途**：停止 NR L1 與 RU/gNB 裝置；釋放資源；將 gNB 標記未配置。

**Parameters / 參數**
- `gnb_id`: `module_id_t`

**Returns / 回傳**
- `int`

---

## start_L1L2()

**Signature**: `int start_L1L2(module_id_t gnb_id)`

**Purpose**: Start runtime: (re)apply MAC/serving cell config, init/start RUs, wait RUs, call `init_eNB_afterRU()`, release thread barrier.  
**用途**：啟動執行期：套用 MAC/小區設定、初始化並啟動 RU、等待 RU、呼叫 `init_eNB_afterRU()`，解除執行緒屏障。

**Data flow / 資料流向**：`MAC/RRC cfg + RU handles → running L1/L2 threads`

**Parameters / 參數**
- `gnb_id`: `module_id_t`

**Returns / 回傳**
- `int`

**Mini flow / 簡易流程**
1. Init scheduler response / apply MAC config
2. Init & start RUs; wait_RUs
3. init_eNB_afterRU(); release sync barrier

---

## wait_nfapi_init()

**Signature**: `void wait_nfapi_init(char *thread_name)`

**Purpose**: Wait on NFAPI sync condition until stack is ready.  
**用途**：等待 NFAPI 同步條件直到協定棧就緒。

**Parameters / 參數**
- `*thread_name`: `char`

**Returns / 回傳**
- `void`

---

## init_pdcp()

**Signature**: `void init_pdcp(void)`

**Purpose**: Initialize NR PDCP layer (not on DU nodes).  
**用途**：初始化 NR PDCP（DU 節點不執行）。

**Parameters / 參數**
- (none)

**Returns / 回傳**
- `void`

---

## initialize_agent()

**Signature**: `void initialize_agent(ngran_node_t node_type, e2_agent_args_t oai_args)`

**Purpose**: Initialize E2 (flexRIC) agent using PLMN/IDs from RRC/MAC; call init_agent_api().  
**用途**：以 RRC/MAC 的 PLMN/識別資訊初始化 E2（flexRIC）代理，呼叫 init_agent_api()。

**Parameters / 參數**
- `node_type`: `ngran_node_t`
- `oai_args`: `e2_agent_args_t`

**Returns / 回傳**
- `void`

---

## main()

**Signature**: `int main(int argc, char **argv)`

**Purpose**: Program entry: background services → load config → set signal handlers → parse options → init stacks/tasks → start time manager → wait config → start L1/L2 → shutdown clean.  
**用途**：主程式：啟動背景服務 → 載入設定 → 設定訊號處理 → 解析選項 → 初始化堆疊/任務 → 啟動時間管理 → 等待設定完成 → 啟動 L1/L2 → 優雅關閉。

**Data flow / 資料流向**：`CLI/CFG → init tasks/stacks → runtime start/stop`

**Parameters / 參數**
- `argc`: `int`
- `**argv`: `char`

**Returns / 回傳**
- `int`

**Mini flow / 簡易流程**
1. start_background_system / load_configmodule / set_softmodem_sighandler
2. get_options; allocate/configure global structures
3. create_gNB_tasks (if applicable)
4. time_manager_start; wait_gNBs / wait_RUs
5. start_L1L2 (+ wait_nfapi_init if needed)
6. on exit: stop_L1 / terminate_task / cleanup

---
