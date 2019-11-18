# MongoDB_Sharding

## Requirement Cluster

- 2 Config Server
- 1 Query Router
- 3 Shard Server

## Keterangan Cluster

- Config Server 1 -> Alamat IP : 192.168.33.11, RAM : 512MB
- Config Server 2 -> Alamat IP : 192.168.33.12, RAM : 512MB
- Config Query Router -> Alamat IP : 192.168.33.13, RAM : 512MB
- Config Shard 1 -> Alamat IP : 192.168.33.14, RAM : 512MB
- Config Server 2 -> Alamat IP : 192.168.33.15, RAM : 512MB
- Config Server 3 -> Alamat IP : 192.168.33.16, RAM : 512MB

## Implementasi

### <h4>Konfigurasi Vagrant
Sebelum implementasi MongoDB, kita harus membuat konfigurasi file vagrant yang berisi beberapa cluster :

```
$ vagrant init
```

Perintah di atas untuk membuat file **vagrantfile**. Untuk konfigurasi dari **vagrantfile** bisa dilihat dibawah ini :

```
Vagrant.configure("2") do |config|

  config.vm.define "mongo_config_1" do |mongo_config_1|
    mongo_config_1.vm.hostname = "mongo-config-1"
    mongo_config_1.vm.box = "bento/ubuntu-18.04"
    mongo_config_1.vm.network "private_network", ip: "192.168.33.11"
    
    mongo_config_1.vm.provider "virtualbox" do |vb|
      vb.name = "mongo-config-1"
      vb.gui = false
      vb.memory = "512"
    end

    mongo_config_1.vm.provision "shell", path: "provision/allhosts.sh", privileged: false
  end

  config.vm.define "mongo_config_2" do |mongo_config_2|
    mongo_config_2.vm.hostname = "mongo-config-2"
    mongo_config_2.vm.box = "bento/ubuntu-18.04"
    mongo_config_2.vm.network "private_network", ip: "192.168.33.12"
    
    mongo_config_2.vm.provider "virtualbox" do |vb|
      vb.name = "mongo-config-2"
      vb.gui = false
      vb.memory = "512"
    end

    mongo_config_2.vm.provision "shell", path: "provision/allhosts.sh", privileged: false
  end

  config.vm.define "mongo_query_router" do |mongo_query_router|
    mongo_query_router.vm.hostname = "mongo-query-router"
    mongo_query_router.vm.box = "bento/ubuntu-18.04"
    mongo_query_router.vm.network "private_network", ip: "192.168.33.13"
    
    mongo_query_router.vm.provider "virtualbox" do |vb|
      vb.name = "mongo-query-router"
      vb.gui = false
      vb.memory = "512"
    end

    mongo_query_router.vm.provision "shell", path: "provision/allhosts.sh", privileged: false
  end

  config.vm.define "mongo_shard_1" do |mongo_shard_1|
    mongo_shard_1.vm.hostname = "mongo-shard-1"
    mongo_shard_1.vm.box = "bento/ubuntu-18.04"
    mongo_shard_1.vm.network "private_network", ip: "192.168.33.14"
        
    mongo_shard_1.vm.provider "virtualbox" do |vb|
      vb.name = "mongo-shard-1"
      vb.gui = false
      vb.memory = "512"
    end

    mongo_shard_1.vm.provision "shell", path: "provision/allhosts.sh", privileged: false
  end

  config.vm.define "mongo_shard_2" do |mongo_shard_2|
    mongo_shard_2.vm.hostname = "mongo-shard-2"
    mongo_shard_2.vm.box = "bento/ubuntu-18.04"
    mongo_shard_2.vm.network "private_network", ip: "192.168.33.15"
    
    mongo_shard_2.vm.provider "virtualbox" do |vb|
      vb.name = "mongo-shard-2"
      vb.gui = false
      vb.memory = "512"
    end

    mongo_shard_2.vm.provision "shell", path: "provision/allhosts.sh", privileged: false
  end

  config.vm.define "mongo_shard_3" do |mongo_shard_3|
    mongo_shard_3.vm.hostname = "mongo-shard-3"
    mongo_shard_3.vm.box = "bento/ubuntu-18.04"
    mongo_shard_3.vm.network "private_network", ip: "192.168.33.16"
    
    mongo_shard_3.vm.provider "virtualbox" do |vb|
      vb.name = "mongo-shard-3"
      vb.gui = false
      vb.memory = "512"
    end

    mongo_shard_3.vm.provision "shell", path: "provision/allhosts.sh", privileged: false
  end

end
```

### Provisioning
Untuk setiap node (config, shard, router) dikasih provision dengan script **mongo_config_1.vm.provision "shell", path: "provision/allhosts.sh"**. Di dalam *allhost.sh* terdapat konfigurasi sebagai berikut :

