

## Основная информация:
```
localhost - ansible host

test
ssh ansible@localhost -p 2220

web
ssh ansible@localhost -p 2222
ssh ansible@localhost -p 2224

check web
ansible web -m ping

start playbook
ansible-playbook site.yml
```

---

1. Установите Ansible на локальную машину.


```
(.venv) iyunakov@mbp-iyunakov-OZON-FVFG74SJQ05N
~/MeGitHub/ansibleprac 
% which pip python
/Users/iyunakov/MeGitHub/ansibleprac/.venv/bin/pip
/Users/iyunakov/MeGitHub/ansibleprac/.venv/bin/python
(.venv) iyunakov@mbp-iyunakov-OZON-FVFG74SJQ05N
~/MeGitHub/ansibleprac 
% ansible --version
ansible [core 2.17.14]
  config file = /Users/iyunakov/MeGitHub/ansibleprac/ansible.cfg
  configured module search path = ['/Users/iyunakov/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /Users/iyunakov/MeGitHub/ansibleprac/.venv/lib/python3.10/site-packages/ansible
  ansible collection location = /Users/iyunakov/.ansible/collections:/usr/share/ansible/collections
  executable location = /Users/iyunakov/MeGitHub/ansibleprac/.venv/bin/ansible
  python version = 3.10.14 (main, Jun 26 2024, 18:07:39) [Clang 15.0.0 (clang-1500.3.9.4)] (/Users/iyunakov/MeGitHub/ansibleprac/.venv/bin/python)
  jinja version = 3.1.6
  libyaml = True
```

2. Подготовьте инвентарный файл для указания IP-адресов удаленных серверов.


Содержмимое файла [inventory.ini](inventory.ini)
```bash
[test]
ubuntu20 ansible_host=localhost ansible_port=2220 ansible_user=ansible

[web]
ubuntu22 ansible_host=localhost ansible_port=2222 ansible_user=ansible
ubuntu24 ansible_host=localhost ansible_port=2224 ansible_user=ansible
```

_. Проверка работы ansible
```bash
% ansible all -i inventory.ini -m ping

[WARNING]: Platform linux on host ubuntu22 is using the discovered Python interpreter at
/usr/bin/python3.10, but future installation of another Python interpreter could change the meaning
of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
ubuntu22 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.10"
    },
    "changed": false,
    "ping": "pong"
}
[WARNING]: Platform linux on host ubuntu20 is using the discovered Python interpreter at
/usr/bin/python3.8, but future installation of another Python interpreter could change the meaning
of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
ubuntu20 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.8"
    },
    "changed": false,
    "ping": "pong"
}
[WARNING]: Platform linux on host ubuntu24 is using the discovered Python interpreter at
/usr/bin/python3.12, but future installation of another Python interpreter could change the meaning
of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
ubuntu24 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "changed": false,
    "ping": "pong"
}
```


_. Структура плейбука + ролей

```yaml
- name: Install and configure Nginx
  hosts: ubuntu22
  become: true
  roles:
    - nginx

- name: Install and configure Apache
  hosts: ubuntu24
  become: true
  roles:
    - apache
```

```bash
% tree roles
roles
├── apache
│   ├── handlers
│   │   └── main.yml
│   ├── tasks
│   │   └── main.yml
│   └── templates
│       └── index.html.j2
├── nginx
│   ├── handlers
│   │   └── main.yml
│   ├── tasks
│   │   └── main.yml
│   └── templates
│       └── index.html.j2
└── roles
    └── nginx
```


_. Запуск плейбука
```bash
% ansible-playbook site.yml


PLAY [Install and configure Nginx] *****************************************************************

TASK [Gathering Facts] *****************************************************************************
[WARNING]: Platform linux on host ubuntu22 is using the discovered Python interpreter at
/usr/bin/python3.10, but future installation of another Python interpreter could change the meaning
of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [ubuntu22]

TASK [nginx : Install nginx] ***********************************************************************
changed: [ubuntu22]

TASK [nginx : Deploy index page] *******************************************************************
changed: [ubuntu22]

RUNNING HANDLER [nginx : restart nginx] ************************************************************
changed: [ubuntu22]

PLAY [Install and configure Apache] ****************************************************************

TASK [Gathering Facts] *****************************************************************************
[WARNING]: Platform linux on host ubuntu24 is using the discovered Python interpreter at
/usr/bin/python3.12, but future installation of another Python interpreter could change the meaning
of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [ubuntu24]

TASK [apache : Install apache] *********************************************************************
changed: [ubuntu24]

TASK [apache : Deploy index page] ******************************************************************
changed: [ubuntu24]

RUNNING HANDLER [apache : restart apache] **********************************************************
changed: [ubuntu24]

PLAY RECAP *****************************************************************************************
ubuntu22                   : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu24                   : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```



_. Проверка работы сервисов
```bash
% ansible ubuntu22 -m shell -a "curl -fsS http://127.0.0.1 | head"

[WARNING]: Platform linux on host ubuntu22 is using the discovered Python interpreter at
/usr/bin/python3.10, but future installation of another Python interpreter could change the meaning
of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
ubuntu22 | CHANGED | rc=0 >>
<h1>Nginx works on ubuntu22</h1>
(.venv) iyunakov@mbp-iyunakov-OZON-FVFG74SJQ05N
~/MeGitHub/ansibleprac 
% ansible ubuntu24 -m shell -a "curl -fsS http://127.0.0.1 | head"

[WARNING]: Platform linux on host ubuntu24 is using the discovered Python interpreter at
/usr/bin/python3.12, but future installation of another Python interpreter could change the meaning
of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
ubuntu24 | CHANGED | rc=0 >>
<h1>Apache works on ubuntu24</h1>
```



