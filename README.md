🧠 โหนด micro-ROS สำหรับควบคุม Digital Potentiometer และ Servo
โปรเจกต์นี้เป็นการสร้างโหนด micro-ROS บนบอร์ดไมโครคอนโทรลเลอร์ที่รองรับ Arduino เพื่อรับข้อมูลจาก Topic ชื่อ /final_cmd_vel และนำไปควบคุมอุปกรณ์ต่างๆ ดังนี้:

Digital Potentiometer (ควบคุมผ่านโปรโตคอล SPI)

Servo Motor

พินเอาท์พุตดิจิทัล 3 เส้น เพื่อควบคุมลอจิกทิศทาง

✅ คุณสมบัติเด่น (Features)
รับข้อมูล (Subscribe) จาก Topic /final_cmd_vel ซึ่งมีประเภทข้อความ (Message Type) เป็น geometry_msgs/msg/Twist

ใช้ค่า linear.x จาก Message เพื่อควบคุม:

ความกว้างของพัลส์เซอร์โว (Servo pulse width) สำหรับควบคุมความเร็วมอเตอร์หรือปรับมุม

ค่าความต้านทานของ Digital Potentiometer เพื่อปรับกระแส/แรงดัน

พอร์ต GPIOs สำหรับควบคุมทิศทาง

มีการลดค่า pulseWidth ของเซอร์โวลงอย่างช้าๆ เมื่อไม่มีคำสั่งเข้ามา (ค่าเป็นกลาง) เพื่อให้การหยุดเป็นไปอย่างนุ่มนวล

แสดงผลการดีบัก (Debug) ผ่านทาง Serial Monitor

📦 อุปกรณ์ที่ต้องใช้ (Hardware Requirements)
บอร์ด Arduino (หรือบอร์ดอื่นที่รองรับ micro-ROS และ SPI เช่น ESP32)

Digital Potentiometer (ควบคุมผ่าน SPI) — เช่นเบอร์ MCP41xxx

Servo motor

สายควบคุมดิจิทัล 3 เส้น (สำหรับ OUT_PIN0, OUT_PIN1, OUT_PIN2)

คอมพิวเตอร์ที่รัน ROS 2 (เช่น Jetson, NUC) พร้อมกับ Micro-ROS Agent

🧩 การเชื่อมต่อกับ ROS 2 (ROS 2 Integration)
โหนดนี้จะรับฟังข้อมูลจาก:

Bash

/final_cmd_vel  (geometry_msgs/msg/Twist)
การแปลงคำสั่งจาก linear.x (Command Mapping)
ค่า $x$	การทำงาน
2.0	ตั้งค่าให้ OUT_PIN0 เป็น HIGH, OUT_PIN1 เป็น LOW และสั่งงาน OUT_PIN2 หลังจากนั้นครู่หนึ่ง (delay) พร้อมตั้งค่า Digital Potentiometer ไปที่ 100
-1.0	ตั้งค่าให้ OUT_PIN1 เป็น HIGH, OUT_PIN0 เป็น LOW และสั่งงาน OUT_PIN2 เป็น HIGH พร้อมตั้งค่า Digital Potentiometer ไปที่ 100 เช่นกัน
อื่นๆ	ตั้งค่าพินทั้งหมดเป็น LOW, Digital Potentiometer เป็น 0 และค่า pulseWidth ของเซอร์โวจะค่อยๆ เพิ่มขึ้น เพื่อจำลองการเบรกหรือหยุดอย่างนุ่มนวล

ส่งออกไปยังชีต
🔧 ภาพรวมโครงสร้างโค้ด (Code Structure Overview)
การตั้งค่าเริ่มต้น (Initialization)
C++

set_microros_transports();      // ตั้งค่าการสื่อสารสำหรับ micro-ROS
SPI.begin();                    // เริ่มการทำงานของ SPI สำหรับ Digipot
servoMotor.attach(...);         // กำหนดค่าและเชื่อมต่อ Servo
การตั้งค่า micro-ROS (micro-ROS Setup)
C++

rclc_support_init(...)
rclc_node_init_default(...)
rclc_subscription_init_default(...)
rclc_executor_add_subscription(...)
ลูปการอัปเดตเซอร์โว (Servo Update Loop)
C++

servoMotor.writeMicroseconds(pulseWidth);  // อัปเดตค่าเซอร์โวทุกรอบการทำงาน
🔬 ภาคผนวก: คำอธิบายโค้ดโดยละเอียด
โค้ดนี้เป็นโปรแกรมสำหรับบอร์ดไมโครคอนโทรลเลอร์ที่รองรับ micro-ROS (เช่น ESP32, Arduino) โดยมีการทำงานหลักคือ:

รับคำสั่งความเร็วจาก ROS 2 ผ่าน Topic /final_cmd_vel

แปลงคำสั่งเพื่อควบคุมทิศทาง (ผ่าน Output Pins)

