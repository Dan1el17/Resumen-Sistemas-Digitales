# CLASE DE SISTEMAS DIGITALES

**Fecha:** 24-11-2026  
**Semana:** 5

## Procesamiento Digital de Señales y Pantallas LCD en ESP32

---

## ¿Qué es el Procesamiento Digital de Señales?

El PDS transforma variables físicas continuas en secuencias de números que un microcontrolador puede manipular.

### Cadena de digitalización
- **Captura física:** convierte una magnitud real en voltaje continuo.
- **Discretización en el tiempo.**

### Resolución del ADC: ¿cuántos bits necesito?
- Niveles por escalón.

### Teorema de muestreo de Nyquist

### Operaciones fundamentales del PDS
- Filtrado digital.
- Transformada rápida de Fourier.
- Convolución discreta.

### ADC del ESP32

### Filtro de promedio móvil

### Ver las frecuencias de las señales
- Detectar ruido o interferencias.

---

## ¿Qué es una pantalla LCD?

**Formatos comunes.**

**LCD** = *Liquid Crystal Display*

### Controlador
- **DDRAM**
- **CGROM**
- **CGRAM**

### Los 16 pines del LCD

### Conexión paralela vs módulo I2C

### El módulo adaptador I2C

### Problemas comunes y cómo resolverlos
- Se ven cuadros negros.
- Pantalla en blanco.

---

## Procesamiento Digital de Señales

> El mundo es analógico; el microcontrolador es digital.

### ¿Qué es el procesamiento digital de señales?
Convierte la señal en una lista de números y hace cuentas con ellos:
- **Convertir:** a unidades físicas (de mV a °C).
- **Limpiar:** reducir el ruido, filtrar.
- **Decidir:** encender una alarma si hace calor.
- **Mostrar o enviar:** en una LCD o por la red.

### La cadena completa: del sensor a la decisión
**SENSOR → ADC → CÓDIGO → SALIDA**

**ADC:** es un conversor analógico-digital, un periférico del microcontrolador que mide un voltaje y lo convierte en un número.

### Muestreo
- **Muestra:** el valor que se toma dentro de un instante.
- **Muestreo:** tomar valores dentro de intervalos fijos.

### Período y frecuencia de muestreo
- **Ts:** período de muestreo. El tiempo que hay entre dos muestras.
- **fs:** frecuencia de muestreo. Muestras por segundo (Hz = veces por segundo).

$$f_s = \frac{1}{T_s}$$

### Teorema de Nyquist: ¿qué tan rápido?

**Aliasing:** muestrear demasiado lento engaña, como las ruedas que giran al revés en un video (la cámara toma pocas fotos por segundo y la rueda parece ir lenta o hacia atrás).

**¿A qué frecuencia aparece la señal falsa?**

$$f_{falsa} = |f - k \cdot f_s|$$

**Ejemplo:**
- Ts = 250 ms → Señal de 3 Hz → 4 < 6 → |3 − 4| = 1 Hz
- fs = 4 Hz → Nyquist pide > 6 Hz → hay aliasing → señal falsa

### Cuantización
Con qué detalle se mide cada muestra: redondear cada muestra al escalón más cercano.

**Error:** lo que se pierde al redondear, como máximo medio escalón.

### ¿Cuántos escalones tiene el ADC?

**LSB:** la altura de un escalón.

$$LSB = \frac{fondo\ de\ escala}{2^n}$$

### Resolución no es exactitud
- **Resolución:** qué tan pequeño es cada escalón.
- **Exactitud:** qué tan cerca está la lectura del valor real.

*Ejemplo:* una balanza con 3 decimales, descalibrada 50 g. Muestra mucho detalle, pero se equivoca. Al ADC del ESP32 le pasa algo parecido por el ruido.

### ¿Cómo encuentra el ADC el número?
Adivinando.

¿≥ 4? → ¿≥ 6? → ¿≥ 5? → 101 = 5  
Si → bit 1 · No → bit 0 · Si → bit 1 · el 0.3 se pierde

---

## El ESP32 tiene dos ADC

### ADC1 — GPIO 32 a 39
- 8 canales.
- Funciona con Wi-Fi.

### ADC2 — GPIO 0, 2, 4, 12-15, 25-27
- 10 canales.
- No funciona con Wi-Fi encendido.
- Varios pines de strapping.

### Arduino UNO vs ESP32

| Arduino UNO | ESP32 |
|---|---|
| Se alimenta de 5 V | Se alimenta con 3,3 V |
| Su ADC usa esos 5 V como regla | Su ADC usa una regla |
| Mide de 0 a 5 V | |

### Atenuación: el zoom del ADC

**Entrada** (voltaje del sensor) → **Atenuador** (divisor interno: achica) → **ADC** (regla de = 1,1 V)

- **0 dB = zoom cerca:** no achica, poco rango, escalones finos.
- **11 dB = zoom lejos:** achica 3,5 veces, mucho rango, escalones más grandes.

### Calibración: cada chip es un poco distinto

| `analogRead(pin)` | `analogReadMilliVolts(pin)` |
|---|---|
| Número crudo de 0 a 4095 | Milivoltios |
| Sin calibrar | Calibrados con los datos de fábrica |

El chip guarda su propio error y lo corrige.

### Ruido: un condensador ayuda
- **Remedio 1 — hardware:** condensador cerámico de 100 nF entre el pin del sensor y GND, cerca del pin.
- **Remedio 2 — software:** tomar varias muestras y promediarlas con un filtro.