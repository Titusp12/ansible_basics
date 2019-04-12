#### NOTE: Steps to integrate Ansible with AWS

1. Import Controller's Public key to AWS (Key Pair).
2. Update Controller's security group by allowing port 80 (HTTP)
3. Configure Boto

```yml

---
- name: Installing boto in Controller Machine
  hosts: localhost
  become: true
  tasks:
    - name: Installing pip
      easy_install:
        name: pip
        state: latest

    - name: Installing Boto
      pip:
name: boto

```
4. Configure Infra
 a. Create user in IAM and input ACCESS key in vars ACCESS, copy SECRET key for command line
 b. Change MYKEY to whatever key name i have
 c. Get subnet ID from default (VPC --> Subnet --> Default) and put in vpc_subnet_id (in two places)

```yml 
---
- import_playbook: configure-boto.yml

- name: Launching an EC2 machine
  hosts: localhost
  become: true
  vars:
    ACCESS: AKIAJZTXLKZHCQ5PGKKQ
    SECRET: mysecretkey
    REGION: us-east-1
    MYKEY: AwsKamalayKey
  tasks:
    - name: Launching an EC2 Machine for Nginx Loadbalancer    
      ec2:
        aws_access_key: "{{ ACCESS }}"
        aws_secret_key: "{{ SECRET }}"
        aws_region: "{{ REGION }}"
        key_name: "{{ MYKEY }}"
        instance_type: t2.micro
        user_data: |
                   #!/bin/sh
                   sudo apt-get install python -y
        image: ami-009307dd5dee4e17b
        wait: yes
        count: 1
        instance_tags:
          Name: nginx-loadbalancer
        vpc_subnet_id: subnet-3a885f66
        assign_public_ip: yes
      register: output
    
    - name: Updating the Hosts file with Nginx Private IP
      lineinfile:
        path: hosts
        regexp: "^[nginx]"
        insertafter: "[nginx]"
        line: "{{ item.private_ip }}"
      with_items: "{{ output.instances }}"

    - name: Launching Two EC2 machines for mysite website
      ec2:
        aws_access_key: "{{ ACCESS }}"
        aws_secret_key: "{{ SECRET }}"
        aws_region: "{{ REGION }}"
        key_name: "{{ MYKEY }}"
        instance_type: t2.micro
        image: ami-6871a115
        wait: yes
        count: 2
        instance_tags:
          Name: mysite-webserver
        vpc_subnet_id: subnet-3a885f66
        assign_public_ip: yes
      register: output

    - name: Updating Hosts file with private IPs of webservers
      lineinfile:
        path: hosts
        regexp: "^[webservers]"
        insertafter: "[webservers]"
        line: "{{ item.private_ip }}"
with_items: "{{ output.instances }}"

```
5. Configure mysite

```yml

---
- name: Configuring my website
  hosts: webservers
  become: true
  tasks:
    - name: Installing httpd 
      yum: name=httpd state=present
  
    - name: Copying mysite contents
      template:
        src: files/mysite.conf.j2
        dest: /var/www/html/index.html

    - name: Restarting the service
service: name=httpd state=restarted

```

6. Configure nginx

```yml

---
- name: Configuring Nginx load balancer
  hosts: nginx
  become: true
  user: ubuntu
  tasks:
    - name: Creating the configuration file for our website mysite
      template:
        src: files/nginx.conf.j2 
        dest: /etc/nginx/conf.d/mysite.conf
    
    - name: Disabling the Default site
      replace:
        dest: /etc/nginx/nginx.conf
        regexp: "include /etc/nginx/sites-enabled/"
        replace: "#include /etc/nginx/sites-enabled/"
 
    - name: Restarting the nginx service
service: name=nginx state=restarted

```

7. Create hosts

```yml

[webservers]

[nginx]

```
