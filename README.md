# Playbook de Ansible para DNS (BIND) y DHCP (dhcpd) en Rocky Linux 9

Este repositorio contiene la solución completa automatizada con **Ansible** para desplegar, configurar y administrar los servicios core de red **DNS (BIND9/named)** y **DHCP (dhcpd)** en servidores **Rocky Linux 9**. El dominio predeterminado configurado es `santiago.gomez.lab`.



## Descripción Completa del Ejercicio

El objetivo de este proyecto es la instalación e integración idempotente y verificable de la infraestructura de red primaria para un entorno de laboratorio o producción interna en la familia RHEL (Rocky Linux 9).

### Componentes y Funcionamiento

1. **DNS (BIND9 - `named`)**:
   - **Autoritativo Internal Zone**: Gestiona la zona directa `santiago.gomez.lab` mapeando nombres de host a direcciones IP (registros `A`).
   - **Zona Inversa (PTR)**: Gestiona la resolución inversa para el bloque IP `192.168.10.0/24` mediante la zona `10.168.192.in-addr.arpa`.
   - **Recursión y Forwarders**: Configurado para resolver peticiones locales y reenviar peticiones a dominios externos utilizando servidores forwarders configurables (por defecto `8.8.8.8` y `1.1.1.1`).
   - **Validación Sintáctica**: Las plantillas Jinja2 incorporan verificaciones automáticas de sintaxis en tiempo de despliegue mediante `named-checkconf` y `named-checkzone`.

2. **DHCP (`dhcpd`)**:
   - **Servidor Autoritativo**: Proporciona direccionamiento dinámico para la subred `192.168.10.0/24`.
   - **Pool de Direcciones**: Entrega un rango de direcciones IP (`192.168.10.100` a `192.168.10.200`).
   - **Opciones de Red**: Configura automáticamente a los clientes con el Gateway por defecto (`192.168.10.1`), el servidor DNS (`192.168.10.10`) y el nombre de dominio (`santiago.gomez.lab`).
   - **Reservas por MAC**: Soporte para asignación de IPs estáticas basadas en la dirección MAC del cliente.
   - **Interfaz Fija**: El demonio se vincula exclusivamente a la interfaz configurada (ej. `eth1`) en `/etc/sysconfig/dhcpd` (`DHCPDARGS`).
   - **Validación Sintáctica**: Valida la sintaxis del archivo de configuración generado mediante `dhcpd -t -cf`.

3. **Seguridad y Cortafuegos**:
   - Apertura automática e idempotente de los puertos requeridos mediante `firewalld`:
     - **DNS**: `53/tcp` y `53/udp`
     - **DHCP**: `67/udp`
   - Los archivos de configuración se despliegan en las rutas estándar (`/etc/named.conf`, `/var/named/`, `/etc/dhcp/dhcpd.conf`) garantizando compatibilidad nativa con **SELinux** en modo *Enforcing*.


## Estructura del Repositorio

```text
.
├── .github/
│   └── workflows/
│       └── ci.yml             # Integración continua (Ansible-Lint)
├── .gitignore                 # Exclusiones de Git
├── README.md                  # Documentación del proyecto
├── Vagrantfile                # Configuración de máquina virtual local (Rocky Linux 9)
├── ansible.cfg                # Configuración global de Ansible
├── group_vars/
│   └── all.yml                # Variables globales (Dominio, IPs, Pools, Reservas)
├── inventory/
│   └── hosts.ini              # Inventario de servidores objetivo
├── roles/
│   ├── dhcpd/                 # Rol de instalación y configuración de DHCP
│   │   ├── defaults/main.yml
│   │   ├── handlers/main.yml
│   │   ├── tasks/main.yml
│   │   └── templates/dhcpd.conf.j2
│   └── dns_bind/              # Rol de instalación y configuración de BIND DNS
│       ├── defaults/main.yml
│       ├── handlers/main.yml
│       ├── tasks/main.yml
│       └── templates/
│           ├── named.conf.j2
│           ├── zone.forward.j2
│           └── zone.reverse.j2
└── site.yml                   # Playbook principal de ejecución
```

---

## Requisitos Previos

- **Nodo de Control (donde ejecutas Ansible)**:
  - Python 3.x
  - Ansible Core >= 2.12
  - Colección `ansible.posix` instalada:
    ```bash
    ansible-galaxy collection install ansible.posix
    ```
- **Para Despliegue Local en Entorno de Laboratorio**:
  - **Vagrant** y **VirtualBox** instalados en tu equipo.
- **Servidor Objetivo (Target)**:
  - Rocky Linux 9 (o distribución compatible RHEL 9).
  - Acceso por SSH con usuario con permisos de `sudo` sin contraseña (o clave SSH configurada).


