# Configurando VXLAN com nmcli

Guia prático para criar e remover um túnel **VXLAN (VLAN 999)** sobre uma rede L3 existente, usando `nmcli` no Linux.

## O que é VXLAN
VXLAN (Virtual Extensible LAN) é um protocolo de encapsulamento que permite estender redes L2 sobre uma infraestrutura L3. Na prática, ele cria um túnel que transporta quadros Ethernet dentro de pacotes UDP, como se fosse uma "VLAN gigante" que atravessa roteadores.

## Cenário

- **VLAN ID:** `999`
- **Porta VXLAN (destino):** `4789` (padrão IANA)
- **Lado local (notebook):** `192.0.2.10` (exemplo — substitua pelo seu IP real)
- **Lado remoto (peer):** `192.0.2.20` (exemplo — substitua pelo IP do peer)
- **Bridge de overlay:** `br-vxlan-999` (sem IP, apenas L2)
- **Interface VXLAN:** `vxlan-999`


## Pré-requisitos

- Dois hosts com conectividade IP entre si (a rede subjacente, ou *underlay*).
- `nmcli` instalado (NetworkManager).
- Privilégios de root ou `sudo`.
- A porta `4789/UDP` liberada entre os peers.

## Criação

```bash
#!/bin/bash
# Cria VXLAN vinda de uma bridge (cenário VLAN 999) para o notebook
# Substitua os IPs de exemplo pelos do seu ambiente antes de rodar.

# 1. Criar o bridge de overlay (sem IP, só L2)
nmcli connection add type bridge con-name br-vxlan-999 ifname br-vxlan-999 \
    ipv4.method disabled ipv6.method disabled

# 2. Criar a VXLAN e já pendurar no bridge
nmcli connection add type vxlan con-name vxlan-999 ifname vxlan-999 \
    vxlan.id 999 \
    vxlan.local 192.0.2.10 \
    vxlan.remote 192.0.2.20 \
    vxlan.destination-port 4789 \
    master br-vxlan-999

# 3. Subir o bridge (a VXLAN sobe junto como slave)
nmcli connection up br-vxlan-999
```