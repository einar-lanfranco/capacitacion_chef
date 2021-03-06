!SLIDE subsection center transition=scrollVert
# NTP con chef

!SLIDE commandline transition=scrollVert
# NTP con chef
## Creamos el cookbook

	$ mkdir -p cookbooks/ntp 

## Creamos el recipe
	$ mkdir -p cookbooks/ntp/recipes
	$ vi cookbooks/ntp/recipes/default.rb

*Es aconsejable tener una receta para `server.rb` y otra para `client.rb` en
este caso, pero por cuestiones de simplicidad, utilizamos únicamente el recipe `default.rb`*

!SLIDE smbullets transition=scrollVert
# ¿Qué deberíamos hacer?

* Instalar el paquete de NTP
* Configurar `/etc/ntp.conf`
* Habilitar el servicio para que bootee e iniciar el servicio

!SLIDE transition=scrollVert
# El recipe

En la edición de `recipes/default.rb` tendríamos:

	@@@ ruby
	package 'ntp'

	template '/etc/ntp.conf' do
		source    'ntp.conf.erb'
		notifies  :restart, 'service[ntp]'
	end

	service 'ntp' do
		action [:enable, :start]
	end

!SLIDE smbullets incremental transition=scrollVert
# ¿Cómo se usa el resource template?

* ¿Qué es el formato erb?
* ¿Cómo uso variables en el template?
* ¿Pueden haber diferentes versiones del mismo template?

!SLIDE commandline transition=scrollVert
# El template para ntp

El template `/etc/ntp.conf` debemos crearlo en el directorio `templates/default`
de mi cookbook con el nombre `ntp.conf.erb` porque es lo que indicamso en la
recipe `default.rb` anteriormente:

	$ mkdir -p cookbooks/ntp/templates/default
	$ vi cookbooks/ntp/templates/default/ntp.conf.erb

!SLIDE smaller transition=scrollVert
# El template para ntp

	@@@ ruby
	# This file was generated by Chef for '<%= node['fqdn'] %>'.
	# Do NOT edit this file by hand!

	restrict default kod nomodify notrap nopeer noquery
	restrict -6 default kod nomodify notrap nopeer noquery
	restrict 127.0.0.1
	restrict -6 ::1
	server <%= node[:ntp][:server]%>
	server  127.127.1.0     # local clock
	driftfile /var/lib/ntp/drift
	keys /etc/ntp/keys

!SLIDE transition=scrollVert
# Nos quedaría definir los atributos usados

* Los valores usados en el template son atributos, a decir:
  * `node['fqdn']`: valor provisto por ohai
  * `node[:ntp][:server]`: valor provisto por este cookbook. **Hay que definirlo**

!SLIDE commandline transition=scrollVert
# Attributes ntp

Editamos `attributes/default.rb`

	$ mkdir  -p cookbooks/ntp/attributes
	$ vi cookbooks/ntp/attributes/default.rb

!SLIDE transition=scrollVert
# Attributes ntp

	@@@ ruby
	default[:ntp][:server] = '0.pool.ntp.org'
	

!SLIDE commandline smaller incremental transition=scrollVert
# Probando la receta ntp

Inicializamos `vagrant` en el directorio en el que estamos trabajando. Considere
que sería el directorio donde se encuentra el directorio `cookbooks` en el cuál
creamos el cookbook `ntp`. 


	$ vagrant init ubuntu-1204 \
		http://vagrantbox.desarrollo.cespi.unlp.edu.ar/pub/ubuntu-12.04.2-cespi-amd64.box
		A `Vagrantfile` has been placed in this directory. You are now
		ready to `vagrant up` your first virtual environment! Please read
		the comments in the Vagrantfile as well as documentation on
		`vagrantup.com` for more information on using Vagrant.


!SLIDE smaller transition=scrollVert
# Probando la receta ntp

Editamos `Vagrantfile` indicando que usaremos chef con la receta de ntp, quedando nuestro archivo:

	@@@ ruby
	Vagrant.configure("2") do |config|
		config.vm.box = "ubuntu-1204"
		config.vm.box_url = "http://vagrantbo......"
		config.vm.network :private_network, ip: "192.168.33.10"
		config.vm.provision :chef_solo do |chef|
			 chef.cookbooks_path = "cookbooks"
			 chef.add_recipe "ntp"
		end
	end

