# Podnikova siet

Zadanie:
Podniková sieť, v prevedení 2 SSID wi-fi (zames nanci, hostia) a 4 káblove siete (servers,
zamesnanci, managment, kamery) , pričom zamestnanci, musia byt oddelene ale majú
mať prístup k serverom a sieti management aj do siete kamery

vytvorenie vlanov

enable
configure terminal

# Create VLAN interfaces for Inter-VLAN routing
interface vlan 10
name zamestnanci
ip address 192.168.10.1 255.255.255.0
no shutdown

interface vlan 20
name hostia
ip address 192.168.20.1 255.255.255.0
no shutdown

interface vlan 30
name managment
ip address 192.168.30.1 255.255.255.0
no shutdown

interface vlan 40
name kamery
ip address 192.168.40.1 255.255.255.0
no shutdown

interface vlan 50
name servery
ip address 192.168.50.1 255.255.255.0
no shutdown



vytvorenie funkcneho acl-ka

vlan 10 zamestnanci

access-list 101 permit ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
access-list 101 permit ip 192.168.10.0 0.0.0.255 192.168.40.0 0.0.0.255
access-list 101 permit ip 192.168.10.0 0.0.0.255 192.168.50.0 0.0.0.255
access-list 101 deny ip 192.168.10.0 0.0.0.255 any

vlan 20 hostia

access-list 102 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
access-list 102 deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
access-list 102 deny ip 192.168.20.0 0.0.0.255 192.168.40.0 0.0.0.255
access-list 102 deny ip 192.168.20.0 0.0.0.255 192.168.50.0 0.0.0.255
access-list 102 permit ip 192.168.20.0 0.0.0.255 any

este pre aplikovanie tych acceslistov, na prislusne porty

interface FastEthernet0/0
ip access-group 101 in

interface FastEthernet1/0
ip access-group 102 in





- router pt: 192.168.50.1 VLAN50 bude ako dhcp pre vsetky siete
- server pt: 192.168.50.2 VLAN50
- zanestnanci: 192.168.10.1 VLAN10
- hostia: 192.168.20.1 VLAN20
- laptop0 (zamestnanci): 192.168.10.2 VLAN10
- pc1 (zamestnanci): 192.168.10.3 VLAN10
- laptop1 (managment): 192.168.30.2 VLAN30
- pc2 (managment): 192.168.30.3 VLAN30
- iot kamery VLAN40:
    - Smart LED IoT4: **192.168.40.2**
    - Smart LED IoT4(1): **192.168.40.3**
- VLANY, priklad rozsahu
    
    
    | VLAN 10 | Zamestnanci | 192.168.10.0/24 | 192.168.10.2-192.168.10.254 | 255.255.255.0 |
    | --- | --- | --- | --- | --- |
    
    | VLAN 20 | Hostia | 192.168.20.0/24 | 192.168.20.2-192.168.20.254 | 255.255.255.0 |
    | --- | --- | --- | --- | --- |
    
    | VLAN 30 | Management | 192.168.30.0/24 | 192.168.30.2-192.168.30.254 | 255.255.255.0 |
    | --- | --- | --- | --- | --- |
    
    | VLAN 40 | Kamery | 192.168.40.0/24 | 192.168.40.2-192.168.40.254 | 255.255.255.0 |
    | --- | --- | --- | --- | --- |
    
    | VLAN 50 | Servery | 192.168.50.0/24 | 192.168.50.2-192.168.50.254 | 255.255.255.0 |
    | --- | --- | --- | --- | --- |
- **Zamestnanci (VLAN 10)** môžu pristupovať do:
    - VLAN 30 (Management)
    - VLAN 40 (Kamery)
    - VLAN 50 (Servery)
- **Hostia (VLAN 20)** môžu pristupovať iba do internetu, žiadny prístup do interných sietí.

![image](https://raw.githubusercontent.com/spekhy/hal_pcv_podnikova_siet_3/refs/heads/main/access-lists.png)
