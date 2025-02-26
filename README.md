from machine import Pin, I2C
from time import sleep
import utime
import ssd1306  # Librería para el display LCD con I2C

# Configuración del sensor de ultrasonidos
trigger = Pin(3, Pin.OUT)
echo = Pin(2, Pin.IN)

# Configuración del I2C para el display LCD
i2c = I2C(0, scl=Pin(5), sda=Pin(4), freq=400000)
display = ssd1306.SSD1306_I2C(128, 64, i2c)

# Distancia umbral para considerar la caneca llena (en cm)
DISTANCIA_UMBRAL = 10

def medir_distancia():
    # Enviar pulso de trigger
    trigger.low()
    utime.sleep_us(2)
    trigger.high()
    utime.sleep_us(10)
    trigger.low()

    while echo.value() == 0:
        signal_off = utime.ticks_us()
    while echo.value() == 1:
        signal_on = utime.ticks_us()

    tiempo = utime.ticks_diff(signal_on, signal_off)
    distancia = (tiempo * 0.0343) / 2
    return distancia

def main():
    while True:
        distancia = medir_distancia()
        display.fill(0)
        if distancia <= DISTANCIA_UMBRAL:
            display.text('Caneca llena', 0, 0)
        else:
            display.text('Caneca vacia', 0, 0)
        display.text('Distancia:', 0, 10)
        display.text(str(distancia) + ' cm', 0, 20)
        display.text('Faltan:', 0, 30)
        display.text(str(DISTANCIA_UMBRAL - distancia) + ' cm', 0, 40)
        display.show()
        sleep(1)

if __name__ == "__main__":
    main()
