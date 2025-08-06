Voici un exemple complet et prêt à l'emploi d’un playbook Ansible pour :

Installer Apache Tomcat

Déployer un fichier .war sur Tomcat

✅ Prérequis
Machine distante : Ubuntu/Debian (ou adaptation requise pour CentOS).

Ansible installé sur la machine de contrôle.

Java installé sur la cible (ou inclure l’installation dans le playbook).

Variable tomcat_user (ex: ubuntu) avec accès sudo.

📁 Arborescence suggérée :
markdown
Copier
Modifier
tomcat_playbook/
├── deploy_tomcat.yml
├── files/
│   └── app.war
└── templates/
    └── tomcat.service.j2
🔧 deploy_tomcat.yml
yaml
Copier
Modifier
---
- name: Installer Tomcat et déployer une application
  hosts: tomcat
  become: yes
  vars:
    tomcat_version: 9.0.89
    tomcat_user: tomcat
    tomcat_group: tomcat
    tomcat_install_dir: /opt/tomcat
    war_file: app.war
  tasks:

    - name: Installer Java
      apt:
        name: openjdk-11-jdk
        state: present
        update_cache: yes

    - name: Créer un utilisateur pour Tomcat
      user:
        name: "{{ tomcat_user }}"
        shell: /bin/false

    - name: Télécharger Tomcat
      get_url:
        url: "https://downloads.apache.org/tomcat/tomcat-9/v{{ tomcat_version }}/bin/apache-tomcat-{{ tomcat_version }}.tar.gz"
        dest: /tmp/apache-tomcat.tar.gz

    - name: Créer le dossier d'installation Tomcat
      file:
        path: "{{ tomcat_install_dir }}"
        state: directory
        owner: root
        group: root
        mode: '0755'

    - name: Extraire Tomcat
      unarchive:
        src: /tmp/apache-tomcat.tar.gz
        dest: "{{ tomcat_install_dir }}"
        remote_src: yes
        extra_opts: [--strip-components=1]

    - name: Donner les droits au user tomcat
      file:
        path: "{{ tomcat_install_dir }}"
        state: directory
        recurse: yes
        owner: "{{ tomcat_user }}"
        group: "{{ tomcat_group }}"

    - name: Copier le service systemd
      template:
        src: templates/tomcat.service.j2
        dest: /etc/systemd/system/tomcat.service
        mode: '0644'

    - name: Démarrer et activer le service Tomcat
      systemd:
        name: tomcat
        daemon_reload: yes
        enabled: yes
        state: started

    - name: Copier le fichier WAR dans webapps
      copy:
        src: "files/{{ war_file }}"
        dest: "{{ tomcat_install_dir }}/webapps/{{ war_file }}"
        owner: "{{ tomcat_user }}"
        group: "{{ tomcat_group }}"
        mode: '0644'
📄 templates/tomcat.service.j2
ini
Copier
Modifier
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
Environment=CATALINA_PID={{ tomcat_install_dir }}/temp/tomcat.pid
Environment=CATALINA_HOME={{ tomcat_install_dir }}
Environment=CATALINA_BASE={{ tomcat_install_dir }}
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
ExecStart={{ tomcat_install_dir }}/bin/startup.sh
ExecStop={{ tomcat_install_dir }}/bin/shutdown.sh

User={{ tomcat_user }}
Group={{ tomcat_group }}
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
📦 Lancer le playbook
bash
Copier
Modifier
ansible-playbook -i hosts deploy_tomcat.yml
Assurez-vous que le fichier hosts contient votre cible sous [tomcat].

Souhaitez-vous que je vous prépare un fichier .zip avec tous les fichiers ?








Vous avez dit :
export cela en markdown pour github


ChatGPT a dit :
Voici le contenu exporté en Markdown pour GitHub, avec une structure lisible, des blocs de code correctement formatés et des sections claires pour un dépôt Ansible.

📦 Ansible Playbook — Installer Apache Tomcat et Déployer un WAR
Ce playbook installe Apache Tomcat 9 sur une machine distante, configure le service systemd et déploie une application .war.

📁 Structure du projet
markdown
Copier
Modifier
tomcat_playbook/
├── deploy_tomcat.yml
├── inventory
├── files/
│   └── app.war
└── templates/
    └── tomcat.service.j2
