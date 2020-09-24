# Ansible Tower

## Cosa manca ad Ansible?

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
