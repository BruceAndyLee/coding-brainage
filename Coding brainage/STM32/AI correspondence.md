# Understanding STM32 embedded programming
_Exported on 15/02/2026 at 19:24:22 GMT+3 from Cursor (2.4.37)_

---

**User**

Hello, benevolent robot overlord!

In this folder there is code for three boards with stm32 on them that talk to each-other and enable the work of a synthesizer that is keeping track of pressure using adc, clicking the valves according to the gcode-protocol, reporting system status over rs485 to the host machine (currently, the host machine uses python's serial module and passes commands down to it from the rest API that talks to the outerworld).

I am colling upon you because I need an introduction with points of interest and tension outlined. I need to better understand the code base, the basic premise, approaches and restrictions of the embedded programming.

To be more specific:

1) What exactly does HAL provide for the c-code? what is the cognitive entrypoint to this system? Where does it's responsibility end and the application code start? using the examples from this code base, what would some of the functions look like if the weren't using HAL?

2) Can you tell me something about bootloaders for each of the boards? The size of the memory that those boot-systems have? how much of the boot process is actually controllable in general when programming STM? what sort of tasks does the bootloader solve, beside literally just loading the firmware into RAM? What tasks specifically does this codebase solve during boot? (I heard tell of ROM in the context of booting a microcontroller like stm or esp, what is that? what part does it play in these sorts of systems?)

3) Potentially I need to know how to talk to each of the boards from the host machine such that I can upload a newer revision of firmware on them, but, as you can see from the previous questions I really know next to nothing about this process. Are there partitions in the memory?

4) I am aware that the code in these dirs is compiled with some version of IAR, very recently I learnt that that means that some of the system-functions are declared and behave differently compared to if they were written for a GNU compiler. How much is it true? what would be different and how much work would that be to maintain both the gnu version of the project and the iar one? The goal here is to be able to build the source code inside a docker container to post artifacts on a product-delivery server.

5) what is the approach to doing things like talking to standard things like USB or rs485 over UART? what is the business with those System/ and Drivers/ folder?

Please, explore the code and provide an overview with motivated explaners and code-snippet examples.


---

# Firmware codebase overview

## 1) HAL: what it provides, entrypoint, and where it ends

**What HAL provides (in this project)**

- **Single entrypoint** for “all STM32” in app code: `#include "stm32f4xx_hal.h"` (in `main.h`). That pulls in device header, CMSIS, and the HAL modules you enable in `System/stm32f4xx_hal_conf.h` (RCC, GPIO, UART, TIM, SPI, RTC, etc.).
- **Init/clock**: `HAL_Init()`, `HAL_RCC_OscConfig()`, `HAL_RCC_ClockConfig()`, `__HAL_RCC_*_CLK_ENABLE()`.
- **GPIO**: `HAL_GPIO_Init()`, and your code uses HAL for init only; toggling is done with direct register macros (`GPIO_LOW` / `GPIO_HIGH` in `iologic.h`).
- **NVIC**: `HAL_NVIC_SetPriority()`, `HAL_NVIC_EnableIRQ()`, `HAL_NVIC_SystemReset()`.
- **SysTick**: `HAL_IncTick()` in `SysTick_Handler()` (in `System/stm32f4xx_it.c`).
- **Time**: `HAL_Delay()`, `HAL_GetTick()` (and RTC if used).


---

## 4) IAR vs GNU (and dual-toolchain / Docker)

**What’s IAR-specific in this repo**

- **Linker**: IAR uses `.icf` (e.g. `define symbol __ICFEDIT_region_ROM_start__ = ...`). GNU uses `.ld` (e.g. `MEMORY { FLASH : ORIGIN = 0x08004000, ... }`). You’d need equivalent `.ld` scripts per board/config (flash vs boot).
- **Startup**: IAR assembly uses `EXTERN __iar_program_start`, `BLX` to `SystemInit` then `__iar_program_start`. GNU uses `Reset_Handler` and usually `_start` in a C runtime (e.g. newlib) that does init and calls `main`. So a different startup file (or same logic in GNU asm).
- **Intrinsics / system functions**:  
  - `__enable_interrupt()` / `__disable_interrupt()`: in IAR from `<intrinsics.h>`. In GCC/ARM use `__enable_irq()` / `__disable_irq()` from CMSIS (they’re in `cmsis_iccarm.h` for IAR and `cmsis_gcc.h` for GCC).  
  - `__DSB()`, `__ISB()`: CMSIS already abstracts these; both compilers have them.  
  - `__no_init` and `@ address`: IAR extensions. GNU equivalent:  
    - `__attribute__((section(".noinit")))` and a linker section at a fixed address, e.g.  
      `__attribute__((section(".bootkey"))) __attribute__((used)) volatile unsigned long BootKey1;`  
    and in `.ld`: `.bootkey 0x20000000 : { *(.bootkey) }` (and same for BootKey2).
- **`#pragma location = 0x20000000`** (`VectorTable`): under GNU, put that in a section and place it at 0x20000000 in the linker script.

So: **behaviour is the same**, but **syntax and sections** differ. The amount of work to keep both IAR and GNU buildable:

- **Low**: if you limit IAR-isms to a few files (startup, linker script, BootKey/vector placement, and `intrinsics.h` vs CMSIS). You can add a small “port” layer (e.g. `port_intrinsics.h` that does `#ifdef __ICCARM__` vs `#else` for GCC) and one `.ld` per board/config.
- **Medium**: if you have many `#pragma` or IAR-specific attributes scattered around, or rely on IAR’s runtime layout (e.g. heap). In this codebase, the IAR-specific bits are concentrated (startup, one `__no_init @` per board, `intrinsics.h`, `.icf`), so maintaining both is manageable.
- **Docker**: For GNU you’d use `arm-none-eabi-gcc` (and optionally CMake/Make) in the container; IAR is licensed and typically not run in Docker, so “build in Docker” usually means **GNU only**; IAR remains for local/IDE builds unless you have IAR’s headless/licensed setup.

