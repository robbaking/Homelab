## Homelab

Ett dokumenterat homelab-projekt byggt på Proxmox VE. Målet var att
samla alla smarta hemenheter under Home Assistant och köra stödtjänster
som DNS-filtrering parallellt, utan att de påverkar varandra.

---

## Tjänster som körs

AdGuard Home körs som LXC-container och hanterar DNS-filtrering och
reklamblockning för hela nätverket.

Homepage körs som LXC-container och är en samlad startpage för alla
tjänster i homelabbet.

Hermes körs som LXC-container.

Home Assistant körs som QEMU VM och hanterar all hemautomation,
Zigbee-enheter och smarta produkter.

---

## Proxmox VE — Virtualiseringsplattform

### Varför

I dagens samhälle lever många med smarta produkter runt omkring sig,
även kallat IoT (Internet of Things). Problemet är att varje produkt
ofta kräver en egen app — om man inte köpt allt från samma tillverkare.
Målet var att samla allt under ett tak via Home Assistant, så man kan
kontrollera och övervaka alla enheter på ett ställe.

För att kunna köra Home Assistant och andra tjänster parallellt utan
att de påverkar varandra behövdes en hypervisor. Valet föll på Proxmox
VE eftersom det är gratis, open source och byggt på Debian Linux.

---

### Resa från Raspberry Pi till PC-server

Allt startade på en Raspberry Pi 3B med Home Assistant OS. Det
fungerade till en början, men när jag ville testa fler tjänster
parallellt kraschade Pi:n under lasten. Hårdvaran räckte inte till.

Jag ville även sätta upp AdGuard Home för DNS-filtrering och
reklamblockning. Det visade sig vara ett problem av två anledningar.
Dels orkade inte Pi:n köra båda tjänsterna samtidigt, dels stödde
inte min dåvarande router att sätta separat DNS per WiFi-nätverk.
Det innebar att om AdGuard aktiverades blockerades även enheter som
inte skulle påverkas, till exempel 2.4GHz-kameror.

Lösningen blev att bygga om en gammal spelدatorn till dedikerad
server och byta till UniFi som nätverksutrustning. Med UniFi kunde
jag separera WiFi-nätverken och styra vilka enheter som använder
AdGuard som DNS och vilka som inte gör det.

Raspberry Pi 3B körde ARM Cortex-A53 med 1GB RAM och kraschade
under last. PC-servern kör AMD Ryzen 5 1400 med 15GB RAM och
kör fyra tjänster stabilt.

---

### Nätverkskonfiguration

Servern ansluter till nätverket via kabel (enp30s0). En Linux
bridge (vmbr0) är skapad ovanpå kabelanslutningen — det är via
den alla LXC-containers och VMs kommunicerar med varandra och
med resten av nätverket.

En USB WiFi-dongel är synlig i Proxmox som wlan0 men används
inte av hosten själv. Den är passerad direkt till Home Assistant
VM så att HA kan kommunicera med 2.4GHz-enheter.

Proxmox har inga egna firewall-regler konfigurerade. All
nätverkssäkerhet hanteras på switch- och routernivå via UniFi,
där servern är kopplad till en Layer 3-switch med PoE.

enp30s0 är ett inbyggt nätverkskort utan IP, dess enda funktion
är att vara slave till vmbr0.

vmbr0 är en Linux bridge med IP 192.168.1.100/24 och gateway
192.168.1.1, det är denna alla VMs och containers använder.

wlan0 är en USB WiFi-dongel, passerad till Home Assistant VM
för 2.4GHz-kommunikation.

---

### Problem vi stötte på

**Problem:** Raspberry Pi 3B kraschade när Home Assistant och
AdGuard kördes samtidigt.
**Lösning:** Uppgraderade till en PC-server med tillräckligt
med RAM och CPU för att virtualisera flera tjänster parallellt.

**Problem:** AdGuard fungerade inte korrekt eftersom routern
inte stödde separat DNS per nätverk. Det gick inte att skilja
på vilka enheter som skulle använda AdGuard och vilka som inte
skulle det.
**Lösning:** Bytte till UniFi som nätverksutrustning vilket
möjliggjorde separata WiFi-nätverk med individuella
DNS-inställningar.

**Problem:** Proxmox stöder inte Linux bridges direkt på WiFi,
vilket krävdes för att VMs och containers ska kunna kommunicera
med nätverket.
**Lösning:** Kopplade servern med kabel till switchen och
skapade bridge på kabelanslutningen istället.

**Problem:** Home Assistant behövde åtkomst till USB-enheter
för att fungera fullt ut.
**Lösning:** Körde Home Assistant som QEMU VM istället för
LXC-container eftersom USB-passthrough bara fungerar på QEMU.

---

### Hårdvarukonfiguration — Home Assistant VM (100)

Home Assistant körs som en QEMU VM med följande specifikationer:

RAM är satt till 2GB allokerat med 4GB som max.
CPU är en kärna med x86-64-v2-AES.
Disken är 32GB via VirtIO SCSI på local-lvm.
Nätverket går via VirtIO kopplat till vmbr0.
BIOS är UEFI via OVMF.

Två USB-enheter är passerade direkt till VM:en från hosten:

USB WiFi-dongel hanterar 2.4GHz-kommunikation med smarta
enheter i hemmet.

Sonoff Zigbee-stick hanterar Zigbee-protokollet för enheter
som smarta lampor och sensorer.

---

### Hur gick man vidare

1. Testade Home Assistant OS på Raspberry Pi 3B
2. Pi:n kraschade under last — hårdvaran räckte inte till
3. Byggde om gammal spelدatorn till dedikerad server
4. Installerade Proxmox VE 8 på bare metal
5. Konfigurerade storage-pooler, local, local-lvm och data
6. Skapade Linux bridge vmbr0 för intern nätverkskommunikation
7. Satte upp LXC-containers för lättviktstjänster
8. Satte upp QEMU VM för Home Assistant med USB-passthrough
9. Bytte till UniFi för nätverksutrustning
10. Löste AdGuard-problemet via separata WiFi-nätverk i UniFi

---

## Skärmdumpar

### Proxmox
![Proxmox dashboard](screenshots/proxmox.png)

### Home Assistant
![Home Assistant dashboard](screenshots/homeassistant.png)
