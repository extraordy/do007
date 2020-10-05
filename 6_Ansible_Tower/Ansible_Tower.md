# Ansible Tower

## Ansible Tower: Cosa manca ad Ansible?

Come abbiamo visto, Ansible è un potentissimo strumento per configurare e gestire ogni parte della nostra infrastruttura però, in quanto strumento da commandline, manca effettivamente di alcune funzionalità più raffinate, fondamentali in ambienti Enterprise, come ad esempio:

- Tracciare chi ha lanciato quale playbook
- Scegliere chi può fare cosa in base al proprio ruolo
- Avere una reportistica delle azioni compiute

Per sopperire a questa problematica esiste per l'appunto **Ansible Tower**


## Ansible Tower: Cosa è?

Anisble Tower è un orchestratore che non fa altro che utilizzare Ansible Engine per eseguire tutti i nostri playbook, ma contemporanemente va ad arricchirlo con delle funzionalità molto interessanti:

- **Schedulazione** di playbook
- Gestione centralizzata dell'inventory
- Controllo **RBAC** (*Role-Based Access Controls*) su gli accessi (chi può fare cosa dove)
- **Esecuzione contemporanea** di più playbook (da cli avremo necessita di aprire più terminali)
- Supporto ad **API REST** (quindi integrazioni con hooks di strumenti di CI/CD)
- Dashboard **grafiche** sullo stato dei playbook eseguiti
- Esecuzione di **Workflow** in base anche agli eventi (concatenare più playbook od eseguirne altri in base allo stato dell'ultima esecuzione)
- Sistema di **notifica integrata** dello stato del playbook (si integra con Slack, SMS, E-Mail, etc..)


## Ansible Tower: Esiste anche la versione Community?

Come ogni progetto di Red Hat, esiste anche per Ansible Tower il relativo progetto community. 
In questo caso il progetto è chiamato AWX e lo possiamo trovare su github al link [github.com/ansible/awx](https://github.com/ansible/awx).

Differisce da Ansible Tower sia per il livello di supporto, unicamente quello della comunity, che per i metodi di installazione, ossia AWX ha come metodo di installazione ufficiale unicamente quello mediante container (che siano gestiti manualmente o in un contesto più elaborato con Openshift).


## Ansible Tower: Come installarlo


Per installare Ansible Tower è necessario richiedere una valutazione gratuita di 60 giorni della piattaforma a questo [link](https://www.ansible.com/products/tower), seguendo le procedure ci verrà fatto scaricare un archivio.

L'archivio conterrà lo script di installazione, che (ovviamente) andrà a chiamare dei playbook ansible che configureranno il sistema secondo le variabili impostate.

Per impostare queste variabili dobbiamo andare a modificare i files in:
- `{$WORKING_DIR}/inventory`

Le variabili minime per eseguire l'installazione da settare sono le seguenti:
- `admin_password=''`
- `pg_password=''`

Nello stesso file avremmo anche la possibilità di configurare l'host sul quale installare Tower in Bundle (Tower + PostgresSQL), nel caso in cui l'installazione avvenga su server remoto è necessario che l'utente utilizzato per installare Tower abbia i permessi di root; potremmo anche impostare i server separatamente nella sezione `[tower]` e `[database]`.

Di seguito un esempio di configurazione minimale del file:

```ini
[tower]
localhost ansible_connection=local

[database]

[all:vars]
admin_password='password'

pg_host=''
pg_port=''

pg_database='awx'
pg_username='awx'
pg_password='awx'
```
Una volta aggiunte queste informazioni minime necessarie, possiamo lanciare lo script `{$WORKING_DIR}/setup.sh` per procedere con l'installazione, durante la quale avremmo l'output di Ansible dell'esecuzione dei vari playbook.

Lo stesso script può essere lanciato con alcuni argomenti, elencati di seguito:\
  `-e EXTRA_VARS`\
  `-i INVENTORY`\
  `-b DATABSE BACKUP`\
  `-r DATABASE RESTORE`\
  `-h HELP`