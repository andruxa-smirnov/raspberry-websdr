# Nodo WEBSdr utilizando un Raspberry PI 3
Leer en [Inglés](README.md), [Español](README.es.md)
Read in [English](README.md), [Español](README.es.md)

![Receptor SDR funcionando en la banda de 40 metros](https://github.com/reynico/raspberry-websdr/raw/master/sdr-40m.jpg)

Esta guía cubre la configuración de un receptor de doble banda (80/40 metros) basada en forma horaria. Usa un relay para intercambiar entre antenas, controlado por un puerto GPiO del Raspberry PI (utilizando un transistor como driver).

Muchas gracias a Pieter PA3FWM, Mark GP4FPH y Jarek SQ9NFI por la gran mano configurando el parámetro progfreq.req setting.

### Requerimientos
- Raspberry PI 3
- Raspbian 9 instalado y funcionando
- Acceso a internet configurado y funcionando
- Receptor RTL-SDR USB

### Configuración y software requerido
```
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install g++ make libsigc++-1.2-dev libgsm1-dev libpopt-dev tcl8.5-dev libgcrypt-dev libspeex-dev libasound2-dev alsa-utils libqt4-dev
sudo apt-get install libsigc++ cmake groff rtl-sdr
```

### Configuración de los pines GPIO
- Copia el archivo etc/rc.local a tu /etc/rc.local
```
sudo cp etc/rc.local /etc/rc.local
```
- Controla y revisa el número de puerto GPiO para el control de antenas y el botón de reinicio por software.

### Reinicio por software
- Hay un script en Python que controla el reinicio de la Raspberry PI a través de un switch de hardware, sin necesidad de quitarle la energía eléctrica.
- Revisa el archivo /etc/rc.local y sincroniza el pin GPiO designado para esta aplicación.
- Copia el archivo lib/systemd/system/reset.service a /lib/systemd/system/reset.service
```
sudo cp opt/reset.py /opt/reset.py
sudo cp etc/systemd/system/reset.service to /etc/systemd/system/reset.service
chmod 644 /lib/systemd/system/reset.service
systemctl enable reset.service
systemctl start reset.service
```
Éste es el circuito esquemático del reinicio por software. Tiene una señal de pull-up a 5v a través de una resistencia de 10k.
![Soft reset](https://github.com/reynico/raspberry-websdr/raw/master/gpio_soft_reset.png)

### Controlador de antena
Esta configuración de WebSDR utiliza un solo receptor RTL-SDR para dos bandas (40/80 metros), crontab toma el control de que banda está funcionando. Así como la longitud de onda no es la misma para las dos bandas, estoy utilizando un pin GPiO para intercambiar entre ellas usando un relé doble polo doble inversor. El pin GPiO3 es controlado por el script check_band.sh
![Circuito esquemático del control de antena](https://github.com/reynico/raspberry-websdr/raw/master/gpio_antenna_control_npn.png)

### WebSDR
- Enviale un email a [Pieter](http://websdr.org/) para obtener una copia de WebSDR.
- Copia el binario websdr-rpi y los archivos de configuración a tu directorio personal (/home/pi/)
- Edita websdr-80m.cfg y websdr-40m.cfg para ajustarlo a tu configuración
- Crea dos Systemd units para controlar websdr y rtl_tcp
```
sudo cp etc/systemd/system/websdr@.service /etc/systemd/system/websdr@.service
sudo cp etc/systemd/system/rtl_tcp@.service /etc/systemd/system/rtl_tcp@.service
```
- Habilita solo rtl_tcp. Websdr es controlado por crontab
```
sudo systemctl enable rtl_tcp@0.service
```

### Cron
- Fabriqué una configuración de crontabl para intercambiar entre las bandas de 40 y 80 metros basada en horarios. Sólo importa las lineas del archivo crontab en tu crontab.

### Control manual
Siempre podrás controlar el cambio de bandas de forma manual. Deshabilita las lineas de cron para evitar cambios automáticos. Luego puedes usar:
```
sudo systemctl stop websdr@40.service
sudo systemctl start websdr@40.service
```
Donde 40 es la banda que quieres recibir. Puedes usar y configurar prácticamente cualquier banda que quieras, siempre y cuando hayas configurado tu archivo websdr-{{banda}}m.cfg
