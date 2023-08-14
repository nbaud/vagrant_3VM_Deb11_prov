# vagrant_3VM_Deb11_prov

The vagrantfile creates 3 VM in Virtualbox 7.0.10 with update of guest additions, of apt and upgrade and installation of python and ansible on the master node with ssh access to the workers

# First bring up the master node which will generate its own key pair

vagrant up master

# Then bring up the worker nodes

vagrant up w1 w2

# Like this, the copy of the ssh keys from the master to the worker nodes will work fine with vagrant

Also, make sure to download the file for guest addition and to install it with the code of vagrant, not with the automatic update from virtualbox, as it is buggy and will cause issues.
