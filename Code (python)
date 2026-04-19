import time
from gpiozero import Servo
from collections import deque

# --- TCS34725 SETUP ---
import board
import busio
import adafruit_tcs34725

i2c = busio.I2C(board.SCL, board.SDA)
sensor = adafruit_tcs34725.TCS34725(i2c)

sensor.integration_time = 50   # <<< CHANGE THIS (10–100 typical)
sensor.gain = 4                # <<< CHANGE THIS (1, 4, 16, 60)

# --- SERVOS ---
servo_red = Servo(18)
servo_blue = Servo(19)

# --- SETTINGS ---
TRAVEL_TIME = 1.2       # <<< CHANGE THIS (time from sensor → servo)
SERVO_HOLD_TIME = 0.5   # <<< CHANGE THIS (how long servo moves)

# --- COLOR MEMORY QUEUE ---
color_queue = deque()

# --- REAL COLOR SENSOR FUNCTION ---
def read_color():
    r, g, b = sensor.color_rgb_bytes

    total = r + g + b
    if total == 0:
        return None

    r_ratio = r / total
    b_ratio = b / total

    # Debug (watch this while tuning)
    print(f"RGB: {r},{g},{b} | R={r_ratio:.2f} B={b_ratio:.2f}")

    if r_ratio > 0.45:   # <<< CHANGE THIS
        return "RED"

    elif b_ratio > 0.45: # <<< CHANGE THIS
        return "BLUE"

    return None


# --- DETECT COLOR ---
def detect_color():
    color = read_color()

    if color in ["RED", "BLUE"]:
        color_queue.append((color, time.time()))
        print(f"Queued: {color}")


# --- HANDLE TIMED TRIGGER ---
def process_queue():
    now = time.time()

    if color_queue:
        color, timestamp = color_queue[0]

        if now - timestamp >= TRAVEL_TIME:
            color_queue.popleft()

            if color == "RED":
                print("RED → Servo 1")
                servo_red.max()
                time.sleep(SERVO_HOLD_TIME)
                servo_red.mid()

            elif color == "BLUE":
                print("BLUE → Servo 2")
                servo_blue.max()
                time.sleep(SERVO_HOLD_TIME)
                servo_blue.mid()


# --- MAIN LOOP ---
while True:
    detect_color()
    process_queue()
    time.sleep(0.05)
