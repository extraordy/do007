# Ruoli Ansible

## Cosa sono gli Ansible Roles e a cosa servono?

Gli Ansible Roles sono un metodo di gestione del codice di automazione: Si basano sull'inclusione selettiva di contenuti e offrono la possibilità di riutilizzare il codice che scriviamo, semplificarne lo sviluppo e aumentarne la coerenza.

Per richiamare un ruolo si utilizza il tag specifico dall'interno di un playbook.
```yaml
---
- hosts: servers
  roles:
    - role1
    - role2
```

Oltre a colmare la necessità di organizzare i playbook in una struttura logica, che nasce al crescere della complessità dei progetti di automazione, i ruoli permettono di includere task, variabili, files e templates, che verranno eseguiti o utilizzati come parametri dai moduli, localizzando questi contenuti in una directory standardizzata.

Oltre che più ordinato, come naturale conseguenza lo sviluppo del codice di automation diviene parallellizzabile e conseguentemente più semplice da sviluppare.

## Utilizzo di Galaxy e di un Ansible Role

In questo esempio mostreremo come cercare, scaricare ed utilizzare un Ansible Role presente sulla piattaforma  Galaxy: il portale della community di Ansible.


```bash
[user@workstation ~]$ ansible-galaxy search 'apache' --author geerlingguy

Found 15 roles matching your search:

 Name                       Description
 ----                       -----------
 geerlingguy.adminer        Installs Adminer for Database management.
 'geerlingguy.apache         Apache 2.x for Linux'.
 geerlingguy.apache-php-fpm Apache 2.4+ PHP-FPM support for Linux.
 geerlingguy.certbot        Installs and configures Certbot (for Lets Encrypt).
 geerlingguy.drupal         Deploy or install Drupal on your servers.
 ...
```
Dopodiché procediamo ad installare il ruolo nella nostra directory ansible di default.

```bash
[user@workstation ~]$ ansible-galaxy install geerlingguy.apache
- downloading role 'apache', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-apache/archive/3.1.0.tar.gz
- extracting geerlingguy.apache to /home/user/.ansible/roles/geerlingguy.apache
- geerlingguy.apache (3.1.0) was installed successfully
```

```bash
[user@workstation ]$ ls roles/geerlingguy.apache/
defaults  LICENSE  meta  README.md  tasks  tests  vars
```

Scriviamo a questo punto un playbook che includa il ruolo appena scaricato e lo utilizzi con alcune variabili

```yaml
---
- name: Deploy apache server
  hosts: lab_server
  become: true
  become_method: sudo
  vars:
    apache_listen_ip: "*"
    apache_listen_port: 80

  roles:
    - geerlingguy.apache

  post_tasks:
    - name: Edit index file
      copy:
        dest: /var/www/html/index.html
        content: "Managed by Ansible! HOST: {{ ansible_facts['fqdn']}}"
```

Andiamo a verificare che il server sia stato configurato correttamente.

```bash
[user@workstation ~]$ curl serverc.lab.example.com
Managed by Ansible! HOST: serverc.lab.example.com
```

