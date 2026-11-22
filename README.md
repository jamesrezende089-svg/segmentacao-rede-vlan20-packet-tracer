# Segmentação de Rede e Isolamento de Broadcast com VLAN (Cisco Packet Tracer)

Este projeto demonstra na prática o planejamento de endereçamento IPv4 (`192.168.10.0/24`), a configuração de gateway em roteador Cisco, a segmentação lógica de rede em Camada 2 utilizando **VLAN** em um switch gerenciável e a validação do isolamento de domínios de broadcast no **Cisco Packet Tracer**.

---

## 🎯 Objetivo do Laboratório

* Planejar e calcular a faixa de endereçamento IPv4 (`192.168.10.0/24`) identificando endereço de rede, gateway, hosts válidos e broadcast.
* Comprovar o comportamento de um **domínio de broadcast único** antes da segmentação.
* Criar a **VLAN 20 (`ISOLADA`)** no switch e associar portas específicas em modo *access*.
* Validar o isolamento de tráfego unicast (`ping` / `tracert`) e a restrição de broadcast entre a VLAN padrão (`VLAN 1`) e a `VLAN 20`.

---

## 🧮 1. Planejamento e Lógica de Endereçamento IPv4

Na sub-rede **`192.168.10.0/24`** (máscara **`255.255.255.0`**), os 24 primeiros bits identificam a rede e os 8 últimos bits ficam livres para os dispositivos ($2^8 = 256$ endereços no total).

Aplicando a regra prática para delimitar a faixa utilizável:
* **Endereço de Rede:** `192.168.10.0` (reservado)
* **Primeiro Host válido (`Rede + 1`):** `192.168.10.1` (atribuído ao gateway da rede)
* **Último Host válido (`Broadcast - 1`):** `192.168.10.254`
* **Endereço de Broadcast:** `192.168.10.255` (reservado)
* **Total de Hosts utilizáveis:** $2^8 - 2 = \mathbf{254}$ endereços disponíveis.

![Lógica de Endereçamento IPv4](01-logica-enderecamento-ipv4.png)

### Tabela de Alocação dos Dispositivos

| Dispositivo | Interface | Endereço IP | Máscara de Sub-rede | Gateway Padrão | VLAN |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Router0 (2911)** | `GigabitEthernet0/0` | `192.168.10.1` | `255.255.255.0` | — | — |
| **PC0** | `FastEthernet0` | `192.168.10.2` | `255.255.255.0` | `192.168.10.1` | VLAN 1 (default) |
| **PC1** | `FastEthernet0` | `192.168.10.3` | `255.255.255.0` | `192.168.10.1` | VLAN 1 (default) |
| **PC2** | `FastEthernet0` | `192.168.10.4` | `255.255.255.0` | `192.168.10.1` | VLAN 1 (default) |
| **PC3** | `FastEthernet0` (`Switch0 Fa0/6`) | `192.168.10.5` | `255.255.255.0` | `192.168.10.1` | **VLAN 20 (ISOLADA)** |
| **PC4** | `FastEthernet0` (`Switch0 Fa0/2`) | `192.168.10.6` | `255.255.255.0` | `192.168.10.1` | **VLAN 20 (ISOLADA)** |

---

## 🗺️ 2. Topologia Lógica

A topologia conta com um roteador **Cisco 2911**, um switch **Cisco 2960-24TT** e cinco estações de trabalho (`PC0` a `PC4`). Os hosts destacados em verde (`PC3` e `PC4`) foram isolados logicamente na **VLAN 20**.

![Topologia da Rede](02-topologia-vlan20.png)

---

## 🔍 3. Comportamento Antes da Segmentação (Domínio Único)

Antes da criação da VLAN 20, todos os dispositivos compartilhavam o mesmo domínio de broadcast (`VLAN 1`). Ao disparar um `ping` para o endereço de broadcast (`192.168.10.255`) a partir do **PC0**, todos os hosts ativos da rede responderam simultaneamente:

![Teste de Broadcast Antes da VLAN](03-teste-broadcast-antes-vlan.%20ping%20192.168.10.255%20no%20PC0.png)

---

## ⚙️ 4. Configurações Aplicadas (CLI)

### Router0 (Gateway da Rede)

```ios
enable
configure terminal
interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
end
write
