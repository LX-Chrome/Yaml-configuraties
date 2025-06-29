# Cloud Services Infrastructure Automation - Complete Setup Tutorial

Dit repository bevat de Ansible configuraties voor het automatisch uitrollen van cloud klanten op Linux servers en VyOS routers.

## COMPLETE SETUP TUTORIAL - STAP VOOR STAP

### FASE 1: VM SETUP IN VMWARE WORKSTATION

#### Stap 1: VM Netwerk Configuratie
1. **Open VMware Workstation**
2. **Voor elke VM (Linux server EN VyOS router):**
   - Rechtsklik VM → Settings → Network Adapter
   - Selecteer **NAT** of **Bridged** (beide VMs moeten hetzelfde gebruiken)
   - Klik OK

#### Stap 2: Linux Server Setup
1. **Start je Linux VM**
2. **Controleer netwerk configuratie:**
   ```bash
   ip addr show
   ```
   - Noteer het IP-adres (bijv. 192.168.137.100)
   - Noteer het netwerk (bijv. 192.168.137.0/24)

3. **Installeer Ansible:**
   ```bash
   # Ubuntu/Debian:
   sudo apt update
   sudo apt install ansible python3-pip git -y
   
   # CentOS/RHEL:
   sudo yum install epel-release -y
   sudo yum install ansible python3-pip git -y
   ```

4. **Controleer Ansible installatie:**
   ```bash
   ansible --version
   ```

### FASE 2: VYOS ROUTER CONFIGURATIE

#### Stap 3: VyOS Basis Setup
1. **Start VyOS VM**
2. **Login met standaard credentials:**
   - Username: `vyos`
   - Password: `vyos`

#### Stap 4: VyOS IP Configuratie
1. **Ga naar configuratie modus:**
   ```bash
   configure
   ```

2. **Configureer IP adres (pas aan naar jouw netwerk):**
   ```bash
   # Als je Linux server 192.168.137.100 heeft, gebruik dan:
   set interfaces ethernet eth0 address 192.168.137.101/24
   set interfaces ethernet eth0 description 'Management Interface'
   ```

3. **Configureer gateway:**
   ```bash
   # Gateway is meestal .1 van je netwerk
   set protocols static route 0.0.0.0/0 next-hop 192.168.137.1
   ```

4. **Enable SSH:**
   ```bash
   set service ssh port 22
   set service ssh listen-address 0.0.0.0
   ```

5. **Opslaan en toepassen:**
   ```bash
   commit
   save
   exit
   ```

#### Stap 5: Test VyOS Connectiviteit
1. **Controleer interface:**
   ```bash
   show interfaces
   ```

2. **Test ping naar Linux server:**
   ```bash
   ping 192.168.137.100
   ```

### FASE 3: SSH KEYS SETUP

#### Stap 6: SSH Keys Genereren (op Linux server)
1. **Genereer SSH key:**
   ```bash
   ssh-keygen -t rsa -b 2048
   # Druk gewoon Enter voor alle vragen
   ```

2. **Kopieer key naar VyOS:**
   ```bash
   ssh-copy-id vyos@192.168.137.101
   ```

3. **Test SSH verbinding:**
   ```bash
   ssh vyos@192.168.137.101
   exit
   ```

### FASE 4: ANSIBLE COLLECTIONS INSTALLATIE

#### Stap 7: Installeer Ansible Collections
1. **Ga naar project directory:**
   ```bash
   cd /pad/naar/je/yaml-configuraties
   ```

2. **Installeer collections één voor één:**
   ```bash
   ansible-galaxy collection install vyos.vyos
   ansible-galaxy collection install ansible.posix
   ansible-galaxy collection install community.general
   ```

3. **Controleer installatie:**
   ```bash
   ansible-galaxy collection list
   ```

### FASE 5: INVENTORY CONFIGURATIE

#### Stap 8: Pas Inventory Aan
1. **Bewerk inventory.yml:**
   ```bash
   nano inventory.yml
   ```

2. **Vervang de IP-adressen met jouw werkelijke IPs:**
   ```yaml
   ---
   all:
     children:
       linux_servers:
         hosts:
           linux-server-01:
             ansible_host: 192.168.137.100  # JE LINUX SERVER IP
             ansible_user: jouw-username     # JE LINUX USERNAME
             ansible_ssh_private_key_file: ~/.ssh/id_rsa
       vyos_routers:
         hosts:
           vyos-router-01:
             ansible_host: 192.168.137.101  # JE VYOS ROUTER IP
             ansible_user: vyos
             ansible_ssh_private_key_file: ~/.ssh/id_rsa
             ansible_network_os: vyos.vyos.vyos
             ansible_connection: network_cli
   ```

### FASE 6: CONNECTIVITEIT TESTEN

#### Stap 9: Test Ansible Connectiviteit
1. **Test Linux server:**
   ```bash
   ansible linux_servers -i inventory.yml -m ping
   ```

2. **Test VyOS router:**
   ```bash
   ansible vyos_routers -i inventory.yml -m ping
   ```

