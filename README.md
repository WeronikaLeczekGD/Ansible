# Ansible

## [1. Create ansible inventory with information about hosts](#ad-1)

## [2. Create ansible playbook for deploying spring-petclinic application. It should include:](#ad-2)

a) Install required software depending of application artifact (java based or docker based)

b) Take specific artifact version (jar file/docker image) from local machine or Nexus and deploy it to application server.

## [3. Deploy application to remote host with ansible](#ad-3)

## [4. Check results in browser](#ad-4)





## Ad 1

my invetory.yaml:

```
application_server:
  hosts:
    18.206.170.76:
      ansible_python_interpreter: /usr/bin/python3

  vars:
    ansible_user: ec2-user
    ansible_ssh_private_key_file: /Users/wleczek/Downloads/spring-project-EC2.pem
```

## Ad 2

my playbook.yalm:

```
- name: Deploy Spring PetClinic Application
  hosts: application_server
  become: true
  vars:
    artifact_version: latest
    artifact_name: spring-petclinic-petclinic
  tasks:
    - name: Install Docker
      apt:
        name: docker.io
        state: present
      when: ansible_os_family == "Debian"

    - name: Install python3-pip
      become: true
      yum:
        name: python3-pip
        state: present

    - name: Install docker-py library
      become: true
      pip:
        name: docker
        executable: pip3
        state: present

    - name: Install Docker
      yum:
        name: docker
        state: present
      when: ansible_os_family == "RedHat"

    - name: Start Docker service
      service:
        name: docker
        state: started


    - name: Save Docker image as TAR file
      docker_image:
        name: wleczek/main:1
        archive_path: /tmp/{{ artifact_name }}:{{ artifact_version }}.tar
        source: local


    - name: Copy Docker image TAR file to remote host
      ansible.builtin.copy:
        src: /tmp/{{ artifact_name }}:{{ artifact_version }}.tar
        dest: /tmp/{{ artifact_name }}:{{ artifact_version }}.tar
        mode: '0644'
        remote_src: true

    - name: Load Docker image from archive
      docker_image:
        name: wleczek/main:1
        load_path: /tmp/{{ artifact_name }}:{{ artifact_version }}.tar
        source: load

    - name: Start Docker container
      docker_container:
        name: spring-petclinic
        image: wleczek/main:1
        ports:
          - "8080:8080"
        state: started
```

## Ad 3

Deploying application to remote host with ansible:

![Screenshot 2023-02-17 at 20 54 03](https://user-images.githubusercontent.com/114099418/219783545-fa3ae8b5-b7a0-404e-96fc-6f1c9c0ba613.png)

## Ad 4

Checking results in browser:

![Screenshot 2023-02-17 at 20 53 15](https://user-images.githubusercontent.com/114099418/219783674-c3156a29-ac69-4e10-8318-63a7956ccf99.png)

