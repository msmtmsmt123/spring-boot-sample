# spring-boot-sample
spring boot sample with cloud by pkrss

author blogs: http://blogs.pkrss.com  
author website: https://www.pkrss.com  

# What

1. test two demo project a and b call other feignclient service
1. test zuul api proxy
1. test docker with spring boot
     
# china aliyun docker accelerate
docker for windows: Setting > Daemon > Registry mirrors: add: xxxxxx.mirror.aliyuncs.com

# preprare

i used windows 10, install "consul" and "docker for windows".
my host network ip is 188.199.1.228. so my host windows add one host record: hosts.local => 188.199.1.228

# create new node

see:  
Get started with Docker Machine and a local VM: https://docs.docker.com/machine/get-started/
Microsoft Hyper-V:https://docs.docker.com/machine/drivers/hyper-v/
Install and Create a Docker Swarm: https://docs.docker.com/swarm/install-w-machine/

in china, download https://github.com/boot2docker/boot2docker/releases/download/v17.05.0-ce/boot2docker.iso save to %userprofile%\.docker\machine\cache\boot2docker.iso
in china, docker-machine create --engine-registry-mirror "https://k4qqsfsl.mirror.aliyuncs.com"
or: sudo sed -i "s|EXTRA_ARGS='|EXTRA_ARGS='--registry-mirror=https://k4qqsfsl.mirror.aliyuncs.com |g" /var/lib/boot2docker/profile

create hypver private network switcher: "Private".

PS C:\WINDOWS\system32> docker-machine create -d hyperv --hyperv-virtual-switch "Primary Virtual Switch" --hyperv-disk-size "2000" --hyperv-memory "512" --engine-registry-mirror "https://k4qqsfsl.mirror.aliyuncs.com" local

PS C:\WINDOWS\system32> docker-machine ls
   NAME    ACTIVE   DRIVER   STATE     URL                                     SWARM   DOCKER        ERRORS
   local   -        hyperv   Running   tcp://192.168.3.131:2376           v17.03.2-ce
PS C:\WINDOWS\system32> docker-machine ssh local
docker@local:~$ docker run --restart=always -d -v /data:/data -p 8300:8300 -p 8301:8301 -p 8301:8301/udp -p 8302:8302 -p 8302:8302/udp -p 8400:8400 -p 8500:8500 -p 8600:8600 consul agent -server -bootstrap-expect 2 -ui -client 0.0.0.0 -advertise 192.168.3.131 -node=local
        
PS C:\WINDOWS\system32> docker-machine create -d hyperv --hyperv-virtual-switch "Primary Virtual Switch" --hyperv-disk-size "3000" --hyperv-memory "1024" --swarm --swarm-master --swarm-discovery consul://192.168.3.131/swarm --engine-registry-mirror "https://k4qqsfsl.mirror.aliyuncs.com" master1
PS C:\WINDOWS\system32> docker-machine create -d hyperv --hyperv-virtual-switch "Primary Virtual Switch" --hyperv-disk-size "3000" --hyperv-memory "1024" --swarm --swarm-discovery consul://192.168.3.131/swarm --engine-registry-mirror "https://k4qqsfsl.mirror.aliyuncs.com" worker1
PS C:\WINDOWS\system32> docker-machine create -d hyperv --hyperv-virtual-switch "Primary Virtual Switch" --hyperv-disk-size "3000" --hyperv-memory "1024" --swarm --swarm-discovery consul://192.168.3.131/swarm --engine-registry-mirror "https://k4qqsfsl.mirror.aliyuncs.com" worker2

PS C:\WINDOWS\system32> docker-machine ssh master1
docker@master1:~$ docker swarm init --advertise-addr 192.168.3.132
  To add a worker to this swarm, run the following command:
    docker swarm join --token SWMTKN-1-1lo5eozjzzci74e8ln2hlsgec0mhue265hatvddyrb2cls1hcv-2y2r7rpj05bgcn6zcr7mnw9aq 192.168.3.132:2377
  To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
