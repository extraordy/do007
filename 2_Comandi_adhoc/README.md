# Comandi ad-hoc

L'invocazione ad-hoc di Ansible permette di eseguire i singoli moduli, di solito
viene usata per comandi semplici come verificare la connettività alle singole
macchine, tramite il modulo `ping`, o per raccogliere informazioni sugli host, tramite il modulo `setup`.

Prima di poter usare Ansible in qualsiasi modalità, sara prima necessario configurare il file `ansible.cfg` e creare un `inventory`.

## `ansible.cfg`
Ansible opera tramite una moltitudine variabili che possono essere configurate in vari modi, i file che le contengono sono nel formato `ini`.
Quando Ansible viene eseguito controlla la presenze dei seguenti file per caricare le impostazioni, con ordine d'importanza crescente:

* `/etc/ansible/ansible.cfg`
* `<cartella home>/.ansible.cfg`
* `ansible.cfg` dalla cartella corrente
* Variabili d'ambiente

Normalmente quando Ansible viene installato verrà creato `/etc/ansible/ansible.cfg` e popolato con tutte le opzioni disponibili commentate.

Le opzioni che principalmente vengono modificate sono:

```ini
[defaults]
remote_user=ansible

[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False
```

* `remote_user`: L' utente che viene usato per connettersi alla macchina remota, di default viene usato l'username dell'utente corrente
* `become`: Abilita o meno il cambio dell'utente nel caso di comandi che necessitano di diritti amministrativi (es. yum)
* `become_method`: Come viene eseguito il cambio di utente, oltre a `sudo` è anche disponibile `su`, altri metodi sono elencati [qui](https://docs.ansible.com/ansible/latest/plugins/become.html#plugin-list)
* `become_user`: L'utente che si vuole impersonare per avere diritti amministrativi
* `become_ask_pass`: Serve per abilitare l'inoltro di eventuali richieste password (es. in caso di sudo)

Questa è solo una selezione ristretta delle opzioni più comuni, ci sono molte più variabili che possono essere usate per modificare altri aspetti di Ansible.  
La lista completa di variabili è reperibile [qui](https://docs.ansible.com/ansible/latest/reference_appendices/config.html)

## Inventory
Per poter passare a Ansible la lista degli host su cui operare è necessario creare un Inventory, gli inventory non sono altro che liste ordinate di host, IP o hostname, in formato `ini`.

### Preparare un inventory

Con inventory si definiscono le liste organizzate di macchine su cui Ansible
deve operare, possono essere utilizzati sia in modalità ad-hoc che tramite i playbook e permettono la gestione organizzata di molti host simultaneamente.  
Gli inventory  si dividono in due categorie:
* #### liste statiche 
  file in formato `ini` con gli host definiti staticamente  
* #### liste dinamiche
  sono eseguibili che ritornano dinamicamente la lista delle macchine nel formato `ini`  

Noi vedremo le liste statiche, sono una lista di ip o di hotname delle macchine
interessate, possono essere organizzate in gruppi per facilitarne la gestione,
è anche possibile definire gruppi di gruppi come nell'esempio sottostante

```ini
[webserversA]
webserverA1
10.101.11.23

[webserversB]
webserverB1
10.101.12.25

[proxies]
10.1.0.[1-2]

[webservers:children]
webserversA
webserversB
```

## Cosa sono, come si usano

I comandi ad-hoc permettono l'invocazione di un singolo modulo di Ansible,
vengono usati per operazioni veloci e poco complesse dove la creazione di un
playbook, un'insieme di task che eseguono sequenzialmente dei moduli, non è necessaria.  
Una lista più esaustiva si può trovare al [modules index di Ansible](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html)

un'esempio di invocazione dei comandi ad-hoc è:

```
ansible webserverB -i inventory -u testuser -m ping 
```

* `ansible` è il nome del programma da riga di comando Ansible per invocare la modalità ad-hoc
* `webserverB` identifica quale gruppo di host voglio invocare, è anche possibile inserire un gruppo di gruppi
* `-i` indica il nome del file contenente l'inventory interessato, nell'esempio `inventory`
* `-u` definisce con che utente intendiamo collegarci, nel caso specifico `testuser`
* `-m` dichiara il modulo che intendiamo usare, specificatamente `ping`

sotto alcuni esempi dei moduli più comuni

### Ping
Il comando ping viene utilizzato per controllare l’effettiva presenza delle
macchine in inventory e verificarne la raggiungibilità da parte del _control node_.

```bash
$ ansible -i inventory webserverA1 -m ping
webserverA1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

[![asciicast](https://asciinema.org/a/wfx5cQqNmdvLGlX6kfg5P1P0E.svg)](https://asciinema.org/a/wfx5cQqNmdvLGlX6kfg5P1P0E)

### Collezione dei _facts_
Ansible, all'avvio di un playbook, esegue automaticamente un task per ottenere maggiori informazioni sugli host che verranno manipolati.  
Queste informazioni sono normalmente utilizzate per aggiungere flessibilità ai playbook,
come per esempio potere riutilizzare lo stesso playbook su Centos, RHEL e Fedora.  
La rappresentazione grezza di questi dati può essere stampata a schermo tramite un comando invocando il modulo `setup`.

```bash
$ ansible proxies -i inventory -m setup
10.1.0.2 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.1.0.2"
        ],
        "ansible_apparmor": {
            "status": "disabled"
        },
[...]
10.1.0.1 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.1.0.1"
        ],
        "ansible_apparmor": {
            "status": "disabled"
        },
[...]
```

### Package manager

Come modulo esistono anche i package manager delle varie distribuzioni,
l'invocazione è analoga agli esempi precedenti, se volessimo installare
`vim` e `zsh` su tutte le macchine `proxies` utilizzeremmo:

```bash
$ ansible proxies -i inventory -m yum -a 'name=zsh,vim state=installed'
```

ovviamente il tempo di risposta dei moduli varia in base al tempo di esecuzione dei programmi invocati dal modulo stesso.
