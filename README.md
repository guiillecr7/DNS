# Projecte de Consultoria DNS – TechCloud Solutions
**Client:** Innovació Tecnològica del Maresme S.L.  
**Autor:** [El teu nom i cognoms]  
**Data:** Novembre 2026  

---

## 1. Fase Teòrica: Píndola Formativa i Recursos Educatius
* **Enllaç al vídeo de la píndola formativa:** [ENGANXA_AQUÍ_EL_TEU_ENLLAÇ_DE_YOUTUBE_O_DRIVE]
* **Descripció del recurs:** Píndola audiovisual de 5 a 10 minuts recolzada en un mapa conceptual visual on s'explica el funcionament teòric del protocol DNS (arquitectura, jerarquia en arbre, tipus de consultes, zones i resolució local mDNS).

---

## 2. Configuració de l'Entorn de Treball i Xarxa (Zorin OS)

La màquina virtual s'ha configurat amb dues interfícies de xarxa segons els requisits del projecte:
1. **Interfície 1 (`enp0s3`):** Mode NAT per garantir la sortida a Internet.
2. **Interfície 2 (`enp0s8`):** Mode Adaptador pont (Bridged) amb IP estàtica per a la resolució a la xarxa local.

![Configuració de Xarxa a VirtualBox](01_xarxa_virtualbox.png)

### Comprovació de les interfícies de xarxa (`ip a`)
![Execució de la comanda ip a](02_ip_a.png)

---

## 3. Fase Pràctica: Auditoria de Resolució de Noms (CLI)

### Part A. Diagnosi avançada amb `dig`

#### 1. Consulta bàsica de registre A (`dig tecnocampus.cat A`)
![Consulta dig A](03_dig_A.png)
* **Anàlisi tècnica:**
  * **IPs de resposta:** S'obtenen les adreces IPv4 associades al domini (ex. `108.157.113.59`, `108.157.113.83`, `108.157.113.8`, `108.157.113.90`).
  * **TTL (Time To Live):** El temps de vida de la memòria cau és de 60 segons, el que garanteix un balanç de càrrega i actualització ràpida a nivell de CDN/proxy.
  * **Servidor DNS que respon:** La consulta s'ha resolt a través del resolutor local `127.0.0.53#53` (`systemd-resolved`).

#### 2. Consulta de servidors de noms (`dig tecnocampus.cat NS`)
![Consulta dig NS](04_dig_NS.png)
* **Anàlisi tècnica:**
  * Els servidors de noms (Name Servers) autoritatius assignats al domini `tecnocampus.cat` pertanyen a la infraestructura d'Amazon Route 53:
    * `ns-130.awsdns-16.com`
    * `ns-535.awsdns-02.net`
    * `ns-1071.awsdns-05.org`
    * `ns-1689.awsdns-19.co.uk`

#### 3. Consulta detallada SOA (`dig tecnocampus.cat SOA`)
![Consulta dig SOA](05_dig_SOA.png)
* **Anàlisi tècnica:**
  * **Correu de l'administrador (`rname`):** `awsdns-hostmaster.amazon.com` (en el registre DNS es representa com `awsdns-hostmaster.amazon.com`, on el primer punt equival a la `@`).
  * **Número de sèrie (`serial`):** `1`. Aquest número s'utilitza per a la sincronització de zones entre els servidors primaris i secundaris.

#### 4. Consulta de resolució inversa (`dig -x <IP>`)
![Consulta dig PTR](06_dig_PTR.png)
* **Anàlisi tècnica:**
  * La consulta utilitza el domini especial `.in-addr.arpa` per mapejar una adreça IP IPv4 cap al seu nom de domini (registre PTR). Permet verificar la legitimitat de la IP i comprovar si està assignada a la xarxa de distribució de continguts (CDN CloudFront / AWS).

---

### Part B. Comprovació de resolució amb `nslookup` (Mode interactiu)

#### 1. Consulta bàsica no autoritativa
![nslookup no autoritatiu](07_nslookup_no_autoritatiu.png)
* **Anàlisi tècnica:**
  * En consultar `tecnocampus.cat` des del servidor per defecte de la màquina local, la consola mostra l'indicador **"Non-authoritative answer"** (resposta no autoritativa). Això s'esdeven perquè la informació s'ha obtingut des de la memòria cau (cache) d'un servidor intermediari o resolutor local, i no directament de la font oficial del domini.

#### 2. Consulta autoritativa directa (`server 205.251.192.130`)
![nslookup autoritatiu](08_nslookup_autoritatiu.png)
* **Anàlisi tècnica:**
  * En canviar el servidor de consulta directament a la IP d'un dels servidors autoritatius d'AWS (`205.251.192.130`), la resposta prové de la font primària oficial. Per aquest motiu, l'indicador "Non-authoritative answer" **desapareix**, confirmant que la dada és 100% directa de l'autoritat del domini.

---

### Part C. Resolucions locals i protocol mDNS

#### 1. Comprovació de nom d'amfitrió i ping local (`ping <hostname>.local`)
![mDNS ping local](09_mdns_ping.png)

#### 2. Anàlisi del protocol mDNS (Multicast DNS)
* **Funcionament:** El protocol mDNS permet resoldre noms d'amfitrió a adreces IP en xarxes locals petites sense la necessitat de tenir un servidor DNS centralitzat ni modificar el fitxer `/etc/hosts`. Funciona enviant missatges Multicast IP a tots els equips de la subxarxa.
* **Port utilitzat:** Utilitza el port **UDP 5353** (adreça multicast `224.0.0.251` en IPv4).
* **Avantatges:** Aporta facilitat de configuració (Plug and Play), ideal per a entorns domèstics o petites oficines on els dispositius s'han de descobrir entre ells (per exemple, impressores o equips de treball) sense dependre d'infraestructura DNS dedicada.

---

## 4. Declaració de Fonts i Eines Consultades
* Documentació oficial de comandes de xarxa en Linux (`man dig`, `man nslookup`).
* Documentació d'Amazon Route 53 i funcionament de registres DNS.
* Assistència d'eines d'IA generativa (Gemini) utilitzada com a suport per a la millora de la redactat tècnic, l'estructuració de la guia en format Markdown i l'assessorament en el workflow de l'activitat.
