import network
import urequests
import time
from machine import Pin, time_pulse_us

# Konfigurasi WiFi
SSID = "Wokwi-GUEST"
PASSWORD = ""

# Konfigurasi Ubidots REST API
UBIDOTS_TOKEN = "BBUS-sXgpjWiZaaEEoRvGP6MJqZ9I3AD3IA"
DEVICE_LABEL = "ultrasonic_device"
VARIABLE_LABEL = "distance"
UBIDOTS_URL = f"http://industrial.api.ubidots.com/api/v1.6/devices/{"my-esp32"}/"

# Konfigurasi API FastAPI
API_URL = "http://your_fastapi_server_ip:8000/data"

# Koneksi ke WiFi
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect(SSID, PASSWORD)

max_wait = 10
while max_wait > 0:
    if wlan.isconnected():
        break
    max_wait -= 1
    time.sleep(1)

if not wlan.isconnected():
    raise RuntimeError("Network connection failed")
else:
    print("Connected to WiFi")

# Konfigurasi Sensor Ultrasonik
TRIG = Pin(5, Pin.OUT)
ECHO = Pin(18, Pin.IN)

def get_distance():
    """Mengukur jarak menggunakan sensor ultrasonik."""
    TRIG.off()
    time.sleep_us(2)
    TRIG.on()
    time.sleep_us(10)
    TRIG.off()
    duration = time_pulse_us(ECHO, 1)  # Hitung durasi pulsa
    distance = (duration * 0.0343) / 2  # Konversi durasi ke jarak (cm)
    return distance

def send_data(distance):
    """Mengirim data jarak ke Ubidots melalui REST API."""
    headers = {
        "X-Auth-Token": UBIDOTS_TOKEN,
        "Content-Type": "application/json"
    }
    payload = {VARIABLE_LABEL: {"value": distance}}
    try:
        response = urequests.post(UBIDOTS_URL, json=payload, headers=headers)
        print("Data sent to Ubidots: ", response.text)
        response.close()
    except Exception as e:
        print("Failed to send data to Ubidots: ", e)

def send_to_api(distance):
    """Mengirim data jarak ke API FastAPI untuk disimpan di MongoDB."""
    headers = {"Content-Type": "application/json"}
    payload = {"device": DEVICE_LABEL, "distance": distance}
    try:
        response = urequests.post(API_URL, json=payload, headers=headers)
        print("Data sent to API: ", response.text)
        response.close()
    except Exception as e:
        print("Failed to send data to API: ", e)

while True:
    distance = get_distance()
    print(f"Distance: {distance} cm")
    send_data(distance)  # Kirim ke Ubidots
    send_to_api(distance)  # Kirim ke API FastAPI
    time.sleep(5)

