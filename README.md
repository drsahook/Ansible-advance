---
# Ansible-advance
# Bad ansible! This playbook is an example of poor/bad practices!
# Bad practices may include:
#
#   Poor formatting and structure
#   Poor use of YAML - but good enough to parse
#   Inconsistent style
#   Incorrect use of modules
#   Poor module choice
#   Unclear names
#   Hard coding / poor use of variables
#   Roles - what are roles?
#   Bare variables
#   No use of handlers

- name: configuration
  hosts: all
  gather_facts: false # remove later! speeds up testing
  become: true

  tasks:
  - name: enable repos
    template:
      src: ./open_three-tier-app.repo
      dest: /etc/yum.repos.d/open_three-tier-app.repo
      mode: 0644

- name: deploy haproxy
  hosts: frontends
  gather_facts: false # remove later! speeds up testing
  become: true

  tasks:
  - name: http
    package:
      name: httpie
      state: latest
  - name: install HAProxy
    yum:
      name=haproxy state=latest
  - name: enable HAProxy
    service:
      name: haproxy
      state: started
  - name: configure haproxy
    template:
      src: ./haproxy.cfg.j2
      dest: /etc/haproxy/haproxy.cfg
  - name: restart HAproxy
    service:
      name: haproxy
      state: restarted


- name: deploy tomcat
  hosts: apps
  gather_facts: false
  become: true

  tasks:
  - name: install tomcat
    package:
      name: tomcat
      state: latest
  - name: enable tomcat at boot
    service:
      name: tomcat
      enabled: yes

  - name: create ansible tomcat directory
    file:
      path: /usr/share/tomcat/webapps/ROOT
      state: directory

  - name: copy static index.html to tomcat webapps/ansible/index.html
    template:
      src: index.html.j2
      dest: /usr/share/tomcat/webapps/ROOT/index.html
      mode: 0644

  - name: start tomcat
    service:
      name: tomcat
      state: started

- name: index.html on app 1
  hosts: app1
  gather_facts: false
  become: true

  tasks:
  - name: copy static index.html to tomcat webapps/ansible/index.html
    template:
      src: index.html.app1
      dest: /usr/share/tomcat/webapps/ansible/index.html

- name: index.html on app 1
  hosts: app2
  gather_facts: false
  become: true

  tasks:
  - name: copy static index.html to tomcat webapps/ansible/index.html
    template:
      src: index.html.app2
      dest: /usr/share/tomcat/webapps/ansible/index.html

- name: deploy postgres
  hosts: apps
  gather_facts: false
  become: true
  hosts: appdbs
  tasks:
  - name: install progress
    command: "yum install -y postgresql-server"

  #- name: install postgres
  #  yum:
  #    name: postgresql-server
  #    state: latest
  - name: enable apache at boot
    service:
      name: postgresql
      enabled: yes
  - name: tell user to finish setting up postgres
    debug:
      msg: "Either uncomment the postgres setup or manually login and initialize"

 # only run the next 2 tasks once!
  - name: initilize postgres
    command: postgresql-setup initdb
#  - name: initilize postgres some more
#    command: chkconfig postgresql on
  - name: start postgres
    service:
      name: postgresql.service
      state: started

- name: deploy apache
  hosts: apps
  gather_facts: false
  become: true
  hosts: apps
  tasks:
  - name: install apache
    yum:
      name: httpd
      state: latest
  - name: enable apache at boot
    service:
      name: httpd
      enabled: yes
  - name: start apache
    service:
      name: httpd
      state: started
