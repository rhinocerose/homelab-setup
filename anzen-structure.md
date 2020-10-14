# Directory structure
```
├── inventory.yml
├── site.yml
├── setup.yml
├── playbook.yml
├── roles/
│   ├── provision/
│   │   ├── tasks/
│   │   │   ├── setup.yml
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   ├── vars/
│   │   │   └── vault.yml
│   │   ├── defaults/
│   │   │   └── main.yml
│   ├── prerequisites/
│   │   ├── tasks/
│   │   │   ├── setup.yml
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   ├── vars/
│   │   │   └── vault.yml
│   │   ├── defaults/
│   │   │   └── main.yml
│   │   ├── templates/
│   │   │   └── main.yml
│   │   ├── files/
│   │   │   ├── CAN_Cell
|   │   │   |   ├── file1
│   │   │   |   └── file2
```
