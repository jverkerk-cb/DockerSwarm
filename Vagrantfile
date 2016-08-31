# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

$script = <<SCRIPT
echo Installing dependencies...
sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
echo installing docker engine
sudo yum -y install docker-engine
sudo usermod -aG docker vagrant
sudo systemctl enable docker.service
sudo systemctl start docker.service
sudo mkdir -p /docker-nginx/html
sudo tee /docker-nginx/html/index.html <<-'EOF'
<!DOCTYPE html>
<html>
<body>

<p>Click the button to display the hostname of the current URL.</p>

<button onclick="myFunction()">Try it</button>

<p id="demo"></p>

<script>
function myFunction() {
    var x = location.hostname;
    document.getElementById("demo").innerHTML= x;
}
</script>

</body>
</html>
EOF
sudo chmod -R 655 /docker-nginx
SCRIPT

$hascript = <<HASCRIPT
yum install -y haproxy
sudo tee /etc/haproxy/haproxy.cfg <<-'EOF'
global
   log /dev/log local0
   log /dev/log local1 notice
   chroot /var/lib/haproxy
   stats socket /run/haproxy/admin.sock mode 660 level admin
   stats timeout 30s
   user haproxy
   group haproxy
   daemon

defaults
   log global
   mode http
   option httplog
   option dontlognull
   timeout connect 5000
   timeout client 50000
   timeout server 50000

frontend http_front
   bind *:80
   stats uri /haproxy?stats
   default_backend http_back

backend http_back
   balance roundrobin
   server manager1 172.20.20.10:80 check
   server worker1 172.20.20.11:80 check
   server worker2 172.20.20.12:80 check
EOF
sudo systemctl start haproxy
HASCRIPT

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  
  #set global image to use (move into each section to change this)
  config.vm.box = "bento/centos-7.2"

  config.vm.define "haproxy" do |haproxy|
      haproxy.vm.hostname = "haproxy"
      haproxy.vm.network "private_network", ip: "172.20.20.15"
      haproxy.vm.provision "shell", inline: $hascript
  end

#  config.vm.define "manager1" do |manager1|
#      manager1.vm.hostname = "manager1"
#      manager1.vm.network "private_network", ip: "172.20.20.10"
#      manager1.vm.provision "shell", inline: $script
#  end
#  config.vm.define "worker1" do |worker1|
#      worker1.vm.hostname = "worker1"
#      worker1.vm.network "private_network", ip: "172.20.20.11"
#      worker1.vm.provision "shell", inline: $script
#  end
#
#  config.vm.define "worker2" do |worker2|
#      worker2.vm.hostname = "worker2"
#      worker2.vm.network "private_network", ip: "172.20.20.12"
#      worker2.vm.provision "shell", inline: $script
#  end
end
