# Ansible Playbook for DNS and DHCP on Rocky Linux

Este proyecto contiene una estructura de Ansible Playbook para instalar y configurar servidores **DNS (BIND)** y **DHCP (dhcpd)** en sistemas **Rocky Linux** (RHEL-based).

## Estructura del Proyecto

```text
.
├── .github/
│   └── workflows/
│       └── ci.yml
├── .gitignore
├── README.md
├── ansible.cfg
├── group_vars/
│   └── all.yml
├── inventory/
│   └── hosts.ini
├── roles/
│   ├── dhcpd/
│   │   ├── defaults/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── templates/
│   │       └── dhcpd.conf.j2
│   └── dns_bind/
│       ├── defaults/
│       │   └── main.yml
│       ├── handlers/
│       │   └── main.yml
│       ├── tasks/
│       │   └── main.yml
│       └── templates/
│           ├── named.conf.j2
│           ├── zone.forward.j2
│           └── zone.reverse.j2
└── site.yml
```

## Requisitos Previos

- Ansible instalado en el nodo de control (`pip install ansible` o mediante paquete del sistema).
- Acceso SSH con privilegios de `sudo` a los servidores destino con Rocky Linux.

## Uso

1. Edita el archivo de inventario en `inventory/hosts.ini` con las direcciones IP o nombres de host de tus servidores.
2. Modifica las variables globales en `group_vars/all.yml` según tu infraestructura (dominio, rangos de IP, gateway, etc.).
3. Ejecuta el playbook principal:

```bash
ansible-playbook -i inventory/hosts.ini site.yml
```