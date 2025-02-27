import time
import RPi.GPIO as GPIO
from smbus2 import SMBus

# Configuración de pines para el sensor ultrasónico
TRIG = 23  # Pin GPIO para Trigger
ECHO = 24  # Pin GPIO para Echo
ALTURA_CANECA = 31  # Altura total de la caneca en cm

# Configuración de la pantalla LCD I2C
I2C_ADDR = 0x27  # Dirección del LCD
LCD_WIDTH = 16   # Tamaño de la pantalla (16x2)

# Comandos de la pantalla LCD
LCD_CHR = 1
LCD_CMD = 0
LCD_LINE_1 = 0x80  # Primera línea
LCD_LINE_2 = 0xC0  # Segunda línea
ENABLE = 0b00000100

# Inicializar la Raspberry Pi
GPIO.setmode(GPIO.BCM)
GPIO.setup(TRIG, GPIO.OUT)
GPIO.setup(ECHO, GPIO.IN)

# Función para enviar datos a la pantalla LCD
def lcd_send_byte(bits, mode):
    bus = SMBus(1)
    bits_high = mode | (bits & 0xF0) | ENABLE
    bits_low = mode | ((bits << 4) & 0xF0) | ENABLE
    bus.write_byte(I2C_ADDR, bits_high)
    bus.write_byte(I2C_ADDR, bits_low)

# Inicializar LCD
def lcd_init():
    lcd_send_byte(0x33, LCD_CMD)
    lcd_send_byte(0x32, LCD_CMD)
    lcd_send_byte(0x28, LCD_CMD)
    lcd_send_byte(0x0C, LCD_CMD)
    lcd_send_byte(0x06, LCD_CMD)
    lcd_send_byte(0x01, LCD_CMD)
    time.sleep(0.2)

# Mostrar mensaje en la pantalla
def lcd_display(message, line):
    lcd_send_byte(line, LCD_CMD)
    for char in message.ljust(LCD_WIDTH, " "):
        lcd_send_byte(ord(char), LCD_CHR)

# Función para medir la distancia con el sensor ultrasónico
def medir_distancia():
    GPIO.output(TRIG, True)
    time.sleep(0.00001)
    GPIO.output(TRIG, False)

    start_time = time.time()
    stop_time = time.time()

    while GPIO.input(ECHO) == 0:
        start_time = time.time()

    while GPIO.input(ECHO) == 1:
        stop_time = time.time()

    tiempo_transcurrido = stop_time - start_time
    distancia = (tiempo_transcurrido * 34300) / 2  # Convertir a cm
    return round(distancia, 1)

# Programa principal
try:
    lcd_init()
    lcd_display("Cargando...", LCD_LINE_1)
    time.sleep(2)
    lcd_display("                ", LCD_LINE_1)

    while True:
        distancia = medir_distancia()

        if distancia <= 1:
            lcd_display("Caneca llena", LCD_LINE_1)
        elif 1 < distancia < ALTURA_CANECA:
            lcd_display("Distancia:", LCD_LINE_1)
            lcd_display(f"{distancia} cm", LCD_LINE_2)
        else:
            lcd_display("Caneca vacia", LCD_LINE_1)
            lcd_display(f"{ALTURA_CANECA} cm", LCD_LINE_2)

        time.sleep(1)  # Refresco cada segundo

except KeyboardInterrupt:
    print("Saliendo...")
    GPIO.cleanup()
