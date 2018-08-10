PUBLIC_IP = '192.168.1.104' # Public IP on your network
BRIDGE = 'eno1' # Set this to the host network interface that connects to the Internet
MATTERMOST_PASSWORD = 'really_secure_password' # The password for the Mattermost database user

# You shouldn't have to change anything under this line
GALERA_CLUSTER_IPS = ['192.168.33.101','192.168.33.102','192.168.33.103'] # The Internal IPs for the Galera nodes.
HAPROXY_IP = '192.168.33.104' # Internal IP of the HAProxy server
GALERA_CLUSTER_PREFIX = 'galera' # The hostname prefix for the galera cluster
MYSQL_ROOT_PASSWORD = 'mysql_root_password' # Self explanatory

Vagrant.configure("2") do |config|
	config.vm.box = "bento/ubuntu-16.04"
	config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
	config.vm.provider "virtualbox" do |v|
		v.memory = 4096
		v.cpus = 1
		v.customize ["modifyvm", :id, "--cableconnected1", "on"]
	end

	node_ips = GALERA_CLUSTER_IPS
	haproxy_conf = File.read('haproxy.cfg')

	node_ips.each_with_index do |node_ip, index|
		box_hostname = "#{GALERA_CLUSTER_PREFIX}#{index+1}"

		config.vm.define box_hostname do |box|
			box.vm.hostname = box_hostname
			box.vm.network :private_network, ip: node_ip
			# box.vm.network :public_network, ip: node_ip, bridge: BRIDGE

			box.vm.provision :shell, run: 'once', inline: <<-SHELL
				apt-get -q -y update
				apt-get install -y -q software-properties-common
				apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
        		add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://nyc2.mirrors.digitalocean.com/mariadb/repo/10.1/ubuntu xenial main'
        		apt-get -q -y update
        		export DEBIAN_FRONTEND=noninteractive
        		debconf-set-selections <<< 'mariadb-server-10.0 mysql-server/root_password password #{MYSQL_ROOT_PASSWORD}'
        		debconf-set-selections <<< 'mariadb-server-10.0 mysql-server/root_password_again password #{MYSQL_ROOT_PASSWORD}'
        		apt-get install -y -q mariadb-server
        		service mysql stop
        		ufw allow 3306,4567,4568,4444/tcp
        		ufw allow 4567/udp
        		cp /vagrant/galera.cnf /etc/mysql/conf.d/galera.cnf
        		cp /vagrant/db_setup.sql /home/vagrant/db_setup.sql
			SHELL

			replace_hostname_cmd = "sed -i 's/THIS_HOSTNAME/#{box_hostname}/' /etc/mysql/conf.d/galera.cnf"
			replace_all_ips_cmd = "sed -i 's/ALL_IPS/#{node_ips.join(',')}/' /etc/mysql/conf.d/galera.cnf"
			replace_this_ip_cmd = "sed -i 's/THIS_IP/#{node_ip}/' /etc/mysql/conf.d/galera.cnf"

			replace_haproxy_ip_cmd = "sed -i 's/HAPROXY_IP/#{HAPROXY_IP}/' /home/vagrant/db_setup.sql"
			replace_password_ip_cmd = "sed -i 's/MATTERMOST_PASSWORD/#{MATTERMOST_PASSWORD}/' /home/vagrant/db_setup.sql"

			box.vm.provision :shell, inline: replace_hostname_cmd, run: 'once'
			box.vm.provision :shell, inline: replace_all_ips_cmd, run: 'once'
			box.vm.provision :shell, inline: replace_this_ip_cmd, run: 'once'

			if index == 0
				box.vm.provision :shell, inline: 'galera_new_cluster', run: 'once'
			else
				box.vm.provision :shell, inline: 'service mysql start', run: 'once'
			end

			box.vm.provision :shell, inline: replace_haproxy_ip_cmd, run: 'once'
			box.vm.provision :shell, inline: replace_password_ip_cmd, run: 'once'

			box.vm.provision :shell, inline: "mysql -uroot -p#{MYSQL_ROOT_PASSWORD} < /home/vagrant/db_setup.sql", run: 'once'
			box.vm.provision :shell, inline: "rm /home/vagrant/db_setup.sql", run: 'once'
		end
	end

	config.vm.define 'haproxy1' do |box|
		box.vm.hostname = 'haproxy1'
		box.vm.network :private_network, ip: HAPROXY_IP
		box.vm.network :public_network, ip: '192.168.1.104', bridge: BRIDGE

		box.vm.provision :shell, run: 'once', inline: <<-SHELL
			apt-get -q -y update
			apt-get install -y -q haproxy
			service haproxy stop
		SHELL
		

		haproxy_conf.sub! 'HAPROXY_IP', HAPROXY_IP

		node_ips.each_with_index do |node_ip, i|
			haproxy_conf += "    server #{GALERA_CLUSTER_PREFIX}#{i} #{node_ip} check weight 1\n"
		end

		box.vm.provision :shell, inline: "echo '#{haproxy_conf}' >> /etc/haproxy/haproxy.cfg", run: 'once'
		box.vm.provision :shell, inline: 'service haproxy start', run: 'once'
		
	end

end