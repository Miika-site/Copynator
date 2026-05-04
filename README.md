# Copynator
Tämä projekti sisältää Ansible‑automaation, joilla varmuuskopioidaan valittuja tietoja lähdepalvelimelta kohdepalvelimelle. 

Projektin tavoitteet: 
- luoda yksinkertainen, toistettava ja luotettava tapa ottaa säännöllisiä varmuuskopioita
- varmuuskopion automaattinen ajastus (cron)
- salasanaton ajo (SSH-avaimet + sudo NOPASSWD)

## Hakemistorakenne
````
Copynator/
├── ansible.cfg
├── hosts.ini
├── LICENSE
├── README.md
├── roles
│   └── backup
│       └── tasks
│           └── main.yml
└── serverbackup.yml
````

## Käyttöönotto
### Projektin kloonaus
````
git clone git@github.com:Miika-site/Copynator
````

### SSH‑avaimen käyttöönotto

Jotta Ansible voi toimia ilman salasanoja, lisää lähdepalvelimen julkinen avain ja kopioi se kohdepalvelimelle:
````
ssh-keygen
ssh-copy-id ~/.ssh/id_ed25519.pub käyttäjä@localhost
````


ja mahdollistaa automatisoidun SSH‑yhteyden.

Testaa:  
`ssh localhost`  - toimii ilman salasanaa.

## Sudo ilman salasanaa

Lähdepalvelimella: `sudo visudo`
lisää `käyttäjä ALL=(ALL) NOPASSWD: ALL`

Testaa `sudo whoami`. Jos tulos on `root` ilman salasanaa -> toimii.

## Projektin rakenne 

### Ansiblen-roolin perusrakenteen luonti
````
mkdir -p roles/backup/tasks
touch ansible.cfg
touch hosts.ini
touch serverbackup.yml
touch roles/backup/tasks/main.yml
````

hosts.ini
````
[all:vars]
ansible_python_interpreter=/usr/bin/python3

[source]
source ansible_host=10.0.2.15 ansible_user=rage2 # lähdepalvelin - valitse oikea IP

[target]
target ansible_host=127.0.0.1 # kohdepalvelin - vaihdetaan oikean palvelimen IP
````

ansible.cfg
````
[defaults]
inventory = hosts.ini
roles_path = roles
 
[privilege_escalation] # varmuuskopioitavat tiedostot voivat vaatia sudo-oikeuksia, varmistetaan tällä kopioinnin onnistuminen
become = True
become_method = sudo
become_ask_pass = False
````
roles/backup/tasks/main.yml

Projektissa käytämme polkua /etc/hosts, jota varmuuskopioimme. 
````
- name: Varmuuskopioitavat polut
  set_fact:
    source_path: "/etc/hosts" # lähdepalvelimen varmuuskopioitava kohde
    target_path: "/home/hosts-backup" # kohdepalvelimen polku, johon varmuuskopio luodaan

- name: Luo hakemisto kohdepalvelimelle
  file:
    path: "{{ target_path }}"
    state: directory
    mode: '0755' # oikeudet kohdepalvelimelle hakemiston luomista varten. Huomioi, että lisää oikeuksissa vain tarvittavat 


- name: Kopioi valitut tiedot kohdepalvelimelle
  copy:
    src: "{{ source_path }}"
    dest: "{{ target_path }}/"
````

## Varmuuskopioinnin ajoituksen automatisointi (cron)

Avaa lähdepalvelimella cron:
`crontab -e`

Lisää tiedostoon:
````
0 */5 * * * cd /home/käyttäjä/git/copynator/Copynator && ansible-playbook serverbackup.yml >> /home/käyttäjä/backup.log 2>&1>
```` 
Ylläoleva ottaa varmuuskopion viiden tunnin välein. Login voi tarkistaa 
````
tail -f /home/käyttäjä/backup.log
````

## Ansible playbookin suorittaminen

Aja playbook:
````
ansible-playbook serverbackup.yml
````

## Kehittäjät 
- Kalle Kyröhonka
- Miika Vänttinen