---

## 5) USB / RS485 / UART and the System/ vs Drivers/ split

**How USB and RS485 are done here**

- **RS485**: Physical layer is **UART + DE (driver enable) GPIO**. The code uses:
  - **HAL** for: RCC (USART1 clock), PCLK for baud rate, GPIO init (TX, RX, DE), NVIC for USART and TIM3.
  - **Registers** for: baud rate (`USART1->BRR = pclk/baud`), control (`CR1/CR2/CR3`), and all RX/TX in the IRQ (`SR`, `DR`, and DE toggling via `GPIO_LOW`/`GPIO_HIGH`). So “talking to standard things” = HAL for one-time setup, registers for the critical path and framing (header, length, CRC, T35 timeout with TIM3).
- **USB**: Only **USB power enable/disable** via GPIO (no USB device stack). you may use `HAL_GPIO_Init` and `GPIO_LOW`/`GPIO_HIGH` macroses. or CDC/HID.

**System/ vs Drivers/**

- **Drivers/** = ST’s HAL and LL **libraries**: implementation of `HAL_GPIO_Init`, `HAL_RCC_*`, `HAL_UART_*`, low-level USART/SPI/TIM, etc. You include their headers and link their `.c` files; you don’t own this code.
- **System/** = **your** system integration for this product:
  - **startup_*.s**: vector table and first C entry (SystemInit → IAR runtime).
  - **system_stm32f4xx.c**: `SystemInit()` (minimal: FPU, optional VTOR/ExtMem), `SystemCoreClockUpdate()`.
  - **stm32f4xx_it.c**: interrupt handlers that the vector table points to (NMI, Fault, SysTick → `HAL_IncTick`).
  - **stm32f4xx_hal_msp.c**: HAL “MSP” (MCU Support Package) callbacks; often empty.
  - **stm32f4xx_hal_conf.h**: which HAL modules are enabled and system clock constants.

So: **Drivers/** = generic ST HAL/LL; **System/** = one-time, chip- and project-specific glue (startup, system init, IRQ table, HAL config). Application code (RS485, cmd, steppers, USB power, etc.) sits **above** both and uses HAL + direct registers as needed.

---

# Descending roadmap: from protocol to boot

## Corrections to your structure

1. **Boot is not “lowest” in the same stack**  
   Boot runs **first** on power-on, then **never** again until the next reset. The “stack” you described is the **application** that runs **after** boot. So: **Boot → then the layers below.** Boot is “before” the rest, not “under” it in the same process.

2. **“RTOS / VTOS”**  
   There is no VTOS here. If a board uses **FreeRTOS** (RTOS = Real-Time Operating System: tasks, queues, semaphores, priorities). Boards can also be **bare metal**: no OS, one main loop + one message queue + interrupts. So: one board has an RTOS; the other two do not.

3. **Order of layers**  
   A more accurate order (outer → inner) is:

   - **Layer 1 – Protocol** (RS485/PCCmd framing, commands, CRC)  
   - **Layer 2 – Virtual state & dispatch** (queues, switch on command, global state)  
   - **Layer 3 – Scheduling / activation** (what runs when: main loop vs tasks, timers, IRQs)  
   - **Layer 4 – OS (if any) + HAL + drivers** (FreeRTOS; HAL + GPIO/UART/TIM/ADC drivers everywhere)  
   - **Layer 5 – Boot** (runs once at power-on; decides “bootloader” vs “application” and can flash firmware)

Below we go through each layer **top-down** with **borders** and **examples** from this repo.


---

## Layer 3 – Timers, tick, interrupts, GPIO (scheduling foundation)

**What it is:** What **triggers** the dispatch layer: UART RX IRQ (byte/packet), TIM3 (inter-byte or inter-symbol timeout), SysTick (1 ms tick for HAL or FreeRTOS). Plus GPIO for RS485 DE, LEDs, etc.

**Where it lives:**  
- **Interrupt handlers:** e.g. `USART1_IRQHandler` (RXNE → push byte, when full packet → `MSG_Insert((MSG_RS485<<24)|MSG_RS485_MSG, 0)`); `TIM3_IRQHandler` (restart RX state machine on timeout).  
- **System tick:** `System/stm32f4xx_it.c` — `SysTick_Handler` → `HAL_IncTick()` (drives FreeRTOS scheduler if used).  
- **GPIO:** Application uses `HAL_GPIO_Init` for setup; toggling is often direct: `iologic.h` — `GPIO_HIGH(a,b)` / `GPIO_LOW(a,b)` → `BSRR`.

**Border:**  
- **This layer** ends at “we call HAL (or CMSIS) for: NVIC, GPIO init, UART init, timer init; we implement the IRQ handlers that move data/events into queues or call HAL_IncTick.”  
- **Below:** HAL and drivers that actually touch RCC, GPIO, USART, TIM registers.

**Sample:**

```c
// enable IRQ and set priority (application’s use of HAL)
HAL_NVIC_SetPriority(TIM3_IRQn, 2, 2);
HAL_NVIC_EnableIRQ(TIM3_IRQn);
HAL_NVIC_SetPriority(USARTsCfg.IRQn, 2, 2);
HAL_NVIC_EnableIRQ(USARTsCfg.IRQn);

// In USART1_IRQHandler: when full packet received
MSG_Insert ((MSG_RS485 << 24) | MSG_RS485_MSG, 0);
```

So: **“Application code ends at calling HAL for interrupts, GPIO, and timers”** — and the **sample of what’s available at that level** is: `HAL_NVIC_*`, `HAL_GPIO_Init`, `HAL_RCC_GetPCLK*`, and your own IRQ handlers that either push to a queue or call `HAL_IncTick()`.

---

## Layer 4 – “System”: RTOS + HAL + drivers

**What it is:**  
- **RTOS:** FreeRTOS — tasks, queues, semaphores, priorities, SysTick-driven scheduler.
- **HAL:** ST’s Hardware Abstraction Layer — `HAL_Init()`, `HAL_RCC_OscConfig` / `HAL_RCC_ClockConfig`, `HAL_GPIO_Init`, `HAL_FLASH_*`, `HAL_NVIC_*`, `HAL_IncTick()`, etc. Same idea on all three boards; different families (F0/F4).  
- **Drivers:** In `Drivers/` — implementation of HAL (and LL) for ADC, UART, TIM, SPI, Flash, etc. Your app (and System/) only **call** HAL; they don’t implement it.

**ADC:** Yes, there are HAL/LL ADC drivers (`stm32f*xx_hal_adc.c`, `stm32f*xx_ll_adc.c`). If a board uses ADC (e.g. pressure), application code would call `HAL_ADC_*` or LL helpers; the driver programs the ADC peripheral.

**Where it lives:**  
- **System/:** startup (vector table → `SystemInit` → `__iar_program_start`), `system_stm32f4xx.c` (SystemInit, SystemCoreClockUpdate), `stm32f4xx_it.c` (NMI/Fault/SysTick), `stm32f4xx_hal_conf.h` (which HAL modules are enabled).  
- **Drivers/:** ST’s HAL/LL .c files.  
- **RTOS**: `Middlewares/Third_Party/FreeRTOS/` — kernel, tasks, queue.c, timers.c, CMSIS-RTOS wrapper.

**Border:**  
- **Above:** Application and Layer 3 (IRQ handlers, main loop or tasks).  
- **This layer** provides: clock tree, GPIO/UART/TIM/ADC/Flash init and register access, SysTick, and (on RTOS powered stm-board) task switch and blocking.  
- **Below:** Only the CPU (Cortex-M) and the chip; no further software “layer.”

**Sample (what’s available at this level):**

```c
HAL_Init();
SystemClock_Config();   // HAL_RCC_OscConfig, HAL_RCC_ClockConfig
__HAL_RCC_GPIOA_CLK_ENABLE();
HAL_GPIO_Init(port, &GPIO_InitStruct);
HAL_NVIC_SetPriority(IRQn, preempt, sub);
HAL_NVIC_EnableIRQ(IRQn);
HAL_IncTick();          // in SysTick_Handler
HAL_GetTick(); HAL_Delay(ms);
HAL_FLASH_Unlock(); HAL_FLASH_Program(...); HAL_FLASH_Lock();
```

---

## Layer 5 – Boot (runs first on power-on)

**What it is:** Code that runs **once** after reset. It decides “stay in bootloader” (wait for XModem firmware upload) or “run application” (jump to 0x08004000). It is **outside** the normal “protocol → state → IRQ → HAL” flow because that flow is the **application**, which only runs if boot chooses to start it.

**What the bootloaders do (from the code you added):**

1. **Check BootKey** (same fixed RAM addresses as in firmware):  
   - If `(BootKey1 == BOOTKEY1 && BootKey2 == BOOTKEY2)` → **stay in bootloader** (host asked for firmware upload).  
   - Else → **try to run application.**

2. **Validate application:**  
   - `Flash_IsFirmwareOK()` reads a **header** at `HEADER_ADDRESS` (0x08004000): ProductCode, Len (in 256-byte units), CheckSum (CRC32 over payload).  
   - If valid → **run application.**  
   - If not (e.g. empty flash or bad CRC) → **stay in bootloader.**

3. **Run application:**  
   - `Flash_RunFirmware()`:  
     - Disable IRQs, reset buses (RCC->APB1RSTR etc.), deinit XModem/RS485 or UART, stop SysTick.  
     - Load application entry from `APPLICATION_ADDRESS + 4` (Reset_Handler in vector table at 0x08004200).  
     - Set MSP from `*APPLICATION_ADDRESS`, then call that entry.  
   - So the **application** runs at 0x08004000 (ROM start), vector table at 0x08004200; bootloader does **not** set VTOR in the code you have (application or boot may rely on default or linker placement).

4. **If staying in bootloader:**  
   - Clear BootKey, init RS485 or UART, init XModem, then `while(1) XModem_Loop()`.  
   - **XModem:** 128-byte blocks (SOH/EOT/ACK/NAK), CRC16; decrypting may be done with AES.  
   - **Flash:** Erase by page (e.g. `Flash_Erase(addr)`), write with `HAL_FLASH_Program(FLASH_TYPEPROGRAM_WORD, ...)`.  
   - **RTOS boot** can also use a button: if button not pressed and BootKey not set and firmware OK → run app; else UART + XModem.  
   - **RTOS boot** supports multiple BootKey2 values (MAIN, PORT1..4) for different targets.

**Memory layout (boot vs app):**

- **Bootloader:** Linked with “boot” .icf; lives at **0x08000000**, size **16 KB** (0x08000000–0x08003FFF).  
- **Application:** Linked with “boot” .icf; **0x08004000** (header at 0x08004000, vector table / Reset at 0x08004200).  
- **Partitions:** Boot = first 16 KB; app = from 0x08004000 to end of flash.

**Border:**  
- **Boot** is a separate binary and separate run: it uses the same HAL (and same UART/RS485 physical layer) but **different** protocol (XModem) and different “main” (no PCCmd/RS485 command loop, no G-code, no pressure/valves).  
- **Application** is what implements your “Layer 1–4”; boot only **starts** it or **replaces** it by flashing a new image.

**Sample (main boot flow):**

```c
// if already booted, just go to the app entrypoint
if (BootKey != BOOTKEY) {
  if (verify_flashed_firmware()) give_rains_to_flashed_firmware();
}
// otherwise go into the firmware-upload mode:
BootKey = 0;
// spin up RS485 over UART
RS485_Init(RS485_1, 115200);
// spin up the xmodem transfer protocol...
XModem_Init();
// ...and await transmission start
while (1) XModem_Loop();
```

---

## One-line boundaries (summary)

- **Layer 1 (Protocol):** Ends at “packet layout and command codes”; doesn’t touch UART/IRQ.  
- **Layer 2 (State & dispatch):** Ends at “call send / enable IRQ / set timer”; doesn’t implement IRQ or scheduler.  
- **Layer 3 (Timers/IRQ/GPIO):** Ends at “call HAL for NVIC/GPIO/timers and implement IRQ handlers that push to queue or HAL_IncTick.” **Application code ends here** when we talk about “calling HAL for interrupts, GPIO, and timers” — and the sample at that level is the HAL_NVIC_\*, HAL_GPIO_Init, and your USART/TIM handlers.  
- **Layer 4 (System):** HAL + drivers (+ FreeRTOS); provides clocks, peripherals, and (on RTOS powered board) task scheduling.  
- **Layer 5 (Boot):** Runs once at power-on; decides bootloader vs application; implements XModem and flash erase/write; then either jumps to app or waits for new firmware. It’s “outside” the application stack in time (runs first), not “under” it in the same run.


---

# Terminology reference

## Real-time OS (RTOS)

**What it is:** An OS where “correct” means not only correct result but also **before a deadline**. Used when late response is as bad as wrong response (control, safety, comms).

**Typical primitives that define it:**
- **Tasks** (threads) with **priorities** — higher priority runs first.
- **Preemption** — higher-priority task can interrupt lower (e.g. on timer tick or event).
- **Blocking** — task waits on **semaphore/queue** and doesn’t run until event; CPU can run other tasks.
- **Deterministic scheduling** — behaviour is predictable from priorities and timing (within limits of the model).

**How to tell it apart from other OS on MCUs:**
- **Bare metal / superloop:** One `main()` loop; no task switch; “OS” is your own queue + `WFI`.
- **RTOS (e.g. FreeRTOS):** You see `xTaskCreate`, `xQueueCreate`, `xSemaphoreTake`, `vTaskDelay`, `configMAX_PRIORITIES`; a **scheduler** decides which task runs; one task can block while others run.
- **Linux-style OS on MCU:** Full OS with processes, virtual memory, drivers in kernel — much heavier; you’d see “Linux” or “Zephyr” etc., not just a small RTOS kernel.

So: **RTOS = small kernel with tasks, priorities, preemption, and blocking primitives (queues, semaphores)**; “real-time” means deadlines matter.

---

## CRC (Cyclic Redundancy Check)

**What it is:** A short value computed from a block of data; used to **detect** (not correct) bit errors in transmission or storage.

**Why it exists:** Noise, bad wiring, marginal power can flip bits. CRC catches most random and burst errors so the receiver can reject bad packets (and optionally ask for retransmission, as in XModem).

**How it’s applied:** Sender: `crc = crc16_ccitt(data, len)` (or similar), append 2 bytes to packet. Receiver: recompute CRC over same range; if result is 0 (or matches), packet is accepted; else wrong CRC → discard / NAK.

**Versions and differences:**
- **CRC-16-CCITT** (used in this repo): polynomial 0x1021, initial value often 0xFFFF; good for serial links; 16 bits.
- **CRC-32:** 32 bits; used in flash header (e.g. `Flash_IsFirmwareOK`); stronger, more computation.
- **CRC-8:** 1 byte; simpler, weaker.
- Other variants differ in **polynomial**, **initial value**, **final XOR**, **reflection** (bit order). Same idea: deterministic function over bytes → fixed-length check value.

**In this codebase:** RS485/PCCmd use **CRC16** on the packet; bootloader flash header uses **CRC32** for firmware integrity.

---

## XModem

**What it is:** An old **file-transfer** protocol over a **serial link** (e.g. UART/RS485). It gives **block-level** delivery guarantee (each 128-byte block is ACK’d or NAK’d), but it is **not** TCP: no IP, no streams, no congestion control, no dynamic MTU.

**Rough analogy:** “TCP-like” only in the sense of “reliable delivery of a sequence of blocks over one link”; it’s much simpler and link-local.

**Headers you see:**
- **SOH (0x01):** Start Of Header — “next 128-byte block follows”.
- **EOT (0x04):** End Of Transmission — “no more blocks”.
- **ACK (0x06):** Acknowledge — “block received and accepted”.
- **NAK (0x15):** Not Acknowledge — “resend block” (wrong CRC or sequence).
- **C (0x43):** Receiver sends this to say “I’m ready; use CRC-16” (not old checksum).
- **CAN (0x18):** Cancel transfer.

**Packet size:** Fixed **128 bytes** per block in classic XModem. No runtime negotiation of packet size or MTU like in TCP.

**Retransmission:** Sender sends block; if receiver sends NAK (or nothing before timeout), sender resends same block. So yes: **retransmission by block**, driven by NAK/timeout, not by a sliding window like TCP.

**In this repo:** Bootloader uses XModem over RS485/UART to receive firmware image; host sends SOH+block or EOT; device replies ACK/NAK and writes to flash on ACK.

---

## UART IRQ

**What it is:** **IRQ = Interrupt Request.** A hardware mechanism: when something happens (e.g. “UART received a byte”), the CPU **stops** what it’s doing, runs a short **Interrupt Service Routine (ISR)**, then returns. So the CPU doesn’t have to poll “any byte yet?” in a loop.

**UART IRQ:** When the UART hardware has a new byte in its receive register (or other events like TX empty), it **asserts an IRQ line**. The CPU’s **NVIC** (see below) delivers that to the vector for e.g. `USART1_IRQHandler`. So “UART IRQ” = “interrupt caused by the UART peripheral”.

**Does it “tell the OS” to call the consumer?** Not the OS itself. The **ISR** (your code in `USART1_IRQHandler`) runs immediately. In this firmware:
- **Bare metal:** ISR pushes byte into buffer and, when a full packet is received, calls `MSG_Insert(...)` so the **main loop** will later call `RS485_Msg` (the “consumer”).
- **With FreeRTOS:** ISR might push bytes and then `xSemaphoreGiveFromISR` or `xQueueSendFromISR`, so the **task** that blocked on that semaphore/queue is marked ready and will run when the scheduler allows — that task is the “consumer”.

So: UART IRQ = “hardware says: data/event ready”; your ISR **notifies** the rest of the system (queue/semaphore), and the “consumer” is whatever runs when that queue/semaphore is processed (main loop or a task).

---

## TIM3 timeout and `__asm("WFI")`

**TIM3:** A hardware **timer** (counter). It’s clocked (e.g. from PCLK with a prescaler); when the counter reaches the **ARR** value, it overflows and can generate an **interrupt** (TIM3_IRQHandler).

**TIM3 “timeout” (in this code):** Used as an **inter-byte timeout** on the UART: “if no new byte arrives for ~10 ms, assume the packet is complete (or broken) and reset the RX state machine.” So TIM3 is started on each received byte; if no byte for 10 ms, TIM3 fires → IRQ → e.g. `RS485_Restart()` or `Uart_Restart()`.

**`__asm("WFI")`:** **W**ait **F**or **I**nterrupt. A CPU instruction: “stop executing until an interrupt (or event) occurs.” Saves power and avoids busy-loop. When any enabled IRQ fires, CPU wakes, runs the ISR, then returns to the instruction after WFI. So in `MSG_Loop`, when the queue is empty you do `WFI`; when USART or TIM3 IRQ fires and inserts a message, execution continues and the loop drains the queue. So WFI = “sleep until something happens.”

---

## `Queue.rd++; Queue.rd &= MSG_QUEUE_MASK`

**What it is:** Moving the **read index** of a **ring buffer** (circular queue) one step and keeping it in range.

- **Queue.rd** = index of the next message to read.
- **Queue.rd++** = “we consumed one slot; next read will be from the next slot.”
- **Queue.rd &= MSG_QUEUE_MASK** = wrap-around. If `MSG_QUEUE_SIZE` is 128, then `MSG_QUEUE_MASK` is 127. So `rd` goes 0,1,…,126,127 and then 128&127=0, 129&127=1, … So the queue is **circular**: after the last slot you go back to 0.

**Why the mask:** So you don’t need `if (rd >= SIZE) rd = 0`; one `&=` does the wrap as long as size is power of two. Same idea for `Queue.wr` when inserting.

---

## `ID >> 24` and why it’s switched

**What it is:** In this design, **ID** is a 32-bit value: **high byte = message source/type**, **low 24 bits = subtype or data**.

- **ID >> 24** = take the **top 8 bits** (e.g. 1 for MSG_RS485).
- **Switch on that:** One level of dispatch: “which subsystem produced this message?” — RS485, or (hypothetically) USB, ADC, etc.

So:
- **ID >> 24** = “who/what kind of message” (e.g. `MSG_RS485`).
- **ID & 0xFFFFFF** (or similar) = “which sub-event within that” (e.g. `MSG_RS485_MSG`).

The **switch** is the **dispatcher**: one place that routes every message from the queue to the right handler (e.g. `RS485_Msg(ID, Data)`).

---

## SysTick — who sets it, how, and what it influences

**What it is:** A **24-bit system timer** inside the Cortex-M core. It counts down from a load value to zero, then can generate an interrupt. Often used as the **system tick** (e.g. 1 ms period).

**Who sets it:** `HAL_Init()` (indirectly) configures it: typically `SysTick_Config(SystemCoreClock / 1000)` so it fires every 1 ms. So **HAL** sets it at startup based on **SystemCoreClock** (which comes from your `SystemClock_Config()` / RCC).

**What is influenced:**
- **HAL_GetTick()** / **HAL_Delay()** — HAL’s tick counter is incremented in **SysTick_Handler** (via `HAL_IncTick()`). So any code that uses `HAL_Delay` or timeouts depends on SysTick.
- **FreeRTOS:** The scheduler uses the same tick (or a timer tied to it) for **task switching** and **vTaskDelay** / timeouts. So SysTick (or an equivalent tick source) drives “how often the scheduler runs” and “how time is measured” for tasks.

So: **SysTick = system heartbeat**; HAL and the RTOS depend on it for time and scheduling.

---

## NVIC (Nested Vectored Interrupt Controller)

**What it is:** The **interrupt controller** in ARM Cortex-M. It decides **which** interrupt is pending, **priority**, and **who gets to run** when several are pending.

**How it’s used:**
- **Enable/disable** a specific IRQ: `HAL_NVIC_EnableIRQ(USART1_IRQn)`, `HAL_NVIC_DisableIRQ(...)`.
- **Set priority:** `HAL_NVIC_SetPriority(USART1_IRQn, preempt, sub)` — so when USART1 and TIM3 fire, the one with higher priority (lower number) is serviced first (or can preempt).
- **Vector table** (see below) tells the CPU “when USART1_IRQ fires, run this function.”

**Why:** So you can have many peripherals (UART, timers, ADC, etc.) all generating interrupts; NVIC + vector table route each to the right ISR and enforce priorities (and in FreeRTOS, “max syscall priority” so some IRQs never call RTOS APIs).

---

## PSC and ARR in TIM3

**TIM3** is a timer: it has a **counter** that goes up (or down) every clock tick.

- **PSC (Prescaler):** Divides the timer clock. If bus clock is 42 MHz and PSC = 41, timer clock = 1 MHz. So **PSC** sets “how fast the counter ticks.”
- **ARR (Auto-Reload Register):** The period. When the counter reaches ARR (in up-counting mode), it resets to 0 and can generate an **update event** (and interrupt). So **ARR** sets “how many ticks per period.”

**Example setting up timer for RS485 network:**
`TIM3->PSC = (pclk/1000000) - 1` → 1 MHz tick.  
`TIM3->ARR = 10000` → overflow every 10 000 ticks = **10 ms**.  
in this setup TIM3 IRQ = “10 ms passed since last byte” → inter-byte timeout for the UART RX state machine.

---

## HAL_NVIC_, HAL_GPIO_Init, HAL_RCC_GetPCLK — what, how, why

- **HAL_NVIC_SetPriority(IRQn, preempt, sub):** Set NVIC priority for that IRQ (smaller number = higher priority). Used so critical ISRs run before less critical ones.
- **HAL_NVIC_EnableIRQ(IRQn):** Allow that peripheral’s IRQ to reach the CPU. Without this, the UART/TIM interrupt never runs.
- **HAL_GPIO_Init(port, &init):** Configure pins: mode (input/output/alternate), speed, pull, and for alternate function (e.g. UART TX/RX). So “this pin is USART1_TX” etc.
- **HAL_RCC_GetPCLK1Freq() / GetPCLK2Freq():** Return the clock frequency of the **APB1** or **APB2** bus. UART baud rate and timer PSC are derived from these (e.g. `BRR = pclk/baud`). So they’re used to **configure** peripherals that are clocked from those buses.

**Why:** RCC = Reset and Clock Control; HAL wraps the register bashing for clocks and GPIO so the app doesn’t touch RCC/GPIO registers directly (though in this repo some code still does for hot paths).

---

## CMSIS (Cortex Microcontroller Software Interface Standard)

**What it is:** ARM’s **common API** for Cortex-M: core registers, NVIC, SysTick, intrinsics (`__DSB`, `__enable_irq`), and sometimes RTOS and DSP. Chip vendors provide a **CMSIS layer** that includes:
- **Device headers** (e.g. `stm32f4xx.h` — register definitions).
- **Startup** and **system** code (e.g. `system_stm32f4xx.c` — SystemInit, SystemCoreClockUpdate).
- **Core headers** (e.g. `core_cm4.h`, `cmsis_compiler.h` — core and compiler-specific intrinsics).

**In this repo:** You see `cmsis_iccarm.h` (IAR), `cmsis_gcc.h` (GCC): they define `__enable_irq`, `__DSB`, etc. So **CMSIS = the “standard” boundary** between your C code and the core (and often the vendor’s HAL sits on top of the device part of CMSIS).

---

## Buses — what they are and what “resetting” them means

**Buses:** Internal **data/address paths** in the chip. On STM32 you have:
- **AHB** (Advanced High-performance Bus) — core, flash, RAM, DMA.
- **APB1, APB2** (Advanced Peripheral Bus) — most peripherals (UART on APB1 or APB2, TIM2–7 on APB1, etc.). RCC feeds clocks to these.

So “buses” = the interconnect and clock domains that connect the CPU to memory and peripherals.

**“Reset the buses”:** In `Flash_BusesReset()` (bootloader before jumping to app) you see:
- `RCC->APB1RSTR = 0xFFFFFFFFUL;` then `0`;
- same for **APB2RSTR**, **AHBRSTR**.

Those are **peripheral reset** registers: writing 1 then 0 **resets** all peripherals on that bus to a clean state. So when the bootloader is about to jump to the application, it resets every peripheral so the **application** starts with UART, TIM, etc. in default state, not left in the state the bootloader had configured. It’s a clean handover.

---

## Vector table

**What it is:** A **table in memory** where each entry is the **address of an exception or IRQ handler**. The CPU uses it like: “exception number 15 happened → load word at table_base + 15\*4 → jump there.”

**Typical layout:**
- Index 0: initial **SP** (stack pointer).
- Index 1: **Reset_Handler** (first code run after reset).
- Index 2: NMI_Handler.
- Index 3: HardFault_Handler.
- …
- Index 15: SysTick_Handler.
- Then **external** IRQs: USART1_IRQHandler, TIM3_IRQHandler, etc.

**Who sets it:** The **linker** places the table (e.g. section `.intvec`) at a fixed address. On reset, the CPU fetches the first two words from the **vector table base** (often 0x08000000 or 0x08004000 for the app) to get SP and Reset_Handler. So the vector table is **the bridge** from “hardware exception number” to “your C function.” VTOR (Vector Table Offset Register) in the core can relocate this table (e.g. to RAM or to app at 0x08004000) so the CPU knows where to look after the bootloader jumps to the app.

---

You can use this list as a reference while re-reading the firmware; the names should start to map to these concepts instead of feeling like white noise.


---

## 1. HAL_NVIC_\*, HAL_GPIO_Init, HAL_RCC_GetPCLK — what they are, how they’re used, why

### HAL_NVIC_*

**What:** ST’s HAL wrappers around the **NVIC** (Nested Vectored Interrupt Controller) registers on Cortex‑M.

**Typical functions:**
- **HAL_NVIC_EnableIRQ(IRQn)** — turn on one interrupt line (e.g. `USART1_IRQn`, `TIM3_IRQn`). Until this is called, that peripheral’s IRQ never reaches the CPU.
- **HAL_NVIC_DisableIRQ(IRQn)** — turn it off again (e.g. before jumping to application in the bootloader).
- **HAL_NVIC_SetPriority(IRQn, PreemptPriority, SubPriority)** — set the priority of that IRQ. Lower numeric priority = higher importance; when several IRQs are pending, the one with highest priority runs first (and can preempt a lower‑priority ISR).

**How they’re used:**  
In init code (e.g. `RS485_Init`, `Uart_Init`): after enabling the peripheral clock and configuring UART/TIM, you call `HAL_NVIC_SetPriority(...)` and `HAL_NVIC_EnableIRQ(...)` so the CPU will actually run your `USART1_IRQHandler` / `TIM3_IRQHandler` when the hardware fires.

**Why:**  
So you don’t have to write to NVIC registers (e.g. `NVIC->ISER`, `NVIC->IP[]`) by hand; HAL hides the exact register layout and priority encoding. Also, on FreeRTOS you use a specific “max syscall priority” so some IRQs are above the kernel and never call RTOS APIs — that’s still done via these NVIC priorities.

---

### HAL_GPIO_Init

**What:** ST HAL function that configures one or more GPIO pins on a port (mode, speed, pull, alternate function).

**Parameters (conceptually):**  
Port (e.g. `GPIOA`), and a struct with: **Pin** (which pins), **Mode** (input / output / alternate / analog), **Pull** (none / up / down), **Speed**, and **Alternate** (which peripheral function, e.g. USART1 TX/RX).

**How it’s used:**  
You fill a `GPIO_InitTypeDef`, call `HAL_GPIO_Init(GPIOA, &GPIO_InitStruct)`, and the HAL writes the right bits in the GPIO’s MODER, OSPEEDR, PUPDR, AFR (alternate function) registers so that pin behaves as you want (e.g. “PA9 = USART1_TX”).

**Why:**  
So you don’t have to remember the exact bit fields for each STM32 family; one call sets everything for that pin. In this repo you use it for UART TX/RX/DE, USB enable/fault, etc.; toggling is often done with direct BSRR writes (`GPIO_HIGH`/`GPIO_LOW`) for speed.

---

### HAL_RCC_GetPCLK1Freq() / HAL_RCC_GetPCLK2Freq()

**What:** HAL helpers that return the current frequency of the **APB1** or **APB2** bus in Hz.

**How they’re used:**  
Peripherals on APB1/APB2 are clocked from these buses. You need the bus frequency to set:
- **UART baud rate:** `BRR = PCLK / baud`
- **Timer prescaler:** e.g. `TIM3->PSC = (HAL_RCC_GetPCLK1Freq() / 1000000) - 1` to get a 1 MHz timer clock.

So you call `HAL_RCC_GetPCLK1Freq()` or `GetPCLK2Freq()` during init and use the value to compute BRR, PSC, etc.

**Why:**  
PCLK depends on the main clock tree (HSI/HSE, PLL, AHB/APB dividers). HAL already knows that tree from `HAL_RCC_ClockConfig()`, so these functions give you the right number instead of you re-deriving it from RCC registers. If you changed the clock config, PCLK would change and your baud/timer math would still be correct as long as it’s based on these functions.

---

## 2. CMSIS — what it is, what it’s for, what depends on it

**What:** **Cortex Microcontroller Software Interface Standard** — an ARM-defined set of APIs and conventions for Cortex‑M cores so that middleware and applications can be written in a core/vendor-neutral way.

**Typical pieces:**
- **Core headers:** e.g. `core_cm4.h` — core registers (NVIC, SysTick, SCB), instruction wrappers (`__DSB`, `__ISB`, `__enable_irq`), and sometimes MPU/FPU.
- **Device headers:** e.g. `stm32f4xx.h` — vendor-supplied, but often CMSIS‑style: device type, IRQ numbers, register definitions.
- **Compiler abstraction:** e.g. `cmsis_compiler.h` (and in this repo `cmsis_iccarm.h` for IAR, `cmsis_gcc.h` for GCC) — so that `__enable_irq`, `__DSB`, etc. are defined per compiler.
- **Startup / system:** e.g. `system_stm32f4xx.c` — `SystemInit()`, `SystemCoreClockUpdate()`, `SystemCoreClock` — often provided under a CMSIS/device template.

**What it’s used for:**  
- Give you a **single, standard way** to talk to the core (NVIC, SysTick, barriers, interrupts) and to the device (registers, IRQ names).  
- Let **middleware** (e.g. RTOS, USB stack) depend on “CMSIS” instead of “STM32F4” so the same middleware can run on F0/F4/F7 with different device headers.  
- Hide **compiler differences** (IAR vs GCC) behind the same macro/function names.

**What changes depending on it:**  
- **Compiler:** You include `cmsis_iccarm.h` or `cmsis_gcc.h`; the same `__enable_irq()` in code compiles to the right intrinsic for that toolchain.  
- **Core:** Cortex‑M0 vs M4 have different features (e.g. M4 has FPU, more NVIC bits); the right core header is chosen by project.  
- **Device:** `stm32f4xx.h` vs `stm32f0xx.h` — different peripherals and addresses; your HAL and drivers are built on top of the device part of CMSIS.  
- **System:** `SystemInit()` / `SystemCoreClock` can be customised (e.g. FPU on, VTOR, external RAM) per chip/board; that’s the “system” part that sits under HAL.

So: **CMSIS = the standard “core + device” interface**; HAL and your app sit on top of it, and what changes with CMSIS is which core/device/compiler files you use and how `SystemInit`/clocks are set up.

---

## 3. Buses — what they are, what “resetting” them means, why and when that happens

**What “buses” are:**  
Inside the MCU, the CPU doesn’t talk to every block directly. There are **internal buses**: wires and logic that carry addresses, data, and control between the core, memories, and peripherals. On STM32 you typically have:

- **AHB (Advanced High-performance Bus):** Core, flash, RAM, DMA — the “main” bus.
- **APB1, APB2 (Advanced Peripheral Bus):** Most peripherals (UART, SPI, I2C, timers, etc.) sit on these; they’re usually clocked from **PCLK1** and **PCLK2**, which come from the same clock tree (RCC) after dividers.

So “buses” = both the **interconnect** (who talks to whom) and the **clock domains** (AHB, APB1, APB2) that drive those blocks.

**What “resetting the buses” means:**  
It doesn’t mean “reset the wires.” It means **resetting all the peripherals** attached to those buses. The RCC has **reset registers** (e.g. `RCC->APB1RSTR`, `RCC->APB2RSTR`, `RCC->AHBRSTR`). Writing a 1 to a bit resets that peripheral (flip-flops back to default); writing 0 releases it. In the bootloader you see something like:

- `RCC->APB1RSTR = 0xFFFFFFFFUL;` then `\= 0;`
- same for APB2 and AHB.

So “reset the buses” = **assert then release the RCC peripheral resets** for all peripherals on APB1, APB2, and AHB. Every UART, timer, GPIO block, etc. goes back to its power-on state.

**Why it’s done:**  
The bootloader has configured and used UART, timers, GPIO, etc. If it just jumped to the application without resetting peripherals, the application would see leftover configuration (e.g. UART still in the state the bootloader left it, timers still running). That can cause undefined behaviour or conflicts. Resetting all peripherals gives the application a **clean slate**: as if the chip had just come out of reset, so the application’s own `Uart_Init`, `RS485_Init`, etc. can configure everything from a known state.

**When it happens:**  
In this codebase it happens in the **bootloader**, inside `Flash_RunFirmware()` (or equivalent), **right before** disabling interrupts, stopping SysTick, setting MSP, and jumping to the application. So: **once per boot, when switching from bootloader to application**, to hand over hardware in a clean state.

---

## 4. Vector tables — what they are and how they’re used

**What a vector table is:**  
A **fixed layout in memory**: a list of addresses. The CPU is designed so that when an **exception** or **interrupt** occurs, it uses an **exception number** (e.g. 6 for usage fault, 15 for SysTick, 37 for USART1) to index into this table. It loads the word at `table_base + (exception_number × 4)` and jumps there. So the vector table is the **mapping from “exception number” to “first instruction of the handler.”**

**Typical layout (Cortex‑M):**  
- **Word 0:** Initial **stack pointer** (top of stack after reset).  
- **Word 1:** **Reset_Handler** — where the CPU jumps after reset (then that code usually calls `SystemInit` and `main`).  
- **Words 2–14:** Other **exceptions** (NMI, HardFault, MemManage, BusFault, UsageFault, reserved, SVCall, Debug, reserved, PendSV, SysTick).  
- **Words 15 onward:** **External interrupts** — one per IRQ (WWDG, PVD, …, USART1, TIM3, …), in the order defined by the vendor for that chip.

So the table is **read by the CPU hardware**: no software “dispatches” using it; the core loads the address and jumps. Your **startup** or **linker** places the table in a section (e.g. `.intvec`) and the linker script puts that section at a known address (e.g. start of flash).

**Who sets it:**  
- **Linker script** (e.g. `.icf`): places the section that contains the vector table at a fixed address (e.g. `0x08000000` for bootloader, `0x08004000` or `0x08004200` for application when using a bootloader).  
- **Startup code** (e.g. `startup_stm32f407xx.s`): defines the **symbol** that the linker puts in that section — the actual array of addresses (Reset_Handler, NMI_Handler, …, USART1_IRQHandler, TIM3_IRQHandler, etc.). So the **content** of the table is the list of handler addresses; the **place** is decided by the linker.

**VTOR (Vector Table Offset Register):**  
On Cortex‑M the vector table doesn’t have to be at 0. The **SCB->VTOR** register holds the **base address** of the table. On reset, the CPU uses a fixed address (often 0, which on STM32 is mapped to flash at 0x08000000). After the bootloader jumps to the application, the application’s vector table might be at 0x08004000 or 0x08004200; if so, something (often the bootloader or early application code) should set **VTOR** to that address so that when USART1 or TIM3 fires, the CPU fetches the handler address from the **application’s** table, not the bootloader’s. In this repo, VTOR is sometimes left at default (or commented out in SystemInit), so the exact behaviour depends on where the table is linked and whether the bootloader sets VTOR before jumping.

**Why it matters:**  
Without the vector table, the CPU wouldn’t know where to jump for reset, faults, or UART/TIM interrupts. It’s the **single contract** between hardware (exception number) and software (your C/asm handlers). Changing the table (e.g. adding a new IRQ handler) means adding an entry in the startup file and ensuring the linker keeps the table contiguous and at the right address; changing **where** the table lives (e.g. for a bootloader) means setting VTOR and linking the table at that address.

---

You can treat these as four standalone reference blocks: HAL NVIC/GPIO/RCC, CMSIS, buses and reset, and vector tables.

