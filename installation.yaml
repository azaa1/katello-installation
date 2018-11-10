---
- name: Configure Hostname
  hosts: test

  vars_prompt:
    - name: "IP"
      prompt: "Enter Server's IP Address"
      private: no

    - name: "host_name"
      prompt: "Enter Your Host Name"
      private: no

  tasks:
    - name: Change hostname
      when: ansible_distribution == "CentOS" and ansible_distribution_major_version == "7"
      command: hostnamectl set-hostname {{host_name}}


    - name: Configure /etc/hosts
      when: ansible_distribution == "CentOS"
      shell: echo {{IP}} {{host_name}} >> /etc/hosts


    - name: Install NTP on CentOS7
      yum:
        name: ntp
        state: present

    - name: Configure /etc/hosts
      command: "timedatectl set-timezone America/Chicago"

    - name: Restart NTP service
      systemd:
        name: ntpd
        state: restarted

    - name: Install FirewallD
      yum:
        name: firewalld
        state: present

    - name: Start FirewallD
      systemd:
        name: firewalld
        state: started

    - name: Configure FirewallD
      command: "firewall-cmd --permanent --zone=public --add-port=80/tcp --add-port=443/tcp --add-port=5647/tcp --add-port=9090/tcp"

    - name: Configure FirewallD
      command: "firewall-cmd --permanent --zone=public --add-port=8140/tcp --add-port=8443/tcp --add-port=8000/tcp --add-port=67/udp --add-port=68/udp --add-port=69/udp"

    - name: Restart FirewallD
      systemd:
        name: firewalld
        state: restarted


    - name: Add katello repo
      yum:
        name: http://fedorapeople.org/groups/katello/releases/yum/3.8/katello/el7/x86_64/katello-repos-latest.rpm
        state: present

    - name: Add foreman repo
      yum:
        name: http://yum.theforeman.org/releases/1.13/el7/x86_64/foreman-release.rpm
        state: present

    - name: Add puppet repo
      yum:
        name: http://yum.puppetlabs.com/puppetlabs-release-el-7.noarch.rpm
        state: present

    - name: Add epel-release
      yum:
        name: http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
        state: present

    - name: Install Required Packages
      yum:
        name: "{{ packages }}"
      vars:
        packages:
          - foreman-release-scl
          - python-django

    - name: Ensure the yum package index is up to date
      yum:
        update_cache: yes
        name: '*'
        state: latest

    - name: Install Katello
      yum:
        name: katello
        state: present
      notify:
      - foreman-installer

  handlers:
    - name: foreman-installer
      command: foreman-installer --scenario katello --foreman-admin-username admin --foreman-admin-password password
