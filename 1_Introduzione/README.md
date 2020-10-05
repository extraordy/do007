# Introduzione ad Ansible

<p align="center">
    <img src="Ansible_logo.svg" alt="Ansible"/>
</p>

## Scopo e funzionamento di Ansible

Ansible è uno strumento per l'automazione, che permette a un amministratore di sistema di effettuare delle configurazioni su più host per raggiungere lo stato desiderato.

Il principio più evidente è la **semplicità**, a partire dall'installazione fino al suo utilizzo, sia per i pochi prerequisiti tecnici, sia per il sostegno della community sui forum e blog.
L'unico requisito necessario per usare Ansible è che i sistemi accettino una connessione SSH e che abbiano Python installato, poiché non viene installato un agente, un piccolo programma che si occupa di ricevere istruzioni, ma distribuisce le modifiche da effettuare sulle macchine gestite secondo un modello *push* (il sistema su cui viene eseguito Ansible funziona da "burattinaio") dando direttamente i comandi da eseguire, il che lo rende minimale e **sicuro**.

L'altro pilastro della filosofia di Ansible è la garanzia che, una volta eseguite con successo tutte le modifiche richieste alla configurazione delle macchine gestite, esse avranno raggiunto tutte il medesimo stato: questa proprietà prende il nome di **idempotenza**, e può essere intesa come il fatto che Ansible richieda le modifiche sulle macchine gestite secondo un file che *descrive* la configurazione finale da raggiungere su di esse.
Per questo i playbook di Ansible, i copioni che descrivono lo stato desiderato, sono anche **ripetibili**, in quanto apportano modifiche solo se necessario, permettendoci anche di omogenizzare la configurazione un gruppo di host altrimenti eterogeneo.
Ne derivava che per la sua semplicità e la ripetibilità dei playbook, potremo quindi gestire più gruppi di host in base alla loro funzione o che devono rispettare determinate caratteristiche, sui quali poi eseguire un playbook.
Un ad esempio può essere l'installazione di un server web limitato solo ad un gruppo di server.

Ultimo ma non per importanza, Ansible è **estendibile**, dal momento che utilizza SSH e nessun'altra dipendenza esterna; naturalmente, è possibile estendere questo strumento aggiungendo dei *moduli* specifici per le configurazioni o le piattaforme gestite, e anche scrivere i propri moduli personalizzati.
Qui Ansible trova tutta la sua forza, grazie al supporto di Red Hat e alla vastissima community, che estende questo strumento con moduli e ruoli disponibili sul portale di Ansible Galaxy.

Per questi motivi Ansible è lo strumento più semplice ma potente a disposizione degli amministratori per promuovere sistemi in produzione, in poco tempo.
## Piattaforme supportate

Dalla [documentazione ufficiale](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#prerequisites) del progetto Ansible, è possibile installare ed eseguire lo strumento `ansible` da qualunque macchina che abbia i seguenti requisiti.

### Nodo di controllo

La macchina che invierà i comandi e gestirà gli altri nodi; essa deve avere:

- Python 2.7 o 3.5, o superiori;
- il SO sia uno tra:
    - una distribuzione Linux basata su RHEL, Debian o BSD;
    - MacOS.

Microsoft Windows al momento non può eseguire `ansible` come un nodo di controllo.

### Nodo gestito

La o le macchine che saranno gestite dal nodo di controllo tramite Ansible; esse devono avere:

- Python 2.6 o 3.5, o superiori (l'interprete sia installato in `/usr/bin/python`);
- un metodo di comunicazione tra ciascun nodo gestito e il nodo di controllo tramite SSH (*SCP* o *SFTP*, configurabile nel file `ansible.cfg`);

Per i nodi gestiti con SO basato su Linux, questo è quanto; tuttavia, se il nodo ha un SO differente, seguono dei requisiti aggiuntivi:

- *su Microsoft Windows*: in aggiunta, è necessario che sia installato PowerShell 3.0 o superiore, il .NET Framework 4.0 o superiore, e sia in ascolto un processo WinRM (per i dettagli rimandiamo alla [sezione dedicata](https://docs.ansible.com/ansible/latest/user_guide/windows_setup.html#host-requirements) della guida ufficiale);
- *su apparati di rete*: Ansible è usato anche per la configurazione di apparati di rete (router, switch, firewall, ecc...), in tal caso la configurazione e i requisiti variano in base alla compatibilità dei dispositivi; questa è la [sezione dedicata](https://docs.ansible.com/ansible/latest/network/getting_started/first_playbook.html#prerequisites) sulla guida ufficiale.


## Metodo di installazione

### RHEL, CentOS, Fedora

In generale, è possibile installare il pacchetto col seguente commando:

```bash
sudo yum install ansible
```

Su Fedora, è possibile invocare direttamente il gestore pacchetti:

```bash
sudo dnf install ansible
```

Su RHEL è necessario attivare i repository che contengono i pacchetti di Ansible, tramite il gestore di iscrizione (sarà necessaria una licenza Red Hat per il sistema operativo):

- **RHEL 8**:

    ```bash
    sudo subscription-manager repos --enable ansible-2.9-for-rhel-8-x86_64-rpms
    ```

- **RHEL 7**:

    ```bash
    sudo subscription-manager repos --enable rhel-7-server-ansible-2.9-rpms
    ```

### Ubuntu 18 o superiore

Eseguite in successione i seguenti comandi:

```bash
sudo apt update
sudo apt install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

### Debian 8 Jessie o superiore

Eseguite in successione i seguenti comandi:

```bash
echo "deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main" | sudo tee /etc/apt/sources.list
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
sudo apt update
sudo apt install ansible
```

### MacOS 10.12 o superiore

Per installare `ansible` su MacOS è necessario usare il gestore di pacchetti Python `pip`; installatelo con:

```bash
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py --user
```

Infine installate `ansible` col comando:

```bash
pip install --user ansible
```

## Supporto e community

Ansible è un progetto open source: vuol dire che la sua community è vasta e varia, oltre al fatto che i suoi membri contribuiscono ad estendere le funzionalità tramite moduli:

- sul sito del progetto Ansible, a [questa pagina](https://www.ansible.com/community), c'è una raccolta di link utili ai vari canali di comunicazione e agli eventi organizzati dalla community;
- la guida ufficiale ha una [sezione dedicata](https://docs.ansible.com/ansible/latest/community/index.html) agli aspiranti membri della community.
