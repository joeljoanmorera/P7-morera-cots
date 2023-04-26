# Práctica 7. Gerard Cots y Joel J. Morera

## Ejercicio practico 1 : Reproducción desde memoria interna

###### **Funcionamiento**

En este ejercico se implementa la reproducción de un archivo AAC desde la memoria interna del ESP32. Para ello, se establece un puerto de comunicacion I2S entre el processador y el DAC. Por último, una vez inicializados ambos dispositivos, se reproduce el archivo AAC.

###### **Código del programa**

- platformio.ini

```
[env:esp32doit-devkit-v1]
platform = espressif32
board = esp32doit-devkit-v1
framework = arduino
monitor_port = /dev/ttyUSB0
monitor_speed = 115200
lib_deps = earlephilhower/ESP8266Audio@^1.9.7
```

- main.cpp

```cpp
#include "AudioGeneratorAAC.h"
#include "AudioOutputI2S.h"
#include "AudioFileSourcePROGMEM.h"
#include "sampleaac.h"

AudioFileSourcePROGMEM *in;
AudioGeneratorAAC *aac;
AudioOutputI2S *out;

void setup()
{
    Serial.begin(115200);
    in = new AudioFileSourcePROGMEM(sampleaac, sizeof(sampleaac));
    aac = new AudioGeneratorAAC();
    out = new AudioOutputI2S();
    out -> SetGain(0.125);
    out -> SetPinout(26,25,22);
    aac->begin(in, out);
}
void loop()
{
    if(aac->isRunning())
    {
        aac->loop();
    }
    else 
    {
        aac -> stop();
        Serial.printf("Sound Generator\n");
        delay(1000);
    }
}
```

###### **Salida del puerto serie**

Pese a que la salida por el puerto serie no nos de información respecto a lo que sucede en el mundo físico, al ejecutar el programa comprovamos que se reproduce el audio descrito en el fichero `sampleaac.h`,

```
Sound Generator
```

***

## Ejercicio practico 2 : Reproducir un archivo WAW en ESP32 desde una targeta SD externa

###### **Funcionamiento**

En este ejercicio se implementa la reproducción de un archivo WAV desde una tarjeta SD externa. Para ello, despues de añadir el fichero wav a la tarjeta SD, se establece un puerto de comunicacion SPI entre la tarjeta SD y el ESP32. Por otra parte, se establece un puerto de comunicacion I2S entre el processador y el DAC. Por último, una vez inicializados ambos dispositivos, se reproduce el archivo WAV.

###### **Código del programa**

- platformio.ini:

```
[env:esp32doit-devkit-v1]
platform = espressif32
board = esp32doit-devkit-v1
framework = arduino
monitor_port = /dev/ttyUSB0
monitor_speed = 115200
lib_deps =  esphome/ESP32-audioI2S @ ^2.0.6
```

- main.cpp:

```cpp
#include "Audio.h"
#include "SD.h"
#include "FS.h"

// Digital I/O used
#define SD_CS       5
#define SPI_MOSI    23    
#define SPI_MISO    19    
#define SPI_SCK     18    
#define I2S_DOUT    25    
#define I2S_BCLK    27    
#define I2S_LRC     26    

Audio audio;

void setup()
{
    pinMode(SD_CS, OUTPUT);
    digitalWrite(SD_CS, HIGH);
    SPI.begin(SPI_SCK, SPI_MISO, SPI_MOSI);
    Serial.begin(115200);
    SD.begin(SD_CS);
    audio.setPinout(I2S_BCLK, I2S_LRC, I2S_DOUT);
    audio.setVolume(21); // 0...21
    audio.connecttoFS(SD, "come-together.wav");
}

void loop()
{
    audio.loop();
}

// optional
void audio_info(const char *info)
{
    Serial.print("info");
    Serial.println(info);
}

void audio_id3data(const char *info) //id3 metadata
{
    Serial.print("id3data");
    Serial.println(info);
}

void audio_eof_mp3(const char *info) //end of file
{
    Serial.print("eof_mp3");
    Serial.println(info);
}

void audio_showstation(const char *info)
{
    Serial.print("station ");
    Serial.println(info);
}

void audio_showstreaminfo(const char *info)
{
    Serial.print("streaminfo ");
    Serial.println(info);
}

void audio_showstreamtitle(const char *info)
{
    Serial.print("streamtitle ");
    Serial.println(info);
}

void audio_bitrate(const char *info)
{
    Serial.print("bitrate ");
    Serial.println(info);
}

void audio_commercial(const char *info) //duration in sec
{
    Serial.print("commercial ");
    Serial.println(info);
}

void audio_icyurl(const char *info) //homepage
{
    Serial.print("icyurl ");
    Serial.println(info);
}

void audio_lasthost(const char *info) //stream URL played
{
    Serial.print("lasthost ");
    Serial.println(info);
}

void audio_eof_speech(const char *info)
{
    Serial.print("eof_speech ");
    Serial.println(info);
}
```

###### **Salida del puerto serie**

De la misma manera que en el ejercicio anterior, aunque la salida por el puerto serie no nos ofrece una interpretación real de lo que sucede, se reproduce el archivo WAV dentro de la SD.

###### **Montaje**

![Montaja WAV](./images/mount_wav.png)

***