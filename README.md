import machine
import urequests
import time
import network

# Koneksi Wi-Fi
ssid = "Wokwi-GUEST"
password = ""

def connect_wifi():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    if not wlan.isconnected():
        wlan.connect(ssid, password)
        while not wlan.isconnected():
            pass
    print("Connected to Wi-Fi:", wlan.ifconfig())

connect_wifi()

# Inisialisasi sensor PIR
pir = machine.Pin(13, machine.Pin.IN)

# Ubidots setup
TOKEN = "BBUS-txswRGOgXbtcHmhXBAfjB8abEspEBQ"
DEVICE_LABEL = "motion-monitor"
VARIABLE_LABEL = "motion"

def send_to_ubidots(status):
    url = f"https://industrial.api.ubidots.com/api/v1.6/devices/{DEVICE_LABEL}"
    headers = {
        "X-Auth-Token": TOKEN,
        "Content-Type": "application/json"
    }
    data = {VARIABLE_LABEL: status}
    try:
        response = urequests.post(url, json=data, headers=headers)
        response.close()
        print("Data sent:", data)
    except:
        print("Failed to send data")

# Loop utama
while True:
    motion_detected = pir.value()
    send_to_ubidots(motion_detected)
    time.sleep(5)



   
    
    
    


