# Windows automation

Forse non tutti sanno che Ansible è in grado gestire il deployment e il provisionig dei sistemi operativi Microsoft Windows. Sempore più sistemisti, infatti, predilgono l Anible per l'automation della maggior parte dei processi IT, proprio per svincolarsi da quelle che sono le soluzioni verticali e proprietarie degli specifici ambienti e poter così attingere risorse e conoscenza dall'enorme community open source.

Quando il nostro managed host non è una macchina UNIX, dobbiamo però avere delle accortezze particolari. In questa aritcolo proveremo a riassumere gli step fondamentali.

## Cosa serve?

### La connessione (non è SSH)

Conclusa un'installaizone di default di Windows non ci troviamo certo un server SSH installato. Per questa ragione si è scelto di utilizzare **WinRM**, un protocollo di gestione basato su HTTP. Non è un protocollo per la login remota interattiva come SSH, anche se comunque il supporto ad OpenSSH è in test.

Lato Ansible, bisognerà impostare il plugin di connessione **winrm** anziché quello di default (**smart**).

### PowerShell e .NET Framework

I moduli Ansible che andremo ad utilizzare non sono gli stessi che abbiamo visto finora per UNIX. Questo non perché un modulo non possa essere scritto per supprtare sistemi operativi diversi, ma proprio perché su Windows cambia il modo con cui sono implemetati i moduli. Ad esempio, non essendoci installato Python di default, è stato scelto di scrivere i moduli in **PowerShell**.

PowerShell è il linguaggio giusto perché è perfettamente integrato in Windows e il suo interprete *conosce* tutti gli stati e le estensioni del sistema operativo, facilitando il lavoro di chi sviluppa codice. Per alcuni moduli particolari inoltre, è necessario anche avere .NET Framework come dipendenza.

## Cosa cambia?

### L'inventory

Se il nostro progetto insiste interamente su macchine Windows, andremo a specificare variabili come **ansible_connection** a livello globale. Altrimenti, possiamo definire le opzioni necessrie direttamente nell'inventory come nell'esempio segeunte:

```
[winhosts]
win01
192.168.50.27

[winhosts:vars]
ansible_user=user
ansible_password=1q2w3e4r
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
```

Nell'esempio precedente, **ansible_connection** punta proprio 

## Preparare un host Windows per essere automatizzato da Ansible

### Requisiti minimi

Tutte le versioni attualemnte supportate da Winsows (sia supporto regalorae che extended) possono esserre autmatizzate da Ansible. Alcune versioni però, come Windows Server 2008 o Windows 7, hanno bisogno dell'aggiornamento della PowerShell e del .NET Framework. Da Windows Server 2012 e da Windows 8 invece, tutte le dipendenze a livello di software sono già spoddisfatte dopo l'installaizone iniziale.

I requisiti minimi sono seguenti:

- Windows Server (2008, 2008 R2, 2012, 2012 R2, 2016, o 2019) oppure Windows (7, 8.1, o 10)
- PowerShell 3.0 o superiore
- .NET Framework 4.0 o superiore

### Configurazioni necessarie

Nonstante componenti come WinRM siano già presenti di default a partire da Windpows Server 2012, è necessario configurare alcune risorse aggiuntive:

- un WinRM listener
- le regole del firewall per accettare traffico HTTP in ingresso (neessarie per WinRM)
- il backend di autenticazione da utilizzare

In un ambiente di produzione, sarà necessario gestire tutte queste confiurazioni magari tramite Sysprep o comunque con la soluzione di installazione automatizzata del proprio parco macchine Windows. Per un ambiente di test invece, è disponibile uno scrip in grado di apportare tutte le configurazioni necessarie ad Ansible per potersi collegare all'host. Lo script si chiama [ConfigureRemotingForAnsible.ps1](https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1) ed è liberamente scaricabile da GitHub.

Qui di seguito è disponbile uno screencast che riproduce l'esecuzione dello script di installazione:
[![Basic Windows host configuration for Ansible](http://img.youtube.com/vi/-8IY8eHZJw8/0.jpg)](http://www.youtube.com/watch?v=-8IY8eHZJw8 "Basic Windows host configuration for Ansible")

