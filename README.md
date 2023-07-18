# VLAN LACP
Для выполнения этого действия требуется установить приложением git:
`git clone https://github.com/altyn-kenzhebaev/vlan-hw24.git`
В текущей директории появится папка с именем репозитория. В данном случае vlan-hw24. Ознакомимся с содержимым:
```
cd vlan-hw24
ls -l
README.md
ansible
Vagrantfile
```
Здесь:
- ansible - папка с плэйбуками
- README.md - файл с данным руководством
- Vagrantfile - файл описывающий виртуальную инфраструктуру для `Vagrant`
Запускаем ВМ:
```
vagrant up
```

## VLAN
Добавляем следующее в playbook.yml:
```
# playbook.yml
- name: set up vlan1
  #Настройка будет производиться на хостах testClient1 и testServer1
  hosts: testClient1,testServer1
  #Настройка производится от root-пользователя
  become: yes
  tasks:
  #Добавление темплейта в файл /etc/sysconfig/network-scripts/ifcfg-vlan1
  - name: set up vlan1
    template:
      src: ifcfg-vlan1.j2
      dest: /etc/sysconfig/network-scripts/ifcfg-vlan1
      owner: root
      group: root
      mode: 0644
  
  #Перезапуск службы NetworkManager
  - name: restart network for vlan1
    service:
      name: NetworkManager
      state: restarted

- name: set up vlan2
  hosts: testClient2,testServer2
  become: yes
  tasks:
  - name: set up vlan2
    template:
      src: 50-cloud-init.yaml.j2
      dest: /etc/netplan/50-cloud-init.yaml 
      owner: root
      group: root
      mode: 0644

  - name: apply set up vlan2
    shell: netplan apply
    become: true

# ifcfg-vlan1.j2
VLAN=yes
TYPE=Vlan
PHYSDEV=eth1
VLAN_ID={{ vlan_id }}
VLAN_NAME_TYPE=DEV_PLUS_VID_NO_PAD
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
IPADDR={{ vlan_ip }}
PREFIX=24
NAME=vlan{{ vlan_id }}
DEVICE=eth1.{{ vlan_id }}
ONBOOT=yes

# 50-cloud-init.yaml.j2
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    version: 2
    ethernets:
        enp0s3:
            dhcp4: true
     
        enp0s8: {}
    
    vlans:
        vlan{{ vlan_id }}:
          id: {{ vlan_id }}
          link: enp0s8
          dhcp4: no
          addresses: [{{ vlan_ip }}/24]
```
## Обьединение в bond
Добавляем следующее в playbook.yml:
```
# playbook.yml

- name: set up bond0
  hosts: inetRouter,centralRouter
  become: yes
  tasks:
  - name: set up ifcfg-bond0
    template:
      src: ifcfg-bond0.j2
      dest: /etc/sysconfig/network-scripts/ifcfg-bond0
      owner: root
      group: root
      mode: 0644
  
  - name: set up eth1,eth2
    copy: 
      src: "{{ item }}" 
      dest: /etc/sysconfig/network-scripts/
      owner: root
      group: root
      mode: 0644
    with_items:
      - templates/ifcfg-eth1
      - templates/ifcfg-eth2
  #Перезагрузка хостов 
  - name: restart hosts for bond0
    reboot:
      reboot_timeout: 3600

# ifcfg-bond0.j2
DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
IPADDR={{ bond_ip }}
NETMASK=255.255.255.252
ONBOOT=yes
BOOTPROTO=static
BONDING_OPTS="mode=1 miimon=100 fail_over_mac=1"
NM_CONTROLLED=yes
USERCTL=no
```
## проверить работу c отключением интерфейсов
На сервере inetRouter пингуем centralRouter:
```
[root@inetRouter ~]# ping 192.168.255.2
PING 192.168.255.2 (192.168.255.2) 56(84) bytes of data.
64 bytes from 192.168.255.2: icmp_seq=1 ttl=64 time=0.952 ms
64 bytes from 192.168.255.2: icmp_seq=2 ttl=64 time=0.930 ms
64 bytes from 192.168.255.2: icmp_seq=3 ttl=64 time=0.760 ms
64 bytes from 192.168.255.2: icmp_seq=4 ttl=64 time=0.914 ms
64 bytes from 192.168.255.2: icmp_seq=5 ttl=64 time=0.875 ms
64 bytes from 192.168.255.2: icmp_seq=6 ttl=64 time=0.444 ms
64 bytes from 192.168.255.2: icmp_seq=7 ttl=64 time=0.482 ms
64 bytes from 192.168.255.2: icmp_seq=8 ttl=64 time=0.702 ms
64 bytes from 192.168.255.2: icmp_seq=9 ttl=64 time=0.886 ms
```
Непрерывая команду ping выполняем на centralRouter следующее:
```
ip l set down eth1
```
Пинги не были прерваны, обьединение портов функционирует успешно!