 Hi ,
 
 some please assist me on this issue , i ma created simpe demo in network setup with ansible, 
 while running playbook under task is showing as changed, also unable to copy the content to different directory.
 
 root@NetworkAutomation-1:~/GNS3_Project# ls
ESTABLISH  ansible.cfg  inventory  output  showospf.yml
root@NetworkAutomation-1:~/GNS3_Project#
root@NetworkAutomation-1:~/GNS3_Project# ansible --version
ansible 2.9.4
  config file = /root/GNS3_Project/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.17 (default, Nov  7 2019, 10:07:09) [GCC 7.4.0]
root@NetworkAutomation-1:~/GNS3_Project#

----
root@NetworkAutomation-1:~/GNS3_Project# cat inventory
R1 ansible_host=192.168.122.90 ansible_user=pandi ansible_ssh_pass=pandi ansible_connection=ssh
R2 ansible_host=192.168.122.92 ansible_user=pandi ansible_ssh_pass=pandi ansible_connection=ssh
R3 ansible_host=192.168.122.91 ansible_user=pandi ansible_ssh_pass=pandi ansible_connection=ssh

[routers]
R1
R2
R3
root@NetworkAutomation-1:~/GNS3_Project#


My play book
root@NetworkAutomation-1:~/GNS3_Project# cat showospf.yml
---
-
  gather_facts: false
  hosts: all
  name: "get ospf neighbor"
  debugger: on_failed
  tasks:
    -
      name: "show ospf Neighbor"
      raw: "sh ip ospf nei"
      register: print_output
    -
      debug: var=print_output.stdout_lines
    -
      copy: content="{{ print_output.stdout[0] }}" dest="/root/GNS3_Project/output/{{ inventory_hostname }}.txt"
      name: Copy the file to different directory
root@NetworkAutomation-1:~/GNS3_Project#
----
output with check:

PLAY [get ospf neighbor] ***************************************************************************************

TASK [show ospf Neighbor] **************************************************************************************
skipping: [R1]
skipping: [R2]
skipping: [R3]

TASK [debug] ***************************************************************************************************
ok: [R2] => {
    "print_output.stdout_lines": "VARIABLE IS NOT DEFINED!"
}
ok: [R1] => {
    "print_output.stdout_lines": "VARIABLE IS NOT DEFINED!"
}
ok: [R3] => {
    "print_output.stdout_lines": "VARIABLE IS NOT DEFINED!"
}

TASK [Copy the file to different directory] ********************************************************************
fatal: [R1]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'dict object' has no attribute 'stdout'\n\nThe error appears to be in '/root/GNS3_Project/showospf.yml': line 16, column 7, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n    -\n      copy: content=\"{{ print_output.stdout[0] }}\" dest=\"/root/GNS3_Project/output/{{ inventory_hostname }}.txt\"\n      ^ here\nWe could be wrong, but this one looks like it might be an issue with\nmissing quotes. Always quote template expression brackets when they\nstart a value. For instance:\n\n    with_items:\n      - {{ foo }}\n\nShould be written as:\n\n    with_items:\n      - \"{{ foo }}\"\n"}
[R1] TASK: Copy the file to different directory (debug)> p
***SyntaxError:SyntaxError('unexpected EOF while parsing', ('<string>', 0, 0, ''))
[R1] TASK: Copy the file to different directory (debug)> q
User interrupted execution---
  
  
  without check:
  root@NetworkAutomation-1:~/GNS3_Project# ansible-playbook showospf.yml
[WARNING]: Skipping plugin (/usr/lib/python2.7/dist-packages/ansible/plugins/connection/napalm.py) as it seems
to be invalid: invalid syntax (cisco_base_connection.py, line 143)


PLAY [get ospf neighbor] ***************************************************************************************

TASK [show ospf Neighbor] **************************************************************************************
changed: [R1]
changed: [R2]
changed: [R3]

<>
TASK [Copy the file to different directory] ********************************************************************
[WARNING]: Unhandled error in Python interpreter discovery for host R3: unexpected output from Python
interpreter discovery

[WARNING]: Unhandled error in Python interpreter discovery for host R2: unexpected output from Python
interpreter discovery

[WARNING]: Unhandled error in Python interpreter discovery for host R1: unexpected output from Python
interpreter discovery

[WARNING]: sftp transfer mechanism failed on [192.168.122.91]. Use ANSIBLE_DEBUG=1 to see detailed information

[WARNING]: sftp transfer mechanism failed on [192.168.122.92]. Use ANSIBLE_DEBUG=1 to see detailed information

[WARNING]: sftp transfer mechanism failed on [192.168.122.90]. Use ANSIBLE_DEBUG=1 to see detailed information

[WARNING]: scp transfer mechanism failed on [192.168.122.92]. Use ANSIBLE_DEBUG=1 to see detailed information

[WARNING]: scp transfer mechanism failed on [192.168.122.90]. Use ANSIBLE_DEBUG=1 to see detailed information

[WARNING]: scp transfer mechanism failed on [192.168.122.91]. Use ANSIBLE_DEBUG=1 to see detailed information



