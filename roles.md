#### ROLES
  * create a folder called roles inside ansible directory
  * lets try ansible-galaxy init httpd --offline
  * command: tree and see the branches
  * Inside httpd folder, you have many directories 
  * Inside tasks directory, you have main.yml file (which will match dependencies for your variables). e.g. (see below)

```yml

---
- name: Installing httpd
  yum:
    name: httpd
    state: latest
  notify: restart httpd
- name: Copying the file
  copy:
    src: Index.html
    dest: "{{ dest }}/index.html"
    mode: 0766
  notify: restart httpd

```
1. Let's have our source file (index.html) inside files directory
2. vi.index.html and write something
3. Let's go to handlers directory and change main.yml file with something below:
```yml
---
- name: restart httpd
  service:
    name: httpd
    state: restarted
 ```
 4. Go inside var directory and edit main.yml
 ```yml
 dest: "var/www/html"
 ```
 ##### Note: Your default path for ansible roles is set inside /etc/ansible/ansible.cfg --vi and search for something with ansible_path, you may change the path as per your need
 1. Let's go inside playbook directory and create file with soemthing like rolegiven.yml
 ```yml
 ---
 - name: Installing httpd
   hosts: db (or whatever you group is)
   become: true
   roles:
     - ../roles/httpd
     ```
