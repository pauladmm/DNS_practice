# DNS_practice

This practice has a main DNS server, a secondary DNS server with a zone and reverse zone.

First of all, we create virtual machine with the following features:

|venus| Debian texto venus.sistema.test .102

|tierra| Debian texto tierra.sistema.test .103
and the host's IP is 192.168.57.0/24

1. Activa solamente la escucha del servidor para el protocolo IPv4.
   Se modifican los archivos de configuración de bind9 (en ambas máquinas), que normalmente se encuentra en /etc/default/named
   Se activa la escucha modificando el archivo de la siguiente manera:
   OPTIONS = "-u bind -4"

   Se hacen copias de ambos archivos de configuracion fuera de las mv.
   Se reinicia al servicio DNS de BIND para que los cambios se apliquen con sudo systemctl restart bind9.
   Se comprueba que hay solo escucha de ipv4 con sudo ss -tuln | grep :53

2. Establecer la opción dnssec-validation a yes

3. Los servidores permitirán las consultas recursivas sólo a los ordenadores en la red 127.0.0.0/8
   y en la red 192.168.57.0/24, para ello utilizarán la opción de listas de control de acceso o acl.

4. El servidor maestro será tierra.sistema.test y tendrá autoridad sobre la zona directa e inversa.

5. El servidor esclavo será venus.sistema.test y tendrá como maestro a tierra.sistema.test.

6. El tiempo en caché de las respuestas negativas de las zonas (directa e inversa) será de dos horas
   (se pone en segundos).

7. Aquellas consultas que reciba el servidor para la que no está autorizado, deberá reenviarlas
   (forward) al servidor DNS 208.67.222.222 (OpenDNS).

8. Se configurarán los siguientes alias:
   a. ns1.sistema.test. será un alias de tierra.sistema.test.
   b. ns2.sistema.test. será un alias de venus.sistema.test..

9. mail.sistema.test. será un alias de marte.sistema.test.

10. El equipo marte.sistema.test. actuará como servidor de correo del dominio de correo
    sistema.test.
