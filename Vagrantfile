# -*- mode: ruby -*-
# vi: set ft=ruby :

box_url  = 'https://s3.amazonaws.com/easybibdeployment/imagineeasy-ubuntu-14.04.3_virtualbox-4.3.26r98988_chef-11.10.4_1.box'
box_file = 'imagineeasy-ubuntu-14.04.3_virtualbox-4.3.26r98988_chef-11.10.4_1'

Vagrant.configure(2) do |config|

  config.vm.box = box_file
  config.vm.box_url = box_url
  config.vm.hostname = 'checkmk'

  config.vm.network "private_network", virtualbox__intnet: "net_mon", ip: "10.10.10.17"
  config.vm.network "forwarded_port", guest: 80, host: 9080

  config.vm.synced_folder "data", "/vagrant_data"

  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo apt-get upgrade -y
    sudo apt-get install -y apache2 apache2-bin apache2-data apache2-mpm-prefork curl debugedit dialog fonts-liberation fping graphviz lcab libapache2-mod-fcgid libapache2-mod-proxy-html libaprutil1-dbd-sqlite3 libaprutil1-ldap libart-2.0-2 libcdt5 libcgraph6 libcurl3 libdbi1 libevent-1.4-2 libfile-copy-recursive-perl libgsf-1-114 libgsf-1-common libgvc6 libgvpr2 libice6 libldb1 liblua5.2-0 libmcrypt4 libnet-snmp-perl libnspr4 libnss3 libnss3-nssdb libntdb1 libpango1.0-0 libpangox-1.0-0 libpathplan4 libperl5.18 libpoppler44 libpython2.7 libradiusclient-ng2 librpm3 librpmbuild3 librpmio3 librpmsign1 libsensors4 libsm6 libsmbclient libsnmp-base libsnmp-perl libsnmp30 libtalloc2 libtdb1 libtevent0 libwbclient0 libwebp5 libwebpmux1 libxaw7 libxmu6 libxt6 php-pear php5-cgi php5-cli php5-common php5-gd php5-json php5-mcrypt php5-readline php5-sqlite poppler-utils pyro python-crypto python-imaging python-ldap python-ldb python-netsnmp python-ntdb python-pil python-renderpm python-reportlab python-reportlab-accel python-samba python-support python-talloc python-tdb rpm rpm-common rpm2cpio samba-common samba-common-bin samba-libs smbclient snmp ssl-cert traceroute update-inetd xinetd
    sudo dpkg -i /vagrant_data/check-mk-raw-1.2.6p12_0.trusty_amd64.deb
    omd create ies
    omd start ies
    echo
    echo "**********************************************"
    echo
    echo "You may want to configure this site. To do so:"
    echo "# vagrant ssh -c 'omd config ies'"
    echo
    echo "**********************************************"
    echo
    ip=$(ip -o a s eth1 | grep /2 | awk '{print $4}' | cut -d/ -f 1)
    echo "In your browser, go to:"
    echo " - http://127.0.0.1:9080/ies/"
    echo
    echo "Default credentials are => omdadmin:omd"
    echo
    echo "Happy monitoring!"
  SHELL
end
