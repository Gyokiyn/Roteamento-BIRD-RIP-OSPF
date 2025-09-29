# Trabalho de Roteamento - BIRD | RIP & OSPF

## Descrição do Projeto
Trabalho acadêmico da disciplina de Redes de Computadores: Internetworking, Roteamento e Transmissão. Comparando os protocolos de roteamento RIP e OSPF utilizando o BIRD em uma topologia de 3 roteadores.

### BIRD (BIRD Internet Routing Daemon)

O **BIRD** é um daemon de roteamento de código aberto para sistemas Unix que implementa diversos protocolos de roteamento.

Foi escolhido devido a sua implementação simples e rapida.

**Versão utilizada:** BIRD 2.14

### Topologia da Rede

[GAR1 - Router 1] ←→ [GAR2 - Router 2] ←→ [GAR3 - Router 3]

### Especificações das VMs
- **Sistema Operacional:** Ubuntu Server 22.04 LTS
- **Memória RAM:** 4GB cada VM
- **Armazenamento:** 25GB cada VM
- **Processadores:** 3 vCPUs cada

## Configuração dos Adaptadores de Rede

### GAR1 (Router 1)
- **Adaptador 1 (enp0s3):** NAT/Bridge - DHCP (Internet)
- **Adaptador 2 (enp0s8):** Rede Interna - `192.168.12.1/24`

### GAR2 (Router 2) 
- **Adaptador 1 (enp0s3):** NAT/Bridge - DHCP (Internet)
- **Adaptador 2 (enp0s8):** Rede Interna - `192.168.12.2/24`
- **Adaptador 3 (enp0s9):** Rede Interna - `192.168.23.2/24`

### GAR3 (Router 3)
- **Adaptador 1 (enp0s3):** NAT/Bridge - DHCP (Internet)
- **Adaptador 2 (enp0s8):** Rede Interna - `192.168.23.3/24`

## Como realizar o trabalho

- Aplique as configurações Netplan de cada roteador
- Instale o BIRD e habilite roteamento IP
- Aplique as Configuração do BIRD - RIP ou OSPF
- Teste as configurações
- Faça a coleta de metricas

## Configuração Netplan.
### Exemplo GAR1 (`/etc/netplan/00-installer-config.yaml`)
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      addresses: [192.168.12.1/24]
```

#### Aplicar configuração
```
sudo netplan apply
```

## Instalação e Configuração do BIRD
### Instalação 
```
sudo apt update
sudo apt install bird2 -y
bird --version
```

### Habilitar Roteamento IP
```
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Aplicar as configurações - RIP ou OSPF
```
sudo ls -la /etc/bird/
```

### Exemplo GAR1 (`sudo nano /etc/bird/bird.conf`)
```
# Apagar tudo e colar:
# Router 1 - RIP Configuration
router id 1.1.1.1;

protocol device {
    scan time 10;
}

protocol direct {
    ipv4;
}

protocol kernel {
    ipv4 {
        export all;
        import all;
    };
    scan time 20;
}

protocol rip {
    ipv4 {
        export all;
        import all;
    };
    
    interface "enp0s8" {
        update time 30;
        timeout time 180;
        garbage time 60;
    };
}
```

### Aplicar configuração e iniciar o BIRD
```
sudo bird -p
sudo systemctl enable bird
sudo systemctl start bird
```

## Teste as configurações
### Comandos BIRD

```
sudo birdc show status
sudo birdc show protocols
sudo birdc show route
```
### Teste de Conectividade. Exemplo GAR1

```
ping -c 3 192.168.12.2
ping -c 3 192.168.23.3 

ip route
traceroute 192.168.23.3
```
## Utilize scripts de Métricas. 
```
nano rip_metrics.sh
```

```
#!/bin/bash
echo "=== BIRD RIP Metrics ==="
date

echo "BIRD Status:"
sudo birdc show status

echo ""
echo "RIP Protocol Status:"
sudo birdc show protocols all rip1

echo ""
echo "RIP Routes:"
sudo birdc show route protocol rip1

echo ""
echo "All Routes:"
sudo birdc show route

echo ""
echo "Routing table size:"
sudo birdc show route | wc -l

echo ""
echo "System routes:"
ip route show | grep -v "169.254"

echo ""
echo "Convergence test - ping end-to-end:"
if [ "$(hostname)" = "GAR1" ]; then
    time ping -c 1 192.168.23.3
elif [ "$(hostname)" = "GAR3" ]; then
    time ping -c 1 192.168.12.1
else
    time ping -c 1 192.168.12.1
fi
```

```
chmod +x rip_metrics.sh
./rip_metrics.sh > rip_results.txt
```

## Autor
Rômulo Lima Siqueira - Trabalho de Redes de Computadores

Universidade: Unisinos POA

Disciplina: Redes de Computadores - Internetworking, Roteamento e Transmissão