🔧 deploy_tomcat.yml
yaml
Copier
Modifier
---
- name: Installer Tomcat et déployer une application
  hosts: tomcat
  become: yes
  vars:
    tomcat_version: 9.0.89
    tomcat_user: tomcat
    tomcat_group: tomcat
    tomcat_install_dir: /opt/tomcat
    war_file: app.war
  tasks:

    - name: Installer Java
      apt:
        name: openjdk-11-jdk
        state: present
        update_cache: yes

    - name: Créer un utilisateur pour Tomcat
      user:
        name: "{{ tomcat_user }}"
        shell: /bin/false

    - name: Télécharger Tomcat
      get_url:
        url: "https://downloads.apache.org/tomcat/tomcat-9/v{{ tomcat_version }}/bin/apache-tomcat-{{ tomcat_version }}.tar.gz"
        dest: /tmp/apache-tomcat.tar.gz

    - name: Créer le dossier d'installation Tomcat
      file:
        path: "{{ tomcat_install_dir }}"
        state: directory
        owner: root
        group: root
        mode: '0755'

    - name: Extraire Tomcat
      unarchive:
        src: /tmp/apache-tomcat.tar.gz
        dest: "{{ tomcat_install_dir }}"
        remote_src: yes
        extra_opts: [--strip-components=1]

    - name: Donner les droits au user tomcat
      file:
        path: "{{ tomcat_install_dir }}"
        state: directory
        recurse: yes
        owner: "{{ tomcat_user }}"
        group: "{{ tomcat_group }}"

    - name: Copier le service systemd
      template:
        src: templates/tomcat.service.j2
        dest: /etc/systemd/system/tomcat.service
        mode: '0644'

    - name: Démarrer et activer le service Tomcat
      systemd:
        name: tomcat
        daemon_reload: yes
        enabled: yes
        state: started

    - name: Copier le fichier WAR dans webapps
      copy:
        src: "files/{{ war_file }}"
        dest: "{{ tomcat_install_dir }}/webapps/{{ war_file }}"
        owner: "{{ tomcat_user }}"
        group: "{{ tomcat_group }}"
        mode: '0644'
📄 templates/tomcat.service.j2
ini
Copier
Modifier
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
Environment=CATALINA_PID={{ tomcat_install_dir }}/temp/tomcat.pid
Environment=CATALINA_HOME={{ tomcat_install_dir }}
Environment=CATALINA_BASE={{ tomcat_install_dir }}
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
ExecStart={{ tomcat_install_dir }}/bin/startup.sh
ExecStop={{ tomcat_install_dir }}/bin/shutdown.sh

User={{ tomcat_user }}
Group={{ tomcat_group }}
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
📦 files/app.war
Placez ici votre fichier WAR à déployer. Par exemple :

Copier
Modifier
files/
└── app.war
🧪 inventory
Exemple d’inventaire :

bash
Copier
Modifier
[tomcat]
your.remote.host ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
🚀 Lancer le playbook
bash
Copier
Modifier
ansible-playbook -i inventory deploy_tomcat.yml
📌 Remarques
Le fichier .war est copié directement dans webapps/, Tomcat le déploiera automatiquement.

Vous pouvez adapter la version de Tomcat dans la variable tomcat_version.

Vérifiez que le port 8080 est ouvert sur votre machine distante.

Souhaitez-vous que je vous génère une archive .zip avec tous les fichiers prêts à déposer sur GitHub ?








Vous avez dit :
oui


ChatGPT a dit :
✅ Voici l'archive ZIP contenant le playbook Ansible complet pour installer Tomcat et déployer un fichier WAR :

📦 Télécharger ansible_tomcat_playbook.zip

Elle contient :

deploy_tomcat.yml (le playbook)

templates/tomcat.service.j2 (template systemd)

files/ (répertoire pour votre fichier .war)

inventory (exemple d’inventaire)

Tu peux directement l’utiliser ou l’importer dans un repo GitHub. Si tu veux que je t’ajoute aussi un README.md, je peux le générer aussi. # ansible-deploy-tomcat
Playbook ansible de deploiement tomcat
