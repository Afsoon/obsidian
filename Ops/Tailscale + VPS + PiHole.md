Casi todo ha sido seguir el primer tutorial, excepto un par de cosas que no estaban claras:

- Cuando hacemos los nameservers de Tailscale, hay que poner la IP de la máquina de VPS y seleccionar `Override local dns`. ¿Por qué? Para que cuando estemos usando el móvil con datos, al menos en iOS, la DNS se resuelva a través de nuestro PiHole.

Extras

- Una vez configurado todo tu VPS, puedes cerrar todos los puertos de entrada y tu red privada seguirá funcionando.

Por probar:
- Mullvad VPN para tener una navegación totalmente anónima. 
### Resources
https://0xmachos.com/2021-05-10-Pi-hole-Unbound-and-Tailscale/
https://tailscale.com/kb/1187/install-ubuntu-2204
https://tailscale.com/kb/1028/key-expiry