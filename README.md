Openstack RDO Demo
==============
This demo is an ansible playbook that installs Openstack Mitaka on the reference topology using centos.
Quickstart: Run the demo
------------------------
(This assumes you are running Ansible 1.9.4 and Vagrant 1.8.4 on your host.)

    git clone https://github.com/cumulusnetworks/cldemo-vagrant
    cd cldemo-vagrant
    mv Vagrantfile-centos Vagrantfile
    vagrant up oob-mgmt-server oob-mgmt-switch leaf01 leaf02 spine01 spine02 server01 server02 leaf03 leaf04 server03 server04
    vagrant ssh oob-mgmt-server
    sudo su - cumulus
    git clone https://github.com/cumulusnetworks/cldemo-openstack-rdo
    cd cldemo-openstack-rdo
    ansible-playbook run-demo.yml


Launch a single VM instance on the shared provider network.

    ssh server01
    . demo-openrc
    openstack server create --flavor m1.nano --image cirros --nic net-id=provider --security-group default cirros01
    ping 192.168.0.101 #wait until it starts responding
    ssh cirros@192.168.0.101
    # password is "cubswin:)"
    hostname
    ping google.com
    exit

Create a private tenant network and launch a VM in it. Give it a floating IP address.

    neutron net-create demonet
    neutron subnet-create --name demonet --dns-nameserver 8.8.8.8 --gateway 200.0.0.1 demonet 200.0.0.0/24
    neutron router-create demorouter
    neutron router-interface-add demorouter demonet
    neutron router-gateway-set demorouter provider
    openstack server create --flavor m1.nano --image cirros --nic net-id=demonet --security-group default cirros02
    openstack ip floating create provider
    # 192.168.0.103 is assumed to be the floating IP from the last command
    openstack ip floating add 192.168.0.103 cirros02
    ping 192.168.0.103
    ssh cirros@192.168.0.103
    hostname
    ping google.com
    exit


To access the horizon terminal, open two new terminals and run one of the
following commands in each. When prompted for a password, enter `CumulusLinux!`.

    ssh -L 8080:server01:80 cumulus@127.0.0.1 -p 2222
    ssh -L 6080:localhost:8888 cumulus@127.0.0.1 -p 2222 ssh -L 8888:controller:6080 server01

Leave these terminals open - they will create tunnels that will allow you to
access the Horizon dashboard and the noVNC console clients using your web
browser. Navigate to  http://localhost:8080/horizon in your web browser.
Log in with the demo user (password is `demo`) and the default domain. You
should be able to see the last two steps. Open one of them and go to the "console"
tab. You should be able to access the serial console from this web page.
