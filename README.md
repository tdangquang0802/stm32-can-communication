# STM32 CAN Communication — 2 Nodes (Normal Mode)

> Hệ thống giao tiếp CAN 2.0 trên 2 board STM32F103C8T6: một node gửi CAN frame, một node nhận và forward dữ liệu lên PC qua UART.

![Platform](https://img.shields.io/badge/Platform-STM32F103C8T6-blue)
![Language](https://img.shields.io/badge/Language-Embedded_C-brightgreen)
![Protocol](https://img.shields.io/badge/Protocol-CAN_2.0_Normal_Mode-orange)
![IDE](https://img.shields.io/badge/IDE-STM32CubeIDE-red)
![Status](https://img.shields.io/badge/Status-Complete-success)

---

## 📌 Tổng quan

Project phát triển tại **FPT Academy** (Jan 2026 – Apr 2026) — team project.

Xây dựng hệ thống CAN 2 node thực tế trên phần cứng STM32F103:
- **Node 1 (Transmitter)**: định kỳ gửi CAN frame lên bus
- **Node 2 (Receiver)**: nhận CAN frame → forward dữ liệu qua UART để Qt Dashboard trên PC đọc và hiển thị

---

## 🔌 Sơ đồ phần cứng

```
[ STM32F103 - Node 1 ]          [ STM32F103 - Node 2 ]
   CAN_TX (PB9) ─────────────────── CAN_RX (PB8)
   CAN_RX (PB8) ─────────────────── CAN_TX (PB9)
        │                                 │
   [MCP2551]                         [MCP2551]
        │                                 │
        └──────── CANH / CANL ────────────┘
                   (CAN Bus)

                                    UART TX (PA9) ──→ USB-TTL ──→ PC
                                    (Gửi data lên Qt Dashboard)
```

---

## 📁 Cấu trúc Project

Repo gồm 2 project STM32CubeIDE riêng biệt:

```
stm32-can-communication/
│
├── CAN_NormalMode_Node1/           ← Flash vào Board 1 (Transmitter)
│   ├── Core/
│   │   ├── Inc/
│   │   │   ├── main.h
│   │   │   └── it.h                # Interrupt handlers header
│   │   └── Src/
│   │       ├── main.c              # ★ Logic chính: CAN init, Tx loop
│   │       ├── it.c                # Interrupt handlers
│   │       ├── msp.c               # MCU peripheral init (HAL_MspInit)
│   │       ├── syscalls.c
│   │       └── sysmem.c
│   ├── Drivers/
│   │   ├── CMSIS/                  # ARM core headers
│   │   └── STM32F1xx_HAL_Driver/   # STM32 HAL library
│   ├── CAN_LoopBack.ioc            # CubeMX config file (clock, CAN, UART)
│   ├── STM32F103C8TX_FLASH.ld      # Linker script
│
├── CAN_NormalMode_Node2/           ← Flash vào Board 2 (Receiver)
│   ├── Core/
│   │   ├── Inc/
│   │   │   ├── main.h
│   │   │   └── it.h
│   │   └── Src/
│   │       ├── main.c              # ★ Logic chính: CAN Rx, UART forward
│   │       ├── it.c
│   │       ├── msp.c
│   │       ├── syscalls.c
│   │       └── sysmem.c
│   ├── Drivers/
│   ├── CAN_LoopBack.ioc
│   ├── STM32F103C8TX_FLASH.ld
│
└── README.md                       ← File này
```

---

## ⚙️ Cấu hình CAN (từ file .ioc)

| Thông số | Giá trị |
|---|---|
| MCU | STM32F103C8T6 (72 MHz) |
| CAN Mode | Normal Mode (2 nodes) |
| Baud Rate | 500 kbps |
| Frame Type | Standard Frame (11-bit ID) |
| CAN TX Pin | PB9 |
| CAN RX Pin | PB8 |
| UART (Node 2) | PA9 (TX), 115200 baud |
| CAN Transceiver | MCP2551 hoặc SN65HVD230 |

---

## 🔄 Luồng hoạt động

```
Node 1 (Transmitter)                    Node 2 (Receiver)
─────────────────────                   ──────────────────
main.c khởi tạo:                        main.c khởi tạo:
  • Clock (72MHz)                          • Clock
  • CAN peripheral                         • CAN peripheral
  • GPIO                                   • UART (115200)

Loop chính:                             Interrupt callback:
  HAL_CAN_AddTxMessage()                  HAL_CAN_RxFifo0MsgPendingCallback()
  → gửi frame mỗi 100ms                    → đọc CAN frame
  → ID: 0x103                               → extract Data[8]
  → Data: payload có ý nghĩa                → gửi qua UART → PC
                                            → Qt Dashboard nhận & hiển thị
```

---

## 🚀 Hướng dẫn Build & Flash

### Yêu cầu
- STM32CubeIDE 1.12+
- 2× STM32F103C8T6 (Blue Pill)
- 2× CAN Transceiver (MCP2551 / SN65HVD230)
- ST-Link V2 để flash
- USB-TTL để xem UART output của Node 2

### Các bước

```bash
# 1. Clone repo
git clone https://github.com/tdangquang0802/stm32-can-communication.git

# 2. Mở STM32CubeIDE
#    File → Import → Existing Projects into Workspace
#    Chọn folder CAN_NormalMode_Node1 → Finish
#    Lặp lại với CAN_NormalMode_Node2

# 3. Build từng project (Ctrl+B)

# 4. Flash Node 1 vào Board 1
#    Run → Debug → chọn target Node1

# 5. Flash Node 2 vào Board 2

# 6. Nối dây 2 board theo sơ đồ trên

# 7. Cắm USB-TTL vào UART TX của Node 2 → mở Serial Monitor (115200 baud)
#    Hoặc mở Qt Dashboard app để xem tín hiệu real-time
```

---

## 📊 Phân chia công việc

| Phần | Effort |
|---|---|
| Cấu hình peripheral: clock, CAN init, GPIO, UART (file .ioc + msp.c) | ~50% |
| Embedded C: CAN Tx/Rx handler, message logic trong main.c + it.c | ~30% |
| Debug & validation qua UART serial output | ~20% |

---

## 🔑 Kiến thức thu được

- CAN 2.0 Normal Mode: yêu cầu ít nhất 2 node trên bus (khác LoopBack)
- STM32 HAL CAN API: `HAL_CAN_Start`, `HAL_CAN_AddTxMessage`, `HAL_CAN_RxFifo0MsgPendingCallback`
- Thứ tự khởi tạo peripheral quan trọng: clock → GPIO → CAN → UART
- Filter configuration: phải set đúng filter bank mới nhận được frame
- UART làm debug channel nhẹ khi không có logic analyzer

---

## 🔗 Project liên quan

- **Qt CAN Dashboard** (nhận và hiển thị dữ liệu từ Node 2) → [github.com/tdangquang0802/qt-vehicle-can-dashboard](https://github.com/tdangquang0802/qt-vehicle-can-dashboard)

---

## 👤 Tác giả

**Dang Quang Trung** — Fresher Embedded / Automotive Software Developer  
📧 tdangquang0802@gmail.com

*FPT Academy — Automotive Software Development Program (2025–2027)*