docker@master1:~$ docker run --restart=always -d -v /data:/data -p 8300:8300 -p 8301:8301 -p 8301:8301/udp -p 8302:8302 -p 8302:8302/udp -p 8400:8400 -p 8500:8500 -p 8600:8600 consul agent -server -ui -client 0.0.0.0 -advertise 192.168.3.132 -node=master1 -join 192.168.3.131
 
PS C:\WINDOWS\system32> docker-machine ssh worker1
docker@worker1:~$ docker swarm join --token SWMTKN-1-4zq9qr1zi9ip15jqtwp5k6quroupjmbp9c4ukymdipgjup24re-56ehxowuw1wd046bcodijg3ew 192.168.1.132:2377

PS C:\WINDOWS\system32> docker-machine ssh worker2
docker@worker2:~$ docker swarm join --token SWMTKN-1-4zq9qr1zi9ip15jqtwp5k6quroupjmbp9c4ukymdipgjup24re-56ehxowuw1wd046bcodijg3ew 192.168.1.132:2377

PS C:\WINDOWS\system32> docker-machine env master1
Error checking TLS connection: Error checking and/or regenerating the certs: There was an error validating certificates
for host "192.168.3.132:2376": x509: certificate is valid for 192.168.3.100, not 192.168.3.132
You can attempt to regenerate them using 'docker-machine regenerate-certs [name]'.
Be advised that this will trigger a Docker daemon restart which might stop running containers.
PS C:\WINDOWS\system32> docker-machine regenerate-certs master1 -f

# how to build

build without generate docker image:  
$ mvn install

build with generate docker image:  
$ mvn clean package -D maven.test.skip=true -P docker

# how to run

$ ~~mvn spring-boot:run~~

# port

** here is hosts profile **

' 192.168.x.x is your local network ip  
192.168.x.x consul.third.docker.pkrss.com  
192.168.x.x consul.microsrv.docker.pkrss.com  
192.168.x.x config.microsrv.docker.pkrss.com  
192.168.x.x discovery.microsrv.docker.pkrss.com  
192.168.x.x gateway.microsrv.docker.pkrss.com  


** here is port table **

| module     | port      | host     | description                        |
|:-----------|:--------- |:-------- |:---------------------------------- |
| consul     | tcp: 8300 8400 8500 8600\n udp:8301 8302 | consul.third.docker.pkrss.com |
| pkrss_microsrv_config | 18001 | config.microsrv.docker.pkrss.com | spring cloud config server |
| pkrss_microsrv_discovery | 18002 | discovery.microsrv.docker.pkrss.com  | spring cloud service discovery |
| pkrss_microsrv_consul | 18003 | consul.microsrv.docker.pkrss.com | spring cloud ??? (dns and profile?, i not sure) |
| pkrss_microsrv_gateway | 18004 | gateway.microsrv.docker.pkrss.com | spring api gateway |
| pkrss_microsrv_hystrix_dashboard | 18005 | hystrix_dashboard.microsrv.docker.pkrss.com | spring boot hystrix background monitor server |
| a.microsrv.docker.pkrss.com | 19001 | AServer | user demo 1 |
| b.microsrv.docker.pkrss.com | 19002 | BServer | user demo 2 |

* develop

1. how to change spring boot cloud config password in this sample project:  
   see it in: pkrss-microsrv-config\src\main\resources\bootstrap.yml
   
2. some urls  	
	http://<eurekaserver>/eureka/apps
	
other drafts:  
Docker 1.12 Swarm Mode集群实战(第三章): https://sanwen8.cn/p/34e2EJX.html
Docker 实战（五）：Docker Swarm Mode: http://www.tuicool.com/articles/nUbIn2z
基于docker1.12创建swarm集群: https://yq.aliyun.com/articles/58886
从零开始部署基于阿里容器云的微服务（consul+registrator+template）(一): http://alice.blog.51cto.com/707092/1896078
在阿里云容器服务上开发基于Docker的Spring Cloud微服务应用: https://yq.aliyun.com/articles/57265

# run in aliyun 

wait for me continue...

# other docker command

(if is swarm mode, need manager)

list all node:  
$ docker node ls

list all container:  
$ docker ps

in docker container run command:  
$ docker exec -it mynginx /bin/sh /root/runoob.sh  
$ docker exec -it $(docker ps | grep pkrss-microsrv-b | awk {'print $1'}) ping consul.local  

# other draft

S C:\WINDOWS\system32> docker-machine.exe env worker2
$Env:DOCKER_TLS_VERIFY = "1"
$Env:DOCKER_HOST = "tcp://188.199.1.68:2376"
$Env:DOCKER_CERT_PATH = "C:\Users\liand\.docker\machine\machines\worker2"
$Env:DOCKER_MACHINE_NAME = "worker2"
$Env:COMPOSE_CONVERT_WINDOWS_PATHS = "true"
 # Run this command to configure your shell:
 # & "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env worker2 | Invoke-Expression
PS C:\WINDOWS\system32> & "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env worker2 | Invoke-Expression
 # i skip: PS C:\WINDOWS\system32> docker run busybox echo hello world
   time="2017-06-27T09:25:17+08:00" level=info msg="Unable to use system certificate pool: crypto/x509: system root pool is
   not available on Windows"
   Unable to find image 'busybox:latest' locally
   latest: Pulling from library/busybox
   27144aa8f1b9: Pull complete
   Digest: sha256:be3c11fdba7cfe299214e46edc642e09514dbb9bbefcd0d3836c05a1e0cd0642
   Status: Downloaded newer image for busybox:latest
   hello world
PS C:\WINDOWS\system32> docker-machine ip worker2
188.199.1.68

default ssh connect 188.199.1.77:22 ssh username:docker password:tcuser

PS C:\WINDOWS\system32> & "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env manager1 | Invoke-Expression
Swarm initialized: current node (ooc0gi7luuskcvhch724sc4lu) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-3v50tuig1x58i9ahrbyycjdodud1w32npy64sf2xfasxregmcv-cbdo22fqfxo3lb8ak5lcqgnfs \
    188.199.1.77:2377

PS C:\WINDOWS\system32> & "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env worker1 | Invoke-Expression
PS C:\WINDOWS\system32> docker swarm join --token SWMTKN-1-3v50tuig1x58i9ahrbyycjdodud1w32npy64sf2xfasxregmcv-17wne27lgf99f3gg0mfhqhkmi 188.199.1.77:2377

PS C:\WINDOWS\system32> & "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env worker2 | Invoke-Expression
PS C:\WINDOWS\system32> docker swarm join --token SWMTKN-1-3v50tuig1x58i9ahrbyycjdodud1w32npy64sf2xfasxregmcv-17wne27lgf99f3gg0mfhqhkmi 188.199.1.77:2377

for china add aliyun accelerate: other man can skip it:
PS C:\WINDOWS\system32> docker-machine ssh manager1
docker@manager1:~$ docker run swarm create
docker@manager1:~$ exit
PS C:\WINDOWS\system32> docker-machine manager1
PS C:\WINDOWS\system32> docker-machine ssh worker1
docker@manager1:~$ sudo sed -i "s|EXTRA_ARGS='|EXTRA_ARGS='--registry-mirror=https://xxxxxxxx.mirror.aliyuncs.com |g" /var/lib/boot2docker/profile
docker@manager1:~$ exit
PS C:\WINDOWS\system32> docker-machine worker1
PS C:\WINDOWS\system32> docker-machine ssh worker2
docker@manager1:~$ sudo sed -i "s|EXTRA_ARGS='|EXTRA_ARGS='--registry-mirror=https://xxxxxxxx.mirror.aliyuncs.com |g" /var/lib/boot2docker/profile
docker@manager1:~$ exit
PS C:\WINDOWS\system32> docker-machine worker2
 
PS C:\WINDOWS\system32> & "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env manager1 | Invoke-Expression
PS C:\WINDOWS\system32> docker run consul
