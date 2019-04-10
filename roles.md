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