[![asciicast](https://asciinema.org/a/364034.svg)](https://asciinema.org/a/364034)




## Come si sviluppa un Ansible Role?

I ruoli di Ansible possono essere sviluppati con un semplice editor di testo, è necessario innanzitutto creare la struttura della directory del ruolo, ci viene in aiuto il comando `ansible-galaxy init`


```bash
[user@workstation ~]$ ansible-galaxy init extraordy.myweb
- Role extraordy.myweb was created successfully

[user@workstation ~]$ tree
.
└── extraordy.myweb
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── README.md
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml
```

Per poi definire il contenuto dei files presenti:

Per sviluppare questo role ci riferiremo ad un playbook esistente **che configura un host per esporre una pagina html generata.** scomponendolo nelle sue singole unità.

```yaml
- name: dynamic http frontpage
  hosts: <HOSTS>
  become: true
  become_method: sudo
  vars:
    listen_port: "Listen 80"
    server_root: "ServerRoot '/etc/httpd'"
    user: "User apache"
    group: "Group apache"

  tasks:
    - name: httpd must be present and updated
      yum:
        name: httpd
        state: latest

    - name: httpd service is enabled and started
      service:
        name: httpd
        state: started
        enabled: yes

    - name: Firewall allows http trafic
      firewalld:
        service: http
        permanent: yes
        state: enabled

    - name: Set apache config
      lineinfile:
        path: /etc/httpd/conf/httpd.conf
        regexp: "{{ item.exp }}"
        line: "{{ item.line }}"
      loop:
        - { exp: "^Listen", line: "{{ listen_port }}" }
        - { exp: "^ServerRoot", line: "{{ server_root }}" }
        - { exp: "^User", line: "{{ user }}" }
        - { exp: "^Group", line: "{{ group }}" }

    - name: index.thtml is copied from provided file
      tags: Index
      copy:
        content: |
              Managed by Ansible! <br />
              -------------------------------------------------------------
              <br />
              HOST = {{ ansible_facts['fqdn'] }} <br />
              IP   = {{ ansible_facts['default_ipv4']['address'] }} <br />
              DATA = {{ ansible_facts['date_time']['date'] }} <br />
              -------------------------------------------------------------
              <br />
              MEMORY = {{ ansible_facts['memtotal_mb'] }} MB <br />
              CPU    = {{ ansible_facts['processor'] }} <br />
              DISTRO = {{ ansible_facts['distribution'] }} <br />
              ARCH   = {{ ansible_facts['architecture'] }}
        dest: /var/www/html/index.html
      notify: Restart apache

  handlers:
    - name: Apache is restarted
      service:
        name: httpd
        state: restarted
```
Procediamo alla scomposizione dell'Ansible Role inserendo nel file `tasks/main.yml` i task in modo tale che possano essere inclusi ed eseguiti nel contesto di un playbook.

```yaml
[user@workstation extraordy.myweb]$ cat tasks/main.yml 

---
# tasks file for extraordy.myweb
- name: httpd must be present and updated
  yum:
    name: httpd
    state: latest

- name: httpd service is enabled and started
  service:
    name: httpd
    state: started
    enabled: yes

- name: Firewall allows http trafic
  firewalld:
    service: http
    permanent: yes
    state: enabled

- name: Set apache config
  lineinfile:
    path: /etc/httpd/conf/httpd.conf
    regexp: "{{ item.exp }}"
    line: "{{ item.line }}"
  loop:
    - { exp: "^Listen", line: "{{ listen_port }}" }
    - { exp: "^ServerRoot", line: "{{ server_root }}" }
    - { exp: "^User", line: "{{ user }}" }
    - { exp: "^Group", line: "{{ group }}" }

- name: index.thtml is copied from provided file
  tags: Index
  copy:
    content: |
          Managed by Ansible! <br />
          -------------------------------------------------------------
          <br />
          HOST = {{ ansible_facts['fqdn'] }} <br />
          IP   = {{ ansible_facts['default_ipv4']['address'] }} <br />
          DATA = {{ ansible_facts['date_time']['date'] }} <br />
          -------------------------------------------------------------
          <br />
          MEMORY = {{ ansible_facts['memtotal_mb'] }} MB <br />
          CPU    = {{ ansible_facts['processor'] }} <br />
          DISTRO = {{ ansible_facts['distribution'] }} <br />
          ARCH   = {{ ansible_facts['architecture'] }}
    dest: /var/www/html/index.html
  notify: Restart apache
```

Definiamo le variabili di defaults nel `defaults/main.yml`, sarà possibile modificare la configurazione eseguendo solo il play che valorizza il placeholder sostituendo le variabili di default.

```yaml
[user@workstation extraordy.myweb]$ cat defaults/main.yml 

---
# defaults file for extraordy.myweb
listen_port: "Listen 80"
server_root: "ServerRoot '/etc/httpd'"
user: "User apache"
group: "Group apache"
```
Infine andremo a definire gli handlers che saranno chiamati dai nostri tasks in `handlers/main.yml`.
```yaml
[user@workstation extraordy.myweb]$ cat handlers/main.yml 

---
# handlers file for extraordy.myweb
- name: Restart apache
  service:
    name: httpd
    state: restarted
```
Eseguita correttamente la scomposizione, il nostro Ansible Role è pronto per essere eseguito, definiamo un playbook che chiama il role per eseguire le azioni di cui abbiamo bisogno. 

Teniamo sempre presente che quando richiamiamo un ruolo tramite un plabyook, Ansible cercherà i contenuti ai quali ci riferiamo nell'alberatura di galaxy della cartella corrente.

```yaml
[user@workstation ~]$ cat myweb.yml

---
- name: Deploy myweb role
  hosts: <HOST>
  become: true
  become_method: sudo

  roles:
    - extraordy.myweb

  post_tasks:
    - name: Check deployment result
      shell: 'curl http://<HOST>'
      register: result

    - debug:
        var: result.stdout
```

Andiamo a valorizzare il placeholder "*\<HOST>*" e siamo pronti per lanciare il ruolo.

[![asciicast](https://asciinema.org/a/364154.svg)](https://asciinema.org/a/364154)