!SLIDE commandline incremental smaller transition=scrollVert
	$ vagrant up
	Bringing machine 'default' up with 'virtualbox' provider...
	[default] Importing base box 'ubuntu-1204'...
	[default] Matching MAC address for NAT networking...
	[default] Setting the name of the VM...
	[default] Forwarding ports...
	[default] -- 22 => 2222 (adapter 1)
	[default] Booting VM...
	[default] Waiting for VM to boot. This can take a few minutes.
	[default] VM booted and ready for use!
	[default] Configuring and enabling network interfaces...
	[default] Mounting shared folders...
	[default] -- /vagrant
	[default] -- /tmp/vagrant-chef-1/chef-solo-1/cookbooks
	[default] Running provisioner: chef_solo...
	Generating chef JSON and uploading...
	Running chef-solo...
	stdin: is not a tty
	[2013-06-13T13:11:43+00:00] INFO: *** Chef 11.4.4 ***
	[2013-06-13T13:11:45+00:00] INFO: Setting the run_list to ["recipe[ntp]"] from
	JSON
	[2013-06-13T13:11:45+00:00] INFO: Run List is [recipe[ntp]]
	[2013-06-13T13:11:45+00:00] INFO: Run List expands to [ntp]
	[2013-06-13T13:11:45+00:00] INFO: Starting Chef Run for
	ubuntu-12.04.2-cespi-amd64
	[2013-06-13T13:11:45+00:00] INFO: Running start handlers
	[2013-06-13T13:11:45+00:00] INFO: Start handlers complete.
	[2013-06-13T13:11:45+00:00] INFO: Processing package[ntp] action install
	(ntp::default line 1)
	[2013-06-13T13:11:45+00:00] INFO: Processing template[/etc/ntp.conf] action
	create (ntp::default line 3)
	[2013-06-13T13:11:46+00:00] INFO: template[/etc/ntp.conf] backed up to
	/var/chef/backup/etc/ntp.conf.chef-20130613131145
	[2013-06-13T13:11:46+00:00] INFO: template[/etc/ntp.conf] updated content
	[2013-06-13T13:11:46+00:00] INFO: Processing service[ntp] action enable
	(ntp::default line 8)
	[2013-06-13T13:11:46+00:00] INFO: Processing service[ntp] action start
	(ntp::default line 8)
	[2013-06-13T13:11:46+00:00] INFO: template[/etc/ntp.conf] sending restart action
	to service[ntp] (delayed)
	[2013-06-13T13:11:46+00:00] INFO: Processing service[ntp] action restart
	(ntp::default line 8)
	[2013-06-13T13:11:47+00:00] INFO: service[ntp] restarted
	[2013-06-13T13:11:47+00:00] INFO: Chef Run complete in 2.319794345 seconds
	[2013-06-13T13:11:47+00:00] INFO: Running report handlers
	[2013-06-13T13:11:47+00:00] INFO: Report handlers complete

!SLIDE commandline small transition=scrollVert
# Verificamos todo

Conectamos por ssh: `vagrant ssh`

Analizar el template generado: `cat /etc/ntp.conf`

Usar el cliente de ntp:

	$ ntpq -c peers
			 remote           refid      st t when poll reach   delay   offset  jitter
		==============================================================================
		 66.60.22.202    105.134.105.216  4 u   12   64    3   10.850   10.758   1.087
		*LOCAL(0)        .LOCL.           5 l   11   64    3    0.000    0.000   0.031



!SLIDE smaller transition=scrollVert
# Personalizamos el nodo

Editamos `Vagrantfile` cambiando el `attribute` provisto por nuestro cookbook al
ntp de cespi

	@@@ ruby
	Vagrant.configure("2") do |config|
		config.vm.box = "ubuntu-1204"
		config.vm.box_url = "http://vagrantbo......"
		config.vm.network :private_network, ip: "192.168.33.10"
		config.vm.provision :chef_solo do |chef|
			 chef.cookbooks_path = "cookbooks"
			 chef.add_recipe "ntp"
			 chef.json = {·
  			:ntp => {
  				:server => "cronos.unlp.edu.ar"
  			}
  		}
		end
	end

!SLIDE commandline small transition=scrollVert
# Verificamos todo

Conectamos por ssh: `vagrant ssh`

Analizar el template generado: `cat /etc/ntp.conf`

Usar el cliente de ntp:

	$ ntpq -c peers
			 remote           refid      st t when poll reach   delay   offset  jitter
		==============================================================================
		 163.10.0.84     163.10.43.120    2 u   17   64    1   10.061    8.278   0.008
		*LOCAL(0)        .LOCL.           5 l   16   64    1    0.000    0.000   0.008

!SLIDE smbullets small transition=scrollVert
# La decepción

* Ingresamos a [http://community.opscode.com](http://community.opscode.com)
  * [El caso de NTP](http://community.opscode.com/cookbooks/ntp)
* Siempre hay que buscar si lo que necesitamos ya existe
  * Buscar también en google
  * Considerar el espacio en [github de opscode](https://github.com/opscode-cookbooks/)
* Muchas veces lo que existe debe modificarse
  * Usemos el workflow propuesto por github forkeando el repositorio
  * Si mejoramos funcionalidad contribuimos
* Algunas reglas que seguir:
  * [Guías de la comunidad](http://docs.opscode.com/community_guidelines.html)
  * [Contribuir con los cookbooks de Opscode](http://wiki.opscode.com/display/chef/How+to+Contribute+to+Opscode+Cookbooks)

!SLIDE commandline incremental transition=scrollVert
# Descargar el ejemplo
## Para descargar el ejemplo dado

	$ git clone https://github.com/chrodriguez/capacitacion_chef.git
	$ cd capacitacion_chef 
	$ cd samples/01-chef-dev/02-ntp


!SLIDE smbullets transition=scrollVert
# Un cookbook más
* Hagamos una nueva receta que cree una base de datos **mysql**  y un usuario que pueda
conectarse
* Buscamos qué cookbooks nos resuelven el problema

!SLIDE smbullets transition=scrollVert
# Un cookbook más

* El cookbook que nos resuelve nuestra necesidad es [database](https://github.com/opscode-cookbooks/database)
* Este cookbook depende de:
  * mysql
  * postgresql
  * xfs
  * aws
* Que a su vez dependen de otros cookbooks:
  * openssl
  * build-essential
* **¿Cómo descargamos estos cookbooks bajo la carpeta `cookbooks/` en nuestra nueva receta?**

