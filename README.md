# domotica
Setup servidor de domotica

bios  -> restart after power loss

### Instalar Debian.

Environment XFCE

[x] SSH

su: root / [pwd_su]

Usuario: [user - domotica] / [pw_usr]

### Conceder persmios SU a usuario desde terminal:

```
su - 
usermod -aG sudo domotica 
exit 
exit 
```

### Set grub boot timeout = 0 

`sudo nano /etc/default/grub`

modificar TIMEOUT a 0; save, exit

`sudo update-grub`

### Docker + portainer 
```
sudo apt-get install ca-certificates curl gnupg lsb-release  
sudo mkdir -p /etc/apt/keyrings  
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg  
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null  
sudo apt-get update  
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin  
sudo docker run -itd -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /docker/portainer:/data portainer/portainer-ce
 ```

`sudo chown domotica -R /docker`

Acceder via navegador [IP]:9000

### samba:  

`sudo apt-get install samba`

`sudo nano /etc/samba/smb.conf`

al final del fichero añadir debajo de ;[profiles] 
```
;[profiles] 
; comment = Users profiles 
; path = /home/samba/profiles 
; guest ok = no 
; browseable = no 
; create mask = 0600 
; directory mask = 0700 

[docker] 
comment = docker domotica 
path = /docker 
guest ok = no 
writeable = yes 
write list = domotica 
force user = root 
force group = root 
browseable = yes 
create mask = 0755 
directory mask = 0755 
```

añadir usuario/password, habilitar usuario y reiniciar servicio: 

```
sudo smbpasswd -L -a domotica 
[password] 
[repeat password]  
sudo smbpasswd -L -e domotica 
sudo /etc/init.d/smbd restart 
```

### Node-red 

Desde portainer/stacks > new stack > copiar & deploy stack

```
services: 
  node-red: 
    image: nodered/node-red:latest 
    container_name: nodered 
    environment: 
      - TZ=Europe/Madrid 
    volumes: 
      - /docker/nodered/data:/data 
      - /etc/localtime:/etc/localtime:ro
    ports: 
      - 1880:1880
    restart: always
```
stop container

`sudo chown domotica:domotica -R /docker`
    
restart container

Desde nodered [IP]:1880

manage pallete

node-red-contrib-influxdb 
node-red-contrib-spreadsheet-in 
node-red-contrib-moment 
node-red-dashboard 
node-red-contrib-ui-media 


### Influxdb2 

Desde portainer/stacks > new stack > copiar & deploy stack

```
services:
  influxdb:
    container_name: influxdb
    image: influxdb:latest
    volumes:
      - /docker/influxdb/var/lib/influxdb2:/var/lib/influxdb2:rw
    ports:
      - "8086:8086"
    restart: always
    environment:
      - TZ=Europe/Madrid
```
tokens, set bucket to persistance = custom / 2Y

### Grafana 

Desde portainer/stacks > new stack > copiar & deploy stack

initial password: admin/admin


```
services: 
  grafana: 
    container_name: grafana 
    image: grafana/grafana 
    user: "0" 
    restart: always 
    ports: 
      - 3000:3000 
    volumes: 
      - /docker/grafana/var/lib/grafana:/var/lib/grafana 
```

Configure InfluxDB authentication:

- Token authentication
  
	- Under Custom HTTP Headers, select Add Header. Provide your InfluxDB API token:
		- Header: Enter Authorization
		- Value: Use the Tokenschema and provide your InfluxDB API token. For example:


'Token y0uR5uP3rSecr3tT0k3n'

 - Under InfluxDB Details, do the following:
	- Database: Enter the database name mapped to your InfluxDB bucket
	- HTTP Method: Select GET