```
# Add hostname
sudo bash -c \\"echo '192.168.33.11 mongo-config-1' >> /etc/hosts\\"
sudo bash -c \\"echo '192.168.33.12 mongo-config-2' >> /etc/hosts\\"
sudo bash -c \\"echo '192.168.33.13 mongo-query-router' >> /etc/hosts\\"
sudo bash -c \\"echo '192.168.33.14 mongo-shard-1' >> /etc/hosts\\"
sudo bash -c \\"echo '192.168.33.15 mongo-shard-2' >> /etc/hosts\\"
sudo bash -c \\"echo '192.168.33.16 mongo-shard-3' >> /etc/hosts\\"

# Copy APT sources list
sudo cp /vagrant/sources/sources.list /etc/apt/
sudo cp /vagrant/sources/mongodb-org-4.2.list /etc/apt/sources.list.d/

# Add MongoDB repo key
sudo apt-get install gnupg
wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -

# Update Repository
sudo apt-get update
# sudo apt-get upgrade -y

# Install MongoDB
sudo apt-get install -y mongodb-org

# Start MongoDB
sudo service mongod start
```
File provision *allhosts.sh* tersebut digunakan untuk install MongoDB pada tiap node ketika *vagrant up*

- <h4>Build Vagrant</h4>
Membuild vagrant dengan cara

```
$ vagrant up
```

Dengan otomatis membuat virtual machine dan menjalankan provision menggunakan box bento/ubuntu-18.04, jika belum terunduh maka vagrant otomatis mengunduhnya.

- <h4>Konfigurasi Config Server</h4>

- Langkah Pertama

Mengunjungi setiap node config server

```
$ vagrant ssh [nama file]
```

- Langkah Kedua

Masuk ke shell mongo masing - masing config server

```
$ mongo
```

Lalu buat username pada MongoDB

```
use admin
db.createUser(
      {
          user: "mongo-admin",
          pwd: "password",
          roles: [ "root" ]
      }
  )
```

- Langkah Ketiga

Konfigurasi */etc/mongod.conf" di tiap config server

```
# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# network interfaces
net:
  port: 27019
  bindIp: (AlamatIP tiap node)


# how the process runs
processManagement:
  timeZoneInfo: /usr/share/zoneinfo

#security:

#operationProfiling:

#replication:
replication:
  replSetName: configReplSet

#sharding:
sharding:
  clusterRole: "configsvr"

## Enterprise-Only Options:

#auditLog:

#snmp:
```

- Langkah Keempat

Melakukan restart MongoDB di tiap config server
```
sudo systemctl restart mongod
```

- Langkah Kelima

Salah satu config server yaitu config server yang pertama melakukan

```
$ mongo 192.168.33.11:27019 -u mongo-admin -p --authenticationDatabase admin
```

- Langkah Keenam

Inisialisasi replika set

```
rs.initiate( { _id: "configReplSet", configsvr: true, members: [ { _id: 0, host: "192.168.33.11:27019" }, { _id: 1, host: "192.168.33.12:27019" } ] } )
```

### Konfigurasi Query Router

- Langkah Pertama

Mengakses node query router dengan cara

```
$ vagrant ssh mongo-query-router
```

- Langkah Kedua

Konfigurasi file baru yang bernama **/etc/mongos.conf** dengan script sebagai berikut :

```
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongos.log

# network interfaces
net:
  port: 27017
  bindIp: 192.168.33.13


sharding:
  configDB: configReplSet/192.168.33.11:27019,192.168.33.12:27019
```

- Langkah Ketiga

Konfigurasi file baru yang bernama **/lib/systemd/system/mongos.service** denagn script sebagai berikut :

```
[Unit]
Description=Mongo Cluster Router
After=network.target

[Service]
User=mongodb
Group=mongodb
ExecStart=/usr/bin/mongos --config /etc/mongos.conf
# file size
LimitFSIZE=infinity
# cpu time
LimitCPU=infinity
# virtual memory size
LimitAS=infinity
# open files
LimitNOFILE=64000
# processes/threads
LimitNPROC=64000
# total threads (user+kernel)
TasksMax=infinity
TasksAccounting=false

[Install]
WantedBy=multi-user.target
```

- Langkah Keempat

Menghentinkan mongod

```
sudo systemctl stop mongod
```

- Langkah Kelima

Menjalankan mongos yang sudah kita buat sebelumnya

```
sudo systemctl enable mongos.service
sudo systemctl start mongos
```

- <h4>Konfigurasi Shard Server</h4>

- Langkah Pertama

Mengakses node shard server dengan cara sebagai berikut :

```
$ vagrant ssh (nama file)
```

- Langkah Kedua

Konfigurasi **/etc/mongod.conf** di setiap node shard server dengan cukup merubah AlamatIp dan menambahkan

```
sharding:
  clusterRole: "shardsvr"
```

- Langkah Ketiga

Salah satu node shard server (shard 1) masuk ke dalam query router dengan cara sebagai berikut :

```
mongo 192.168.33.13:27017 -u mongo-admin -p --authenticationDatabase admin
```

- Langkah Keempat

Untuk penambahan shard, pada shell mongo dilakukan

```
sh.addShard("192.168.33.14:27017") -> AlamatIP shard 1
sh.addShard("192.168.33.15:27017") -> AlamatIP shard 2
sh.addShard("192.168.33.16:27017") -> AlamatIP shard 3
```

### Import Dataset

- Langkah Pertama

Kita download datasetnya terlebih dahulu

```
wget https://data.baltimorecity.gov/api/views/k5ry-ef3g/rows.csv?accessType=DOWNLOAD
```

- Langkah Kedua

Lalu kita import datasetnya dengan cara

```
ongoimport -h 192.168.33.13:27017 -u mongo-admin -d restaurantDB -c restaurantcl --type CSV --file rows.csv\?accessT
ype\=DOWNLOAD --headerline --authenticationDatabase admin
```
