# Podniková Sieť

## Úvod

Podniková sieť zabezpečuje efektívne a bezpečné pripojenie pre rôzne kategórie používateľov a zariadení. Sieť je rozdelená na segmenty pomocou VLAN podľa funkcií a úrovní prístupu, čím sa zaisťuje izolácia medzi rôznymi skupinami používateľov. 

- **Zamestnanci** majú prístup k serverom, manažmentu a kamerám.
- **Hostia** majú obmedzený prístup iba k internetu, nie do interných systémov.
- **Manažment** má prístup k serverom a kamerám, aby dohliadali na chod spoločnosti a bezpečnostné zariadenia.
- **Kamery** sú izolované, aby sa minimalizovalo riziko zneužitia alebo narušenia ich funkčnosti.

Tento návrh zaisťuje bezpečnosť a poskytuje jednoduchú správu siete, pričom každá kategória používateľov má presne definované práva.

Výsledkom bude sieť nižšie.

![image](https://github.com/spekhy/hal_pcv_podnikova_siet_3/blob/main/siet.png?raw=true)

---

## Jadro

### Použité Elementy

- 1x **Server-PT**.
- 1x **Router-PT**.
- 2x **WRT300N**.
- 5x **Switch-PT**.
- 5x **PC-PT / Laptop-PT**.
- 2x **SmartLED**.

### Vytvorenie VLAN

Každá VLAN bola nakonfigurovaná tak, aby poskytovala izoláciu a špecifické IP adresné rozsahy. Tieto VLAN sú základom pre oddelenie siete a efektívne prideľovanie zdrojov.

```plaintext
enable
configure terminal

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
```

### Vytvorenie ACL (Access Control List)

Pravidlá ACL definujú presné povolenia a obmedzenia prístupu medzi VLAN, čím zabezpečujú kontrolovaný tok dát. 

#### VLAN 10 - Zamestnanci

Zamestnanci majú prístup do manažmentu, kamier a serverov, no sú izolovaní od ostatných VLAN.

```plaintext
access-list 101 permit ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
access-list 101 permit ip 192.168.10.0 0.0.0.255 192.168.40.0 0.0.0.255
access-list 101 permit ip 192.168.10.0 0.0.0.255 192.168.50.0 0.0.0.255
access-list 101 deny ip 192.168.10.0 0.0.0.255 any
```

#### VLAN 20 - Hostia

Hostia majú prístup výhradne k internetu, čím sa znižuje riziko neautorizovaného prístupu.

```plaintext
access-list 102 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
access-list 102 deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
access-list 102 deny ip 192.168.20.0 0.0.0.255 192.168.40.0 0.0.0.255
access-list 102 deny ip 192.168.20.0 0.0.0.255 192.168.50.0 0.0.0.255
access-list 102 permit ip 192.168.20.0 0.0.0.255 any
```

### Aplikácia ACL na porty

Každé pravidlo ACL bolo aplikované na príslušné porty na základe ich funkcie a pripojenia.

```plaintext
interface FastEthernet0/0
ip access-group 101 in

interface FastEthernet1/0
ip access-group 102 in
```

Potom môžeme exitnúť config commandom exit, a zobraziť si aktuálné ACL listy pomocou show access-lists

![image](https://github.com/spekhy/hal_pcv_podnikova_siet_3/blob/main/access-lists.png?raw=true)

### Konfigurácia DHCP pre siete

Router vo VLAN 50 je nakonfigurovaný ako DHCP server, čím sa zabezpečuje dynamické prideľovanie IP adries pre všetky zariadenia v sieti.

### Priradenie IP adries

- **Server PT:** 192.168.50.2 (VLAN 50)
- **Zamestnanci:**
  - Laptop0: 192.168.10.2 (VLAN 10)
  - PC1: 192.168.10.3 (VLAN 10)
- **Management:**
  - Laptop1: 192.168.30.2 (VLAN 30)
  - PC2: 192.168.30.3 (VLAN 30)
- **Kamery (IoT):**
  - Smart LED IoT4: 192.168.40.2 (VLAN 40)
  - Smart LED IoT4(1): 192.168.40.3 (VLAN 40)

### Prehľad VLAN

| VLAN ID | Názov         | IP Rozsah        | Použiteľné adresy       | Mask          |
|---------|---------------|------------------|-------------------------|---------------|
| 10      | Zamestnanci   | 192.168.10.0/24 | 192.168.10.2-192.168.10.254 | 255.255.255.0 |
| 20      | Hostia        | 192.168.20.0/24 | 192.168.20.2-192.168.20.254 | 255.255.255.0 |
| 30      | Management    | 192.168.30.0/24 | 192.168.30.2-192.168.30.254 | 255.255.255.0 |
| 40      | Kamery        | 192.168.40.0/24 | 192.168.40.2-192.168.40.254 | 255.255.255.0 |
| 50      | Servery       | 192.168.50.0/24 | 192.168.50.2-192.168.50.254 | 255.255.255.0 |

---

## Záver

### Výhody navrhovanej siete
- **Bezpečnosť:** Jasne definované segmenty a prístupové pravidlá chránia citlivé údaje a zariadenia pred neoprávneným prístupom. 
- **Efektívnosť:** Segmentácia znižuje prenosové zaťaženie, čo vedie k vyššej rýchlosti prenosu a lepšiemu výkonu celej siete.
- **Jednoduchá správa:** Centrálne riadené VLAN a ACL výrazne znižujú čas potrebný na správu a zlepšujú prehľadnosť.

### Nevýhody a možné zlepšenia
- **Obmedzená flexibilita hostí:** Hostia nemajú prístup ani k lokálnym tlačiarňam či projektorom, čo môže byť prekážkou pre niektoré aplikácie.
- **Závislosť na ACL:** Rozsiahle ACL môžu byť náročné na údržbu a aktualizáciu, čo zvyšuje riziko chyby.
- **Chýbajúca redundancia:** Návrh neobsahuje žiadne záložné linky ani ochranu proti výpadkom, čo môže ohroziť dostupnosť siete v prípade zlyhania.

### Návrhy na zlepšenie

Navrhovaná podniková sieť ponúka pevný základ pre bezpečnú a efektívnu prevádzku. Zároveň však otvára priestor na ďalšie zlepšenia, ktoré môžu zvýšiť jej funkčnosť a spoľahlivosť v dlhodobom horizonte.

- **Monitoring:** Implementácia nástroja na monitorovanie siete na detekciu a prevenciu hrozieb.
- **Redundancia:** Zavedenie záložných liniek a záložných DHCP serverov pre zvýšenie spoľahlivosti.
- **Vylepšenie pre hostí:** Poskytnutie prístupu k vybraným lokálnym zdrojom, ako sú tlačiarne alebo prezentačné zariadenia, bez ohrozenia bezpečnosti.
