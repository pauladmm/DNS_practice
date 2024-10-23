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
   Dentro del archivo /etc/bind/named.conf.options se modifica la opcion ndssec-validation a yes.
   Tambien se comenta la escucha de IPv6 por si acaso.
   Se copia el archivo de esta configuracion fuera y se reinicia bind para aplicar los cambios.
   systemctl status named para comprobar su estado.

3. Los servidores permitirán las consultas recursivas sólo a los ordenadores en la red 127.0.0.0/8
   y en la red 192.168.57.0/24, para ello utilizarán la opción de listas de control de acceso o acl.

4. El servidor maestro será tierra.sistema.test y tendrá autoridad sobre la zona directa e inversa.
   Se modifica el archivo /etc/bind/named.conf.local para configurar el servidor como maestro y darle autoridad para la zona directa (sistema.test) e inversa (192.168.57.0/24). Se comprueba que el archivo es correcto:
   named-checkconf /etc/bind/named.conf.local
   Se crean los archivos de las zonas directa (/var/lib/bind/db.sistema.test) e inversa (/var/lib/bind/192.168.57). Se crean en este directorio porque el servicio named (que se usa para la comprobación) dispone permiso para leer en estas carpetas. Para comprobar que la zona es correcta se usa:
   named-checkzone sistema.test. /var/lib/bind/db.sistema.test
   Se reinicia bind para aplicar los cambios.
   Se copian los archivos fuera.

5. El servidor esclavo será venus.sistema.test y tendrá como maestro a tierra.sistema.test.
   Se modifica el archivo /etc/bind/named.conf.local para configurar el servidor DNS esclavo venus y su maestro tierra.
   Se hace una copia del archivo.
   Se reinicia bin para aplicar los cambios.
   En el archivo /etc/named.conf.local del servidor maestro de la zona, se debe indicar que se permitan transferencias de zona hacia el servidor esclavo (allow-transfer), y además se configura para que se notifique a los servidores esclavos los cambios que se produzcan en el maestro (notify).
   También se modifica en el archivo de zona del servidor maestro la existencia de otro servidor DNS para la zona, añadiendo un nuevo registro.
   Para comprobar que está todoo bien consultamos desde el esclavo al maestro por el registro axfr:
   dig @192.168.57.103 sistema.test axfr

6. El tiempo en caché de las respuestas negativas de las zonas (directa e inversa) será de dos horas
   (se pone en segundos).
   Se modifica el archivo maestro correspondiente (db.sistema.test) para cambiar el TTL a 7200s.
   Se reinicia y se copia el archivo.

7. Aquellas consultas que reciba el servidor para la que no está autorizado, deberá reenviarlas
   (forward) al servidor DNS 208.67.222.222 (OpenDNS).
   Se cambiar la configuracion (named.conf.options) del maestro para autorizar el reenvio de solucituudes al servidor DNS dado (activar fowarders).
   Comprobacion:
   named-checkconf /etc/bind/named.conf.local

8. Se configurarán los siguientes alias:
   a. ns1.sistema.test. será un alias de tierra.sistema.test.
   b. ns2.sistema.test. será un alias de venus.sistema.test..
   Se modifica el archivo correspondiente en el maestro (db.sistema.test) para indicar los nuevos CNAME de tierra y venus.
   Se comprueba con:
   dig @localhost alias

9. mail.sistema.test. será un alias de marte.sistema.test.
   Se añade otro CNAME para marte.sistema.test. que corresponde a mail.sistema.test. en el mismo archivo de antes. Se hace la misma comprobacion.

10. El equipo marte.sistema.test. actuará como servidor de correo del dominio de correo
    sistema.test.
