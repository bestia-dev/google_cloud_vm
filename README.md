<!-- markdownlint-disable MD041 -->
[//]: # (auto_md_to_doc_comments segment start A)

# google_cloud_vm

***Configuration of my Google VM***

 ![maintained](https://img.shields.io/badge/maintained-green)
 ![ready_for_use](https://img.shields.io/badge/ready_for_use-green)
 [![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/bestia-dev/dropbox_backup_to_external_disk/blob/main/LICENSE)
 ![google_cloud_vm](https://bestia.dev/webpage_hit_counter/get_svg_image/1598265305.svg)

Google is very kind to give me free resources on the cloud for 12 months.  
I tried Microsoft Azure, but it was unclear to me what resources are 30 days and what 12 months free.  
*Things are changing fast. This is the situation on 2020-04-30.*

## keep the secret secret

Don't ever write any passwords and keys in any readme or code ever.  

## always free google VM

Now my 12 months trial is expired. I have still the "always free".  
I had to move my one "mini" vm-snapshot from Singapore to Oregon: us-west1-b for a "e2-micro" "always free".  
e2-micro (2 vCPU, 1 GB memory, Intel Broadwell), 30 GB-months HDD, 5 GB-month snapshot storage  
Google Cloud Free Tier is also available for external IP addresses that are being used by VM instances.  
It will be enough for my developer projects. But not for anything in production.  
I am afraid of the possibility to have an enormous bill. So I went to Billing - Budgets and alerts and set a
10 eur alert.  

## NOT FREE egress traffic

Egress traffic is not free. It can accumulate a lot of cost if a DDOS attack want to download a lot of data.  
The Billing budgets and alert don't really help. The interval to take Egress traffic under control is probably too slow, sometimes 24 hours. Then it just sends an email alert that can be missed. Later is already too late because the DDOS attack has already made your bill enormous. We are talking easy 3000$ per month for a VM that is basically free. Crazy. Till last month I had on average to pay 0,71$/month for Egress traffic, but last month was the eye opening 3$/month. After researching I found out, that I am just lucky that it is not 3000$. Google will not limit my traffic in any way or form because they make money if I have crazy traffic. But this could never be organic traffic for a personal hobby website. That could be only a DDOS attack, or now we can call it Denial of Wallet, because they could empty my already empty wallet. Bad Google made no effort to cap or limit my spending in any way. I am afraid of surprise enormous bills for basically nothing.
How to solve this problem? I don't know. Maybe I could try to limit traffic on NGINX ?!? To keep the data under 30GB/month that is around 5$/month. My maximum acceptable cost for DDOS attacks.

## google cloud Debian VM

This was an extreme adventure for a windows guy.  
It is a lot to learn and nothing is visual or intuitive.  
So I need to write my own manual with only commands I really need.  
First I created a cheap VM. They call it Compute Engine.  

- external ip:  35.199.190.85  (set to static in google console)  
- internal ip.  10.148.0.2  

I don't want to pay for a domain name, so I use the IP numbers.  
There is no free "url name" from google cloud platform.  

The VM's in cloud engine don't come with a root password setup by default.  
Passwords aren't configured for local users on Linux VMs.  
By default, Compute Engine uses project metadata to configure SSH keys and to manage SSH access.

Generate SSH key on every "client "computer, that is used as the connection initiator.  
Never move that private key from there. The destination computer does not matter, you copy there only the public key.  
If I have a "mac" I use the name Luciano_mac:  
`ssh-keygen -t rsa -f ~/.ssh/luciano_mac -C luciano_bestia`  
in the last comment (-C) is the username. This is a google or Debian convention.  
GitHub wants the email address for example.  
-- connect from WSL  
`ssh -i ~/.ssh/luciano_googlecloud luciano_bestia@bestia.dev -v`  

## Copying files

TotalCmd has a plugin for SFTP that includes also SSH file transfer. This is great.  
Created a project `c:\Users\Luciano\rustprojects\googlecloud\` to prepare and edit files locally.  
Then I use TotalCmd to copy them to the VM.  
It includes `/etc/nginx/sites-available/` and `/var/www/`.  
-- give me write permission on subfolders and file  
`sudo setfacl -d -R -m u:luciano_bestia:7 /var/www`  
`sudo setfacl -m u:luciano_bestia:7 /etc/nginx/sites-available/default`
Another way to copy files over ssh:  
`rsync -e ssh -avz --delete-after /home/luciano/rustprojects/googlecloud/var/www/webapps/mem6_game/`

For backup of the entire /var/www folder run on server:

```bash
pg_dump -F t -U admin -h localhost -p 5432 webpage_hit_counter > postgres_backup/webpage_hit_counter_backup_2024_07_23.tar
tar --exclude='bestia.dev/guitaraoke/videos' -czvf 2024_07_23_backup_of_var_www.tar.gz /var/www 
sudo tar -czvf 2024_07_23_backup_of_home_luciano_bestia.tar.gz /home/luciano_bestia 
```

Then use Totalcmd SSH to copy the backup file.  
GitHub does not allow files bigger than 100MB, so I compressed these backup files with Zip as multiple files of 99_000_000 bytes.

## Bash

Arrows up/down for command history is truly great.  
Ctrl+r for search history. Not very good.  

Linux file system is very case sensitive. Be careful if you come from windows. The easiest way is to have all file/folder names always and only in small caps.  
~ is the special sign for "user folder"  
The Linux application mc is a great file manager  
`sudo apt install mc`  
-- Remove directory not empty:  
`rm -rf mem2`  
Create a symlink in WSL to win rustprojects folder  
`ln -s /mnt/c/Users/Luciano/rustprojects ~/rustprojects`  
Cargo build inside WSL bash for Linux target. There will not be any windows servers any more in the future.  

## NGINX

Terrible settings and manuals for nginx. I hope I will use rust-warp in the future for reverse proxy.  
At the end of this readme is an example of the conf file.  
Some nginx tutorial:  
<https://phoenixnap.com/kb/nginx-reverse-proxy>  
-- install  
`sudo apt install nginx`  
-- edit main conf  
`sudo nano /etc/nginx/nginx.conf`  
-- edit other conf about websites  
`sudo nano /etc/nginx/sites-enabled/default`  
-- test conf file  
`sudo nginx -c /etc/nginx/nginx.conf -t`  
-- reload conf file  
`sudo nginx -s reload`  
-- if nothing else works  
`sudo reboot`  
-- read error log  
`sudo tail -30 /var/log/nginx/error.log`  
-- quick test if http server forwarding works  
`curl bestia.dev -v`  
`curl bestia.dev/mem4 -v`  
-- quick test if mem4_server works  
`curl 127.0.0.1:8084 -v`  
`curl 127.0.0.1:8084/content/alphabet/text.json -v`  

sudo service nginx start
sudo service nginx stop
sudo service nginx restart

### NGINX SSL

SSL mostly does not work on IP address. It requires a domain name.  
Google cloud does not give me a domain name, just an IP.  
So I have to buy the domain name `bestia.dev` at porkbun.com for 11$ per year.  
The `.dev` TLD (top level domain) is a `forced https:` (but that's something only browsers care about).  
In the porkbun.com I edited the DNS: added one A record (address) with the IP 35.199.190.85.  
In <https://console.cloud.google.com> check Firewalls - Allow HTTPS traffic.  

<https://linux4one.com/how-to-secure-nginx-with-lets-encrypt-ssl-on-debian-10/>  
`sudo certbot --nginx -d bestia.dev -d bestia.dev`  
The certbot writes in the folder: `/etc/letsencrypt/`  
The certbot changes the config file: `/etc/nginx/sites-enabled/default`  
Check if the site works:  
`curl -k https://bestia.dev`  
<https://www.ssllabs.com/ssltest/analyze.html?d=bestia.dev>  
List the certificates with expiration date:  
`sudo certbot certificates`  
To try if there is need for renewal:  
`sudo certbot renew --dry-run`  
Use crontab with sudo to schedule a renew attempt every 12 hours:  
`sudo crontab -e`  
choose nano text editor. Add this line:  
`0 */12 * * * root test -x /usr/bin/certbot -a \! -d /run/systemd/system && perl -e 'sleep int(rand(43200))' && certbot -q renew`  
If the renew is not needed nothing happens. The -q stands for quiet.  
To see the crontab log:  
`grep CRON /var/log/syslog`  

## snapshot

Make a VM snapshot in GoogleCloud console. So in the case of a catastrophe, it is easy to return back to a working situation.  

## Git and ssh

--use of ssh key for git with ssh agent  
`ssh-agent bash -c 'ssh-add ~/.ssh/luciano_mac; git clone git@github.com:bestia-dev/mem4_game.git'`  

## mem4_game

Basically is the same for every webapp.  
Prepare the files in  
`c:\Users\Luciano\rustprojects\googlecloud\var\www\webapps\mem4_game\`  
and copy them with TotalCmd.  
-- command in SSH bash  
`cd /var/www/webapps/mem4_game`  
-- make the file executable (only once)  
`chmod +x mem4_server`  
-- to start the application in a new tab of Zellij
name it mem4-8084
-- start the game server  
`sudo ./mem4_server 127.0.0.1 8084`  
-- detach background process  
session, detach
--the next time attach to this process with  
`zellij attach`  

## Zellij command to execute after VM reboot

Nginx is automatically started and so are static web-file-server pages:  
bestia.dev/index.html, /mem1, /amafatt,...

Postgres is also automatically started, I guess.  
For the webapps I run them in separate sessions with Zellij multiplexer.

-- open new tabs for every application and run the commands  

-- game mem6  
`cd /var/www/webapps/mem6_game ; sudo ./mem6_server 127.0.0.1 8086`  
-- cargo_crev_web web server  
`cd /var/www/webapps/cargo_crev_web ; ./cargo_crev_web`  
-- cargo_crev_web git fetch repo  
-- prepare credentials if needed (not for fetch)  
`eval $(ssh-agent -s) ; ssh-add ~/.ssh/web_crev_dev_for_github`  
-- schedule for every hour at xx:04 minutes  
`foreground_scheduler 4 cargo-crev "crev repo fetch trusted"`  
-- run webpage_hit_counter
`cd bin/hit_counter_and_env; webpage_hit_counter`

## Containers and Podman

I don't use Docker anymore for Linux containers, but instead I use Podman. It has the same functionality. Same same, but different.  
It is important to understand the difference between images and containers.  
The images are passive files you can copy around. They are like installation files. The containers are active running applications of these images.  
First you have to build your own image. Usually an image is based on somebody others prepared image that you find on DockerHub.  
To build an image I use Buildah, the companion of Podman. I never liked to use the old school docker-files. With Buildah you can run instructions step by step to create an image. Then this steps can be written in a simple bash script. Total control.  

## IPv6

IPv6 is the future. But it is really hard to activate this in google platform cloud.  
Internally they have only IPv4 ?! For an external IPv6 address you need to create a load balancer and instance group. Too much work.  

## nginx conf files

edit conf:  
`sudo nano /etc/nginx/nginx.conf`  
add:  

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''  close;
}
```

edit conf:  
`sudo nano /etc/nginx/sites-enabled/default`  
looks like this:  

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;

    # Add index.php to the list if you are using PHP
    index index.html index.htm index.nginx-debian.html;

    server_name _;

    #region mem4
    #without the trailing / it is not a directory (for the server and for the browser)
    location = /mem4 {
        return 301 /mem4/;
    }
    #for index.html a special location
    location = /mem4/ {
        proxy_pass http://127.0.0.1:8084/index.html;
    }
    #websocket special header - Upgrade rules
    location /mem4/mem4ws/ {
        proxy_pass http://127.0.0.1:8084/mem4ws/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }

    # the trailing / after both of these lines means this route is not appended to the forwarding
    location /mem4/ {
        proxy_pass http://127.0.0.1:8084/;
    }

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.  
        try_files $uri $uri/ =404;
    }
}
```

## Upgrade to Debian 11 bullseye

<https://www.cyberciti.biz/faq/update-upgrade-debian-10-to-debian-11-bullseye/>

The procedure is as follows:

1.Backup the system. GoogleCloud snapshot.  
2.Update existing packages and reboot the Debian 10 system.  
3.Edit the file /etc/apt/sources.list using a text editor and replace each instance of buster with bullseye.  
  Next find the security line, replace keyword   buster/updates with bullseye-security.  
4.Update the packages index on Debian Linux, run:
sudo apt update
5.Prepare for the operating system minimal system upgrade, run:
sudo apt upgrade --without-new-pkgs
6.Finally, update Debian 10 to Debian 11 bullseye by running:
sudo apt full-upgrade
7.Reboot the Linux system so that you can boot into Debian 11 Bullseye
8.Verify that everything is working correctly.

```bash
cat /etc/debian_version
    10.12
cat /etc/issue
    Debian GNU/Linux 10 \n \l
sudo apt update 
sudo apt dist-upgrade 
sudo reboot


In all this files here change the word buster to bullseye:
sudo nano /etc/apt/sources.list 

sudo apt update
sudo apt upgrade
sudo apt full-upgrade
sudo apt autoremove
sudo apt clean

sudo reboot
cat /etc/issue
    Debian GNU/Linux 11 \n \l
cat /etc/debian_version
    11.3
```

## upgrade Debian 11 to 12

<https://www.cyberciti.biz/faq/update-upgrade-debian-11-to-debian-12-bookworm/>

1. backup - I did a Snapshot to Archive of Google Compute Engine
2. Update existing packages `sudo apt update` and `sudo apt upgrade` and `sudo reboot` the Debian 11 system.
3. Edit the file `sudo nano /etc/apt/sources.list`
and replace each instance of `bullseye` with `bookworm`.
4. Update the packages index on  Debian Linux Linux, run: `sudo apt update`
5. Prepare for the operating system minimal system upgrade, run: `sudo apt upgrade --without-new-pkgs`
6. Finally, update  Debian 11 to Debian 12 Bookworm by running: `sudo apt full-upgrade`  
WARNING: On question if you want to change some conf files always choose the `default`: `not change the old one`. The new conf file will be stored as `pkg-new`  
Later you can compare the old and new files and modify it accordingly.
7. Reboot the Linux system `sudo reboot` so that you can boot into Debian 12 Bookworm
8. Verify that everything is working correctly `lsb_release -a`, `cat /etc/debian_version`, `uname -mrs`
   `sudo systemctl status nginx.service`
9. Free space `sudo apt --purge autoremove`
10. Run podman pod for hit counter, Zellij for mem6, web.crev.dev and crev git

## SSH from Debian as a directory mount

```bash
sshfs lucianobestia@bestia.dev:/ /mnt/bestia_dev_ssh -o reconnect
```

## disk full, I need to free space

It happens to everybody. The disk is so small, that it is easy to fill it.  
Nothing can work with the disk full. Neither the certbot for LetsEncrypt. Bad !  
How much space do I have on my limited small VM disk 10 GB?

```bash
df -h
   /dev/sda1       9.8G  9.8G  0.0G  100% /
```

Clean some space:

```bash
sudo apt-get clean
```

Now I have a little more space:

```bash
df -h
   /dev/sda1       9.8G  7.6G  1.7G  83% /
```

Find the biggest files and directories

```bash
sudo du -a / | sort -n -r | head -n 20
```

## Remove Podman and pods for counter_hit and Postgres

Podman pods for `counter_hit` and `Postgres 13` occupy too much space on my micro-limited disk 10 GB. Installing them normally on the Debian OS takes less space.  
Remove pods, containers, images and podman. Be careful that the database data are in an external volume on the disk.

```bash
podman pod list
# remove pods
podman pod rm pod_id

podman ps
# remove containers
podman rm container_id

podman images
# remove images
podman rmi image_id

# Complete Uninstall
sudo apt --purge remove buildah skopeo podman docker

sudo ls /etc/containers
sudo rm -rf /etc/containers

sudo ls /var/lib/containers 
sudo rm -rf /var/lib/containers 

# Remember to delete containers personal to users:
ls /home/luciano_bestia/.local/share/containers
rm -rf /home/luciano_bestia/.local/share/containers
```

## Uninstall Postgres

If needed uninstall old Postgres. Be careful to not delete your data directory. My data directory is `/home/luciano_bestia/postgres_data`.

```bash
dpkg -l | grep postgres

# result:
# ii  postgresql-client                     15+248                                  all          front-end programs for PostgreSQL (supported version)
# ii  postgresql-client-13                  13.15-1.pgdg120+1                       amd64        front-end programs for PostgreSQL 13
# ii  postgresql-client-15                  15.7-0+deb12u1                          amd64        front-end programs for PostgreSQL 15
# ii  postgresql-client-common
# remove all the above

sudo apt-get --purge remove postgresql-client postgresql-client-13 postgresql-client-15 postgresql-client-common

sudo rm -rf /var/lib/postgresql/
sudo rm -rf /var/log/postgresql/
sudo rm -rf /etc/postgresql/
```

## Install Postgres 13 on Debian 12

For now, I will use the same version of Postgres 13 as before.  
The installation is awkward:  
<https://computingforgeeks.com/install-postgresql-on-debian-linux/>

```bash

sudo apt update && sudo apt -y upgrade
sudo apt -y install gnupg2
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg

echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
cat /etc/apt/sources.list.d/pgdg.list
sudo apt update

sudo apt -y install postgresql-13

# change data_directory
sudo /etc/init.d/postgresql status
sudo /etc/init.d/postgresql stop
sudo nano /etc/postgresql/13/main/postgresql.conf

# (change requires restart)
# from:
data_directory = '/var/lib/postgresql/13/main'
# to:
data_directory = '/home/luciano_bestia/postgres_data/webpage_hit_counter_pod'

# start the executable for debugging before starting as a service, because there are no logs
# first enter terminal as the user postgres
sudo su  - postgres
# then start postgres manually with debugging level 3 enabled:
/usr/lib/postgresql/13/bin/postgres -d 3 -c config_file=/etc/postgresql/13/main/postgresql.conf
# error:  data directory "/home/luciano_bestia/postgres_data/webpage_hit_counter_pod" has wrong ownership

# correct the ownership to the user postgres
sudo chown -Rf postgres:postgres /home/luciano_bestia/postgres_data
sudo chmod -R 700 /home/luciano_bestia/postgres_data

# retry 
/usr/lib/postgresql/13/bin/postgres -d 3 -c config_file=/etc/postgresql/13/main/postgresql.conf
# it should work.


https://openbasesystems.com/2023/06/20/postgresql-error-fatal-role-username-does-not-exist/

from my old installation:

-e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=password \

id admin
# not exist
useradd admin

ALTER ROLE role_name WITH attribute_options;
ALTER DATABASE postgres OWNER TO postgres;


sudo /etc/init.d/postgresql start
# enter a terminal session as the database user postgres
sudo su  - postgres
psql
SHOW data_directory;
   /var/lib/postgresql/15/main
\q
exit

```

The default folders for Postgres 13 data is: ``
I already have data in the folder `~/postgres_data` and the backup in `postgres_backup`.  
How to attach this folder to postgres13 ?
<https://www.squash.io/exploring-postgresql-check-db-dir-in-postgresql-databases/>
Yes, you can change the default  database directory in  PostgreSQL by modifying the data_directory configuration parameter in the  postgresql.conf file. By default, the data directory is set to /var/lib/postgresql//main on Linux systems.
sudo find / -name postgresql.conf

/etc/postgresql/13/main/postgresql.conf
/home/luciano_bestia/postgres_backup/before_upgrade/webpage_hit_counter_pod/postgresql.conf
/home/luciano_bestia/postgres_data/webpage_hit_counter_pod/postgresql.conf

the default data directory is /var/lib/postgresql/13/main
the directory /etc/postgresql/13/main is for internal use by Postgres

The default values of these variables are driven from the -D command-line
option or PGDATA environment variable, represented here as ConfigDir.

## Open-source and free as a beer

My open-source projects are free as a beer (MIT license).  
I just love programming.  
But I need also to drink. If you find my projects and tutorials helpful, please buy me a beer by donating to my [PayPal](https://paypal.me/LucianoBestia).  
You know the price of a beer in your local bar ;-)  
So I can drink a free beer for your health :-)  
[Na zdravje!](https://translate.google.com/?hl=en&sl=sl&tl=en&text=Na%20zdravje&op=translate) [Alla salute!](https://dictionary.cambridge.org/dictionary/italian-english/alla-salute) [Prost!](https://dictionary.cambridge.org/dictionary/german-english/prost) [Nazdravlje!](https://matadornetwork.com/nights/how-to-say-cheers-in-50-languages/) 🍻

[//bestia.dev](https://bestia.dev)  
[//github.com/bestia-dev](https://github.com/bestia-dev)  
[//bestiadev.substack.com](https://bestiadev.substack.com)  
[//youtube.com/@bestia-dev-tutorials](https://youtube.com/@bestia-dev-tutorials)  

[//]: # (auto_md_to_doc_comments segment end A)
