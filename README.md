# Practica-01-09-IAW Jose Francisco León López
##  Para esta práctica vamos a necesitar dos instancias de Amazon: Una va a ser la máquina del frontend y otra la del backend.
## Su configuración en amazon se deberían de ver así:
### La configuración de la máquina del frontend es esta
![1](https://github.com/JoseFco04/practica-01-09-iaw/assets/145347148/f57015c3-913f-43bf-b48f-5d46b79bca13)
### Y su grupo de seguridad es este:
![Captura de pantalla 2023-12-08 212026](https://github.com/JoseFco04/practica-01-09-iaw/assets/145347148/e6ae71af-6a57-4993-a7ac-ebed39b15230)

### La configuarción de la máquina del backend es esta :
![1](https://github.com/JoseFco04/practica-01-09-iaw/assets/145347148/22a730aa-27aa-43a5-ba6d-3ac92c5d45d1)

### Y su grupo de seguridad es este:
![2](https://github.com/JoseFco04/practica-01-09-iaw/assets/145347148/4decf889-2ad6-42ca-bcde-c9c5bc3f1f16)

### El objetivo de la práctica es automatizar la instalación y configuración de una página web a través de dos máquinas virtuales de ubuntu. Una de ellas será la que controlará la parte del frontend y otra la del backend.

### Para esta práctica debemos separar los scripts de instalación en dos un install_lamp y un deploy para el frontend y otros distintos para el del backend.

### Aparte de estos scripts tenemos que tener el script del lets encrypt que ya hemos usado en prácticas anteriores, un .env con las variables que necesitamos para instalar la aplicación web que debería verse parecido a este:
