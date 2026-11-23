# swichfinger
Smart Mechanical Button Pusher (ESP8266 &amp; Home Assistant)
Tired of carrying physical remotes around or dealing with legacy devices that lack smart capabilities? This 3D-printed mechanical button pusher allows you to actuate any physical button or switch remotely through Home Assistant using a low-cost ESP8266 module and a micro servo motor.
🛠️ Bill of Materials (BOM)
3D Printed Parts

    1x Main Enclosure / Mounting Bracket (included raw STL files)
    1x Servo Arm / Pusher Lever

Hardware & Fasteners

    4x M2 × 12 mm Screws
    8x M3 × 16 mm Screws

Electronics

    1x ESP8266 Microcontroller (NodeMCU / Wemos D1 Mini / ESP-12)
    1x Micro Servo Motor 9g (SG90 or MG90S)
    1x Micro-USB Power Cable & 5V Adapter

🖨️ Recommended Print Settings

    Material: PLA or PETG
    Layer Height: 0.20 mm
    Infill: 20% (Grid or Gyroid)
    Supports: Recommended for mounting tabs (depending on orientation)
    Walls/Perimeters: 3

⚡ Wiring & Pinout
Servo Motor Wire	ESP8266 Pin
Yellow / Orange (Signal)	GPIO 5 (D1)
Red (VCC)	5V / VIN
Brown / Black (GND)	GND

import network
import socket
import time
from machine import Pin, PWM

# ==============================================================================
# WI-FI CONFIGURATION
# ==============================================================================
WIFI_SSID = "YOUR_WIFI_SSID"
WIFI_PASS = "YOUR_WIFI_PASSWORD"
DEVICE_NAME = "finger-test"

# ==============================================================================
# SERVO MOTOR CONFIGURATION (GPIO 5 / Pin D1)
# ==============================================================================
servo = PWM(Pin(5), freq=50)

ANGLE_OFF = 90   # Idle / OFF position (90 degrees)
ANGLE_ON = 55    # Pressed / ON position (55 degrees)

current_state = "off"

def set_angle(angle):
   angle = max(0, min(180, angle))
   duty_val = int(25 + (angle / 180) * 100)
   servo.duty(duty_val)

def turn_on():
   global current_state
   set_angle(ANGLE_ON)
   current_state = "on"

def turn_off():
   global current_state
   set_angle(ANGLE_OFF)
   current_state = "off"

# Move to default OFF position on startup
turn_off()

# ==============================================================================
# WI-FI CONNECTION (DHCP)
# ==============================================================================
def connect_wifi():
   wlan = network.WLAN(network.STA_IF)
   wlan.active(True)
   
   try:
       wlan.config(dhcp_hostname=DEVICE_NAME)
   except Exception:
       pass

   wlan.connect(WIFI_SSID, WIFI_PASS)
   print("Connecting to Wi-Fi...")
   
   timeout = 20
   while not wlan.isconnected() and timeout > 0:
       time.sleep(1)
       timeout -= 1
       
   if wlan.isconnected():
       print("Connected to Wi-Fi! Device IP:", wlan.ifconfig()[0])
   else:
       print("Wi-Fi connection failed.")

# ==============================================================================
# HTTP SERVER (PORT 80)
# ==============================================================================
def start_server():
   addr = socket.getaddrinfo('0.0.0.0', 80)[0][-1]
   s = socket.socket()
   s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
   s.bind(addr)
   s.listen(5)
   print("HTTP Server active on port 80...")

   while True:
       try:
           conn, client_addr = s.accept()
           request = conn.recv(1024).decode('utf-8')
           
           if "/off" in request:
               turn_off()
           elif "/on" in request:
               turn_on()
           elif "PUT" in request or "POST" in request:
               if current_state == "off":
                   turn_on()
               else:
                   turn_off()

           response = f"HTTP/1.1 200 OK\r\nContent-Type: application/json\r\nConnection: close\r\n\r\n{{\"status\":\"ok\",\"state\":\"{current_state}\"}}"
           conn.send(response)
           conn.close()

       except Exception as e:
           try:
               conn.close()
           except Exception:
               pass

connect_wifi()
start_server()

 
🏠 Home Assistant Integration

Add the following YAML snippet to your configuration.yaml in Home Assistant. Replace YOUR_ESP8266_IP with the local IP address assigned to your module by your router.

 

command_line:
 - switch:
     unique_id: finger_test_esp_cmd
     name: "finger test"
     command_on: "curl -X GET http://YOUR_ESP8266_IP/on"
     command_off: "curl -X GET http://YOUR_ESP8266_IP/off"

 

If you found this model helpful, please leave a Like, share your Makes, or leave feedback in the comments!
