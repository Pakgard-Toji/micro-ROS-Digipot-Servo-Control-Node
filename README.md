# 🧠 micro-ROS Node for Digital Potentiometer + Servo + Direction GPIOs

A tiny **micro-ROS** node (Arduino-compatible) that subscribes to **`/final_cmd_vel`** (`geometry_msgs/msg/Twist`) and controls:

* **Digital Potentiometer** via **SPI** (e.g., MCP41xxx)
* **Servo motor** via `writeMicroseconds()`
* **Three digital GPIO lines** (OUT\_PIN0/1/2) for direction logic

The node maps **`linear.x`** to hardware actions and gently ramps the servo pulse toward neutral when commands stop (soft-stop).

---

## ✨ Features

* Subscribe to **`/final_cmd_vel`** (`geometry_msgs/msg/Twist`)
* Use **`linear.x`** to control:

  * **Servo** pulse width (speed/angle)
  * **Digital Potentiometer** wiper (0–255)
  * **Direction GPIOs** (OUT\_PIN0/1/2)
* **Idle soft-stop**: when no new messages arrive, servo pulse ramps smoothly toward neutral
* **Serial debug**

---

## 🧰 Hardware

* Arduino-compatible board (tested with **ESP32** + **Arduino** core; works with classic AVR too)
* **MCP41xxx** digital potentiometer (SPI)
* **Servo** (5V)
* Three digital outputs for direction logic (OUT\_PIN0, OUT\_PIN1, OUT\_PIN2)
* Host machine (Jetson/NUC/PC) running **ROS 2** + **micro-ROS Agent**

> ⚠️ Power your servo with a stable 5V supply (common **GND** with MCU). Avoid drawing servo current from the MCU 5V pin if it cannot supply sufficient current.

---

## 🪛 Pinout / Wiring

### Default pins (change in code as needed)

```cpp
// SPI (MCP41xxx)
#define PIN_CS_DIGIPOT  5   // Chip Select for MCP41xxx
// MOSI/MISO/SCK use hardware SPI pins of your board

// Servo
#define PIN_SERVO       14

// Direction GPIOs
#define OUT_PIN0        25
#define OUT_PIN1        26
#define OUT_PIN2        27
```

### MCP41xxx to MCU (SPI)

| MCP41xxx     | MCU                                    |
| ------------ | -------------------------------------- |
| VDD          | +5V (or 3.3V per part)                 |
| VSS          | GND                                    |
| CS           | `PIN_CS_DIGIPOT`                       |
| SCK          | HW SPI SCK                             |
| SDI (MOSI)   | HW SPI MOSI                            |
| SDO (MISO)\* | HW SPI MISO (optional for this sketch) |
| P0A/P0W/P0B  | To your analog path/load               |

> \*This sketch only **writes** the wiper; MISO is optional.

---

## 🧩 ROS 2 Integration

The node **subscribes** to:

* **`/final_cmd_vel`** — `geometry_msgs/msg/Twist`

### Command mapping (`msg.linear.x`)

| `linear.x` | Action                                                                              |
| ---------: | ----------------------------------------------------------------------------------- |
|      `2.0` | `OUT_PIN0 = HIGH`, `OUT_PIN1 = LOW`, pulse `OUT_PIN2`, set DigiPot = **100**        |
|     `-1.0` | `OUT_PIN1 = HIGH`, `OUT_PIN0 = LOW`, `OUT_PIN2 = HIGH`, set DigiPot = **100**       |
|      other | All OUT pins **LOW**, DigiPot = **0**, servo pulse ramps toward neutral (soft-stop) |

---

## ⚙️ Build & Flash

### 1) Arduino IDE

1. Install **Arduino core** for your board (e.g., ESP32), **Servo** & **micro\_ros\_arduino** libraries.
2. Select your board/port.
3. Open the `.ino` below and **Upload**.

> For **micro\_ros\_arduino**: install via Library Manager or from the official repo; select **Serial** transport by default.

### 2) PlatformIO (optional)

Create a project and add libs:

```ini
[env:esp32]
platform = espressif32
board = esp32dev
framework = arduino
lib_deps =
  micro_ros_arduino
  Servo
```

---

## 🛰️ Run micro-ROS Agent

On your ROS 2 machine:

```bash
# Example: serial agent at /dev/ttyUSB0  (adjust port/baud)
ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyUSB0 -b 115200
```

In another terminal, verify the topic:

```bash
ros2 topic echo /final_cmd_vel
```

Publish a quick test:

```bash
ros2 topic pub -r 5 /final_cmd_vel geometry_msgs/Twist '{linear: {x: 2.0}}'
```

---

## 🔧 Configuration Knobs

Edit these constants in code:

```cpp
#define SERVO_MIN_US        800
#define SERVO_MAX_US        2200
#define SERVO_NEUTRAL_US    1500
#define SERVO_IDLE_START_US  880   // where ramp starts when idling
#define SERVO_IDLE_STEP_US     2   // ramp step per loop
#define IDLE_TIMEOUT_MS     200   // after last cmd before soft-stop kicks in
#define DIGIPOT_RUN_VALUE   100   // wiper value when moving
```

---



---

## 🔍 Debug Tips

* If the **Agent** is not running, the node won’t receive commands.
* Use `ros2 topic echo /final_cmd_vel` to verify publishes.
* Scope/logic analyzer can help confirm **SPI** traffic and **GPIO** levels.
* If servo jitters, check **grounding** and **power** quality (add a 100–470 µF cap near the servo power).

---

## 📝 License

MIT

---

## 🇹🇭 สรุปภาษาไทย

สเก็ตช์นี้ทำหน้าที่เป็น **micro-ROS Subscriber** ที่รับ `Twist` จาก Topic `/final_cmd_vel` แล้วแปลง **`linear.x`** เป็นการควบคุม **ทิศทาง (OUT\_PIN0/1/2)**, ตั้งค่า **ดิจิโพเทนชิออมิเตอร์** (ผ่าน SPI) และสั่ง **เซอร์โว** ผ่าน `writeMicroseconds()` พร้อม **soft-stop** เมื่อไม่มีคำสั่งเข้ามา เพื่อหยุดอย่างนุ่มนวล เหมาะสำหรับระบบต้นแบบที่ต้องการควบคุมโหลดอนาล็อก + ทิศทาง + เซอร์โวจาก ROS 2

> ต้องการแยกไฟล์เป็น `README.md` และ `.ino` ให้ไหม หรือเพิ่มตัวเลือก **PlatformIO** พร้อม `platformio.ini`? บอกมาได้เลย 🙂