## Guía de Ejecución

### 1. Ubicación y Carpeta de Trabajo
Todas las operaciones deben ejecutarse **desde la raíz del repositorio**:

```bash
cd /ruta/a/Ansible-Playbook-for-DNS-and-DHCP-on-Rocky-Linux
```

### 2. Iniciar las Máquinas Virtuales de Laboratorio (Vagrant)
El proyecto incluye un entorno de laboratorio multimáquina con Vagrant:
- **`server` (`rocky-infra-01`)**: Servidor con Rocky Linux 9 y la IP fija `192.168.10.10`.
- **`client` (`rocky-client-01`)**: Máquina cliente con Rocky Linux 9 configurada para obtener su IP por DHCP.

Inicia ambas máquinas virtuales ejecutando:

```bash
vagrant up
```

*(O de forma individual: `vagrant up server` / `vagrant up client`)*

### 3. Configurar el Inventario
Revisa el archivo `inventory/hosts.ini` (preconfigurado para el entorno Vagrant local):

```ini
[infra_servers]
rocky-infra-01 ansible_host=192.168.10.10 ansible_user=vagrant
```

### 4. Personalizar Variables (Opcional)
Edita `group_vars/all.yml` si deseas adaptar las variables del dominio o subred:

```yaml
dns_domain: "santiago.gomez.lab"
dns_server_ip: "192.168.10.10"
dhcp_interface: "eth1"
dhcp_range_start: "192.168.10.100"
dhcp_range_end: "192.168.10.200"
```

### 5. Comandos de Ejecución

- **Verificación de Sintaxis y Linting**:
  ```bash
  ansible-lint site.yml
  ```

- **Ejecución en Modo Simulación (Dry-Run / Check mode)**:
  Permite ver qué cambios se realizarían en el servidor sin aplicar nada:
  ```bash
  ansible-playbook -i inventory/hosts.ini site.yml --check --diff
  ```

- **Ejecución Real (Despliegue)**:
  Aplica la configuración completa en el servidor:
  ```bash
  ansible-playbook -i inventory/hosts.ini site.yml
  ```

- **Prueba de Idempotencia**:
  Vuelve a ejecutar el comando anterior. El resultado en el resumen final debe mostrar `changed=0`.

---

## Acceso y Verificación en Múltiples Terminales

Para realizar las pruebas del laboratorio de manera interactiva y en tiempo real, abre dos ventanas o pestañas de terminal independientes.

### Terminal 1: Servidor DNS/DHCP (`rocky-infra-01`)

En la primera terminal, accede por SSH a la máquina virtual del servidor:

```bash
cd /ruta/a/Ansible-Playbook-for-DNS-and-DHCP-on-Rocky-Linux
vagrant ssh server
```

*(O conectándote directamente por IP: `ssh vagrant@192.168.10.10`)*

Dentro del servidor, puedes validar los servicios y monitorear las solicitudes de red en tiempo real:

1. **Verificar el estado de los servicios**:
   ```bash
   systemctl status named dhcpd
   ```

2. **Verificar puertos escuchando (`53` TCP/UDP y `67` UDP)**:
   ```bash
   ss -tulpn | grep -E ':(53|67)'
   ```

3. **Monitorear solicitudes y concesiones DHCP en tiempo real**:
   ```bash
   sudo tail -f /var/log/messages | grep dhcpd
   ```

---

### Terminal 2: Cliente de Red (`rocky-client-01`)

En la segunda terminal, accede por SSH a la máquina virtual del cliente:

```bash
cd /ruta/a/Ansible-Playbook-for-DNS-and-DHCP-on-Rocky-Linux
vagrant ssh client
```

Dentro del cliente, ejecuta las pruebas de solicitud de IP por DHCP y resolución de nombres DNS contra el servidor:

1. **Solicitar / Renovar dirección IP por DHCP**:
   ```bash
   sudo dhclient -v -r eth1   # Liberar IP previa en la interfaz de red privada
   sudo dhclient -v eth1      # Solicitar nueva IP al servidor DHCP (192.168.10.10)
   ```

2. **Verificar IP asignada y servidor DNS configurado**:
   ```bash
   ip a show eth1
   cat /etc/resolv.conf
   ```
   *Debe mostrar una IP dentro del rango del pool `.100 - .200` y `nameserver 192.168.10.10` con `search santiago.gomez.lab`.*

3. **Probar resolución de nombres DNS**:
   ```bash
   dig @192.168.10.10 ns1.santiago.gomez.lab
   dig @192.168.10.10 -x 192.168.10.10
   dig @192.168.10.10 google.com
   ```