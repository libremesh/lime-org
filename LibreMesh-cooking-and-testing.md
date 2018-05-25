* LibreMesh cooking and testing
** cook with Docker
sudo systemctl start docker
sudo docker run -v "$(pwd)":/app cooker --<parameters>
** test on VM
first, install virt-manager virt-bootstrap (Arch/Manjaro)

sudo systemctl start libvirtd virtlogd