ควบคุมแรงดัน/กระแส ผ่าน Digital Potentiometer (DigiPot) ด้วยโปรโตคอล SPI

ส่งค่า PWM เพื่อควบคุมเซอร์โวผ่านคำสั่ง writeMicroseconds

1. ไลบรารีและตัวแปรที่ใช้ (Libraries and Variables)
C++

#include <Servo.h>
#include <SPI.h>
#include <micro_ros_arduino.h>
#include <rcl/rcl.h>
#include <rclc/rclc.h>
#include <rclc/executor.h>
#include <geometry_msgs/msg/twist.h>
โปรแกรมใช้ไลบรารีมาตรฐานสำหรับ Servo, SPI และไลบรารีสำหรับ micro-ROS เพื่อเชื่อมต่อกับ ROS 2 นอกจากนี้ยังมีการสร้างตัวแปร global สำหรับจัดการ Node, Executor, และ Message ของ ROS (geometry_msgs/msg/Twist)

2. การควบคุม DigiPot (DigiPot Control)
C++

void write_digipot(int val) {
  digitalWrite(CS_DIGIPOT, LOW);
  SPI.transfer(B00010001); // Command byte to write to wiper register
  SPI.transfer(val);      // Value (0-255)
  digitalWrite(CS_DIGIPOT, HIGH);
}
ฟังก์ชันนี้ใช้สำหรับเขียนค่า (0-255) ไปยัง Wiper Register ของ DigiPot ผ่าน SPI เพื่อปรับค่าความต้านทาน ซึ่งสามารถนำไปประยุกต์ใช้ควบคุมความแรงของอุปกรณ์อื่น เช่น ปรับแรงดันที่จ่ายให้มอเตอร์

3. การควบคุมทิศทางผ่าน Output Pins
C++

#define OUT_PIN0 0
#define OUT_PIN1 1
#define OUT_PIN2 2
มีการใช้ Output Pin 3 เส้นเพื่อควบคุมลอจิกของวงจรขับเคลื่อน เช่น การกำหนดทิศทางการหมุน (ซ้าย/ขวา) และการเปิด/ปิดการทำงานของมอเตอร์

4. การรับข้อมูลจาก ROS Topic
C++

void cmd_vel_callback(const void * msgin)
นี่คือฟังก์ชัน Callback ซึ่งจะถูกเรียกใช้งานทุกครั้งที่มีข้อความใหม่เข้ามาใน Topic /final_cmd_vel โดยมีตรรกะการทำงานดังนี้:

ถ้า linear.x == 2.0 → สั่งให้หมุนไปในทิศทางที่หนึ่ง

ถ้า linear.x == -1.0 → สั่งให้หมุนไปในทิศทางตรงกันข้าม

ถ้าไม่ใช่ทั้งสองกรณี → หยุดการทำงาน โดยตั้งค่า Output Pin ทั้งหมดเป็น LOW และลดค่า DigiPot เป็น 0

5. การควบคุมเซอร์โว (Servo Control)
C++

servoMotor.writeMicroseconds(pulseWidth);
โปรแกรมใช้คำสั่ง writeMicroseconds เพื่อส่งสัญญาณ PWM ควบคุมเซอร์โวด้วยความละเอียดระดับไมโครวินาที ซึ่งให้ความแม่นยำสูงกว่าคำสั่ง write() ทั่วไป โดยเมื่อไม่มีคำสั่งเข้ามา (สถานะหยุด) ค่า pulseWidth จะเริ่มต้นที่ 800 และค่อยๆ เพิ่มขึ้นทีละ 2 จนถึง 880 เพื่อจำลองการเบรกอย่างนุ่มนวล

6. การตั้งค่า micro-ROS
C++

rclc_support_init(...)
rclc_node_init_default(...)
rclc_subscription_init_default(...)
ฟังก์ชันเหล่านี้ใช้สำหรับตั้งค่าส่วนประกอบต่างๆ ของ micro-ROS ได้แก่ Node, Subscription, และ Executor เพื่อให้บอร์ดสามารถสื่อสารกับ ROS Master ได้ โดย Subscription จะทำหน้าที่รับข้อความจาก ROS และเรียกใช้ฟังก์ชัน Callback (cmd_vel_callback) เพื่อควบคุมฮาร์ดแวร์

7. โครงสร้างหลักของโปรแกรม
setup(): เป็นฟังก์ชันที่ทำงานครั้งเดียวเมื่อเปิดเครื่อง ใช้สำหรับตั้งค่าพื้นฐานต่างๆ เช่น I/O pins, เริ่มการสื่อสาร SPI, ตั้งค่า ROS 2 node, และเชื่อมต่อกับเซอร์โวมอเตอร์

loop(): เป็นฟังก์ชันที่ทำงานวนซ้ำไปเรื่อยๆ ทำหน้าที่ตรวจสอบข้อความใหม่จาก ROS ผ่าน 