**Als je errors krijgt:**
- Controleer IP-adressen in inventory.yml
- Controleer SSH keys
- Controleer firewall instellingen

### FASE 7: PLAYBOOKS UITVOEREN

#### Stap 10: Dry Run Test
1. **Test wat er zou gebeuren:**
   ```bash
   ansible-playbook -i inventory.yml main-playbook.yml --check --diff
   ```

#### Stap 11: Echte Uitvoering
1. **Voer configuratie uit:**
   ```bash
   ansible-playbook -i inventory.yml main-playbook.yml
   ```

### FASE 8: OSTICKET SETUP (OPTIONEEL)

#### Stap 12: Docker Installatie
1. **Installeer Docker:**
   ```bash
   # Ubuntu/Debian:
   sudo apt install docker.io docker-compose -y
   sudo systemctl start docker
   sudo systemctl enable docker
   sudo usermod -aG docker $USER
   ```

2. **Herstart terminal of logout/login**

#### Stap 13: Start osTicket
1. **Start osTicket:**
   ```bash
   docker-compose up -d
   ```

2. **Controleer status:**
   ```bash
   docker-compose ps
   ```

3. **Toegang via browser:**
   - Open browser naar: `http://je-linux-server-ip:8080`

## TROUBLESHOOTING GIDS

### Probleem: "No route to host"
**Oplossing:**
1. Controleer of beide VMs op hetzelfde netwerk staan
2. Controleer firewall instellingen
3. Test ping tussen VMs

### Probleem: SSH verbinding mislukt
**Oplossing:**
1. Controleer SSH service: `sudo systemctl status ssh`
2. Controleer SSH keys: `ssh -v vyos@ip-adres`
3. Controleer gebruikersnamen in inventory.yml

### Probleem: Ansible collections niet gevonden
**Oplossing:**
1. Installeer individueel: `ansible-galaxy collection install vyos.vyos --force`
2. Controleer Python versie: `python3 --version`
3. Update pip: `pip3 install --upgrade pip`

### Probleem: VyOS configuratie mislukt
**Oplossing:**
1. Test handmatige VyOS commando's
2. Controleer vyos.vyos collection versie
3. Controleer VyOS SSH toegang

### Probleem: Docker/osTicket start niet
**Oplossing:**
1. Controleer Docker service: `sudo systemctl status docker`
2. Controleer poort 8080: `sudo netstat -tlnp | grep 8080`
3. Check logs: `docker-compose logs`

## KLANTEN INFORMATIE

### Huidige Klanten:
1. **Stichting BCA**
   - Linux user: `StichtingBCA`
   - Wachtwoord: `ACBgnitthcitS`
   - Router IP: `10.100.201.11`

2. **Regenboog BV**
   - Linux user: `RegenboogBV`
   - Wachtwoord: `VBgoobnegeR`
   - Router IP: `10.100.201.12`

3. **Vereniging Golf**
   - Linux user: `VerenigingGolf`
   - Wachtwoord: `floGginginerev`
   - Router IP: `10.100.201.13`

## NIEUWE KLANTEN TOEVOEGEN

### Stap 1: Bewerk YAML bestanden
1. **Open linux-server-config.yml**
2. **Voeg nieuwe klant toe aan customers lijst:**
   ```yaml
   - name: "NieuweKlant"
     password: "tnalKweueiN"  # naam achterstevoren
   ```

3. **Open router-config.yml**
4. **Voeg nieuwe klant toe:**
   ```yaml
   - name: "Nieuwe Klant"
     ip: "10.100.201.14"      # volgend IP
     interface: "lo14"        # volgend interface
   ```

### Stap 2: Git Workflow
```bash
git add .
git commit -m "Add new customer: NieuweKlant"
git push origin main
```

### Stap 3: Uitrollen
```bash
ansible-playbook -i inventory.yml main-playbook.yml
```

## BESTANDEN OVERZICHT

- `linux-server-config.yml` - Linux gebruikers configuratie
- `router-config.yml` - VyOS router configuratie  
- `main-playbook.yml` - Hoofdplaybook
- `inventory.yml` - Server definities (PAS DEZE AAN!)
- `requirements.yml` - Ansible collections
- `docker-compose.yml` - osTicket systeem
- `ansible.cfg` - Ansible instellingen

## BELANGRIJKE NOTITIES

- **Vervang altijd de IP-adressen in inventory.yml met je werkelijke IPs**
- **Test altijd eerst met --check voordat je echt uitvoert**
- **Backup je configuraties voordat je wijzigingen maakt**
- **Alle wachtwoorden worden automatisch gehashed voor security**
- **Linux gebruikers krijgen geen shell toegang (/bin/false)**

## HULP NODIG?

Als je vastloopt, controleer dan:
1. Netwerk connectiviteit tussen VMs
2. SSH toegang tot beide systemen  
3. Ansible collections zijn geïnstalleerd
4. IP-adressen in inventory.yml zijn correct
5. Firewall instellingen op beide systemen