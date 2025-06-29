# Cloud Services Infrastructure Automation

Dit repository bevat de Ansible configuraties voor het automatisch uitrollen van cloud klanten op Linux servers en VyOS routers.

## Overzicht

De organisatie gebruikt deze configuraties om snel nieuwe klanten toe te voegen aan de infrastructuur:

### Linux Server Configuratie
- Gebruikersaccounts per klant
- Groep: `cloud_users`
- Wachtwoord: klantnaam achterstevoren
- Geen shell-toegang voor security

### Router Configuratie  
- Loopback interfaces per klant
- IP-range: 10.100.201.11 en opvolgend
- Beschrijving: 'customer <naam>'

### Huidige Klanten
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

## Bestanden

- `linux-server-config.yml` - Ansible playbook voor Linux server configuratie
- `router-config.yml` - Ansible playbook voor VyOS router configuratie
- `main-playbook.yml` - Hoofdplaybook dat beide configuraties uitvoert
- `inventory.yml` - Ansible inventory met host definities
- `requirements.yml` - Benodigde Ansible collections
- `docker-compose.yml` - osTicket ticketing systeem

## Gebruik

### 1. Installeer benodigde collections
```bash
ansible-galaxy collection install -r requirements.yml
```

### 2. Pas inventory aan
Bewerk `inventory.yml` en pas de IP-adressen aan naar uw werkelijke servers:
- Linux server IP
- VyOS router IP
- SSH gebruikers en keys

### 3. Voer configuratie uit
```bash
# Beide systemen configureren
ansible-playbook -i inventory.yml main-playbook.yml

# Of individueel:
ansible-playbook -i inventory.yml linux-server-config.yml
ansible-playbook -i inventory.yml router-config.yml
```

### 4. Start osTicket (optioneel)
```bash
docker-compose up -d
```
osTicket is dan beschikbaar op: http://localhost:8080

## Nieuwe Klanten Toevoegen

1. Bewerk de `customers` variabele in beide YAML bestanden
2. Voeg nieuwe klant toe met:
   - `name`: Klantnaam (zonder spaties)
   - `password`: Klantnaam achterstevoren
   - `ip`: Volgend beschikbaar IP (10.100.201.14, etc.)
   - `interface`: Volgend loopback interface (lo14, etc.)

3. Commit wijzigingen naar Git repository
4. Voer playbooks uit vanaf Ansible server

## Git Workflow

```bash
# Wijzigingen maken
git add .
git commit -m "Add new customer: [KlantNaam]"
git push origin main

# Op Ansible server
git pull origin main
ansible-playbook -i inventory.yml main-playbook.yml
```

## Security Overwegingen

- Linux gebruikers hebben geen shell toegang (/bin/false)
- Wachtwoorden worden gehashed opgeslagen
- SSH keys worden gebruikt voor Ansible toegang
- Alle configuraties zijn in version control

## Troubleshooting

### Linux Server Issues
- Controleer of `cloud_users` groep bestaat
- Verificeer SSH toegang voor Ansible
- Check `/etc/passwd` voor correcte shell setting

### Router Issues  
- Controleer VyOS SSH toegang
- Verificeer dat vyos.vyos collection geïnstalleerd is
- Test handmatige VyOS commando's

### osTicket Issues
- Controleer Docker service status
- Verificeer database connectie
- Check logs: `docker-compose logs osticket`