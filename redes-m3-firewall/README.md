# Redes M3 - Firewall

## Objetivo
Diseñar la segmentación de red y las reglas de firewall de una tienda online ficticia con 5 componentes: sitio web, base de datos, empleados de oficina, empleados remotos y servidor de archivos. Cada componente se ubica en una zona de red (untrusted, screened subnet o trusted) y se definen los flujos permitidos entre zonas, indicando el puerto y el protocolo de cada uno. El diseño sigue el principio de denegar todo por defecto y permitir solo lo necesario.

## Empresa
Tienda online ficticia que vende productos a clientes por internet. Sus 5 componentes son:

- **Sitio web:** la tienda que ven los clientes desde internet, donde navegan los productos y hacen sus pedidos.
- **Base de datos:** guarda la información de clientes y pedidos, y le provee al sitio web los datos para mostrar productos y procesar compras.
- **Empleados de oficina:** trabajan desde equipos dentro de la red interna, navegan por internet y usan el servidor de archivos.
- **Empleados remotos:** cumplen la misma función que los de oficina, pero se conectan desde afuera mediante una VPN client-to-site, que tuneliza la conexión hacia la red de la oficina.
- **Servidor de archivos:** repositorio compartido donde los empleados guardan y usan los archivos de la empresa.

## Zonas de red

| Zona | Qué vive ahí | Por qué |
|---|---|---|
| **Untrusted** | Internet (los clientes) | Red externa, fuera del control de la empresa |
| **Screened subnet** | Sitio web, Servidor VPN | Equipos que internet debe poder alcanzar, pero solo por puertos concretos. Si se comprometen, aún queda un firewall antes de la red interna |
| **Trusted** | Servidor de archivos, Base de datos, Empleados de oficina, Empleados remotos (una vez conectados por VPN) | Red interna, sin acceso directo desde internet |

## Reglas de firewall
Las reglas se evalúan en orden y gana la primera que coincide con el tráfico. El firewall es stateful, así que no se necesitan reglas aparte para las respuestas.

| # | Origen | Destino | Puerto | Acción |
|---|---|---|---|---|
| 1 | Internet (untrusted) | Sitio web (screened subnet) | 443/TCP | Permitir |
| 2 | Sitio web (screened subnet) | Base de datos MS SQL (trusted) | 1433/TCP | Permitir |
| 3 | Empleados de oficina (trusted) | Internet (untrusted) | 443/TCP | Permitir |
| 4 | Empleados de oficina (trusted) | Internet (untrusted) | 53/UDP, 53/TCP | Permitir |
| 5 | Internet (untrusted) | Servidor VPN (screened subnet) | 500/UDP, 4500/UDP, ESP (protocolo 50) | Permitir |
| 6 | Servidor VPN (screened subnet) | Servidor de archivos (trusted) | 445/TCP | Permitir |
| 7 | Cualquiera | Cualquiera | Todos | **Denegar** (y registrar) |