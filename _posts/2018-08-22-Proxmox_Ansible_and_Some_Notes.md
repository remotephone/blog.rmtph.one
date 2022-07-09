---
layout: post
author: remotephone
title:  "Proxmox, Ansible, And Some Notes"
date:   2018-08-21 10:31:00 -0600
categories: [lab, homelab, proxmox, ansible, workflow]
largeimage: /images/avatar.jpg

---


I've been using Ansible a little bit now and while it has a very easy learning curve, there's some finer points I had some trouble figuring out. I'm still not the best with it, but being barely functional can still help you move worlds with a single playbook.

## Learning Ansible

Lots of ways to learn it, but these were some of the resources that were most useful.

This [quick start video](https://www.ansible.com/resources/videos/quick-start-video) is a good intro.

Start writing some simple [playbooks](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html). Some suggestions are:

- [Install some packages](https://docs.ansible.com/ansible/latest/modules/apt_module.html) (LXC is great to do this in since those containers can be blown away so easily)
- Add some [users](https://docs.ansible.com/ansible/latest/modules/user_module.html) and [groups](https://docs.ansible.com/ansible/latest/modules/group_module.html)
- [Copy](https://docs.ansible.com/ansible/latest/modules/copy_module.html) a file to a device

This was a [fantastic introduction](https://github.com/leucos/ansible-tuto) to Ansible - work through tasks step by step to understand Ansible feature by feature. If you only do one, do this one.

My general approach for using a module I've never used before is find it in the Ansible documentation, scroll down to the [examples](https://docs.ansible.com/ansible/latest/modules/copy_module.html#id4), find one that does close to what I want, modify it to suit my needs, and reach each of the [parameters](https://docs.ansible.com/ansible/latest/modules/copy_module.html#id2) after I have a skeleton of something that looks like it works.

## Roles

When I write a role, I tend to start with an empty directory with the structure all in place. Ansible's [official documentation](https://docs.ansible.com/ansible/2.5/user_guide/playbooks_reuse_roles.html#id5) gives this nice and neat sample structure I keep reusing.

~~~
site.yml
webservers.yml
fooservers.yml
roles/
   common/
     tasks/
     handlers/
     files/
     templates/
     vars/
     defaults/
     meta/
   webservers/
     tasks/
     defaults/
     meta/
~~~

I create empty directories and blank files and then fill them in as I go. This is really handy as it gives me reminders of what features (vars, handlers, etc) I don't always use that might make something I'm having trouble with easier.

## Using Ansible facts to rename Network Interfaces

I wrote an Ansible playbook that uses a couple of roles to configure my Proxmox servers. This came in handy the more I worked with Proxmox and broke things. I would make configuration changes I'd forget or bork storage in one new way or another and be unable to work my way backwards. Needless to say, I figured it'd save me some trouble to automate the Proxmox setup.

One task I had to repeat each time was renaming secondary network interfaces. In this [post](https://remotephone.github.io/2018/01/19/Proxmox_Lab_on_a_Slightly_Larger_Budget_Part1.html) I explained how to rename an interface manually. This isn't as easy as I hoped it'd be in Ansible since the interface MAC will be different for every NIC and I didn't want to statically configure it as a variable. I believe [this](https://unix.stackexchange.com/questions/335461/predictable-network-interface-names-break-vm-migration) post is what helped me answer the questions.

If you know the hosts you're working with have two intefaces, you can gather Ansible facts and figure out what you need to do with them. To make the following examples work, you'll need to have Ansible amd sshpass installed and working and add this to your /etc/ansible/hosts file, where 1.1.1.1 is your actual proxmox host IP:

~~~
[pve1]
1.1.1.1  
~~~

I put my task together bit by bit. Knowing I want to end up with this line in a file, I can start figuring out how to find the Mac address:

~~~
{% raw %}

SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="XX:XX:XX:XX:XX:XX", NAME="eno2"
{% endraw %}

~~~

First, figure out what interfaces ansible knows about.

~~~
{% raw %}

test@test:~$ sudo ansible all -m setup --connection=local -a 'filter=ansible_interfaces'
127.0.0.1 | SUCCESS => {
    "ansible_facts": {
        "ansible_interfaces": [
            "eno1",
            "enxabcd1234abcd1234",
            "lo"
        ]
    },
    "changed": false
}
{% endraw %}

~~~

You can then query Ansible for the interface that doesn't match the ones you expect. When you use the Ansible [set_fact](https://docs.ansible.com/ansible/latest/modules/set_fact_module.html) module, you can assign the resultant value of your request to a variable. To test what you're going to set your fact to, you can use debug messages.

I queried all the interfaces, piped that to the Ansible filter [difference](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#id15), told it to exclude the loop back and default interface and tell me what was left. This is the playbook I tested with:

~~~
{% raw %}

test@test:~$ cat simple.yml
---
- hosts: pve1
  tasks:
    - debug:
        msg: "{{ ansible_interfaces | difference(['lo',ansible_default_ipv4.alias]) | last }}"
{% endraw %}

~~~

Here's that playbook running:

~~~
{% raw %}

test@test:~$  sudo ansible-playbook simple.yml --ask-pass
SSH password:

PLAY [pve1] ********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************
ok: [10.0.0.11]

TASK [debug] *******************************************************************************************************************************************************
ok: [10.0.0.11] => {
    "msg": "enxabcd1234abcd1234"
}

PLAY RECAP *********************************************************************************************************************************************************
10.0.0.11                  : ok=2    changed=0    unreachable=0    failed=0
{% endraw %}

~~~

So now we have the name and set it as a fact, called second_nic. Then we take that and get it's mac address like this:

~~~ bash
{% raw %}
- name: get the name of the second interface so we can work with it
  set_fact:
    second_nic: "{{ ansible_interfaces | difference(['lo',ansible_default_ipv4.alias]) | last }}"

- name: wut
  set_fact:
    second_mac: "{{ hostvars[inventory_hostname]['ansible_' + second_nic]['macaddress'] }}"
{% endraw %}

~~~

The double curly braces are [jinja2 templating](https://docs.ansible.com/ansible/latest/user_guide/playbooks_templating.html), something else you should read about. It allows you to create outputs or inputs by applying parsing or processing logic to data. In the above case, we took all the host variables (hostvars), pulled the hostname, got the ansible_<Nic_name> (which is what anisble refers to the interface as), and request it's Mac address so we can set it to a new fact, second_mac.

Now, using this last task, I create the file, "/etc/udev/rules.d/10-network.rules" that will contain the configuration the OS will read at startup to name the interface. The [lineinfile](https://docs.ansible.com/ansible/latest/modules/lineinfile_module.html) module will find a line in a file by regex and replace it. If the file or line doesn't exist, its created with "create:yes".

~~~
{% raw %}

- name: Conveniently rename interfaces
  lineinfile:
    dest: /etc/udev/rules.d/10-network.rules
    regexp: 'SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="{{ second_mac }}", NAME="eno2"'
    line: 'SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="{{ second_mac }}", NAME="eno2"'
    create: yes
{% endraw %}

~~~

Badabing badaboom. Renamed network interface once you reboot the system.

## Wrapping up

Ansible can be as simple or complicated as you want it to be. You don't have to be the best at it to get a lot of value out of it either. I consider myself barely functional but have written some very useful things and you can too. Get some practice in LXC containers so you can break things freely and test before breaking VMs.
