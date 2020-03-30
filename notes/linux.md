# Linux

### Hotkeys
* **ctrl-c** kills a command
* **ctrl-d** closes a window
* **ctrl-a** skips to beginning of line
* **ctrl-e** skips to end of line
* **ctrl-w** yanks previous word
* **ctrl-u** yanks previous lines
* **ctrl-y** pastes yanked code

### Process
Search for and kill process
``` bash
ps aux | grep PROCESS
kill PID
```

### Memory
* Show filesystem storage
``` bash
df -h
```
* Show directory size
``` bash
du -sh DIRECTORY
```
* Show RAM usage
``` bash
free -h
```

### Compressing
* Zip and tar the source directory to the specified path
``` bash
tar -czvf DEST_PATH SOURCE_PATH
```
* Untar and unzip from the specified path
``` bash
tar -xzvf SOURCE_PATH
```

### Format Drive
* Show drive info (locate appropriate device **sdX1**)
``` bash
sudo fdisk -l
```
* Unmount the file system for the drive
``` bash
sudo umount /dev/sdX1
```
* Zero out a drive
``` bash
sudo dd if=/dev/zero of=/dev/sdX1 bs=4k conv=fdatasync
```
* write the disk image
``` bash
sudo dd if=os.iso of=/dev/sdX1 bs=4k conv=fdatasync
```
* Mount the given drive, optionally specifying the filesystem
``` bash
sudo mount -t vfat /dev/sdX1 /mnt/DIR
```
http://www.lostsaloon.com/technology/how-to-format-usb-drives-flash-drive-from-linux-command-line/

### Grow Volume
* Install growpart
``` bash
sudo apt-get install cloud-guest-utils
```
* extend the first partition in sdX to fill all available space
``` bash
growpart /dev/sdX1 1
```
* Resize ext file systems (enlargen an unmounted file system located on device sdX)
``` bash
resize2fs /dev/sdX
```

### SSH
* Copy ssh public key to remote server
``` bash
ssh-copy-id -i ~/.ssh/mykey USER@HOST
```
* Setup background ssh tunnel for a MySQL database connection
``` bash
ssh -i ~/.ssh/KEY -L 3310:localhost:3306 MYSQL_IP_ADDRESS -Nf
```

### Grep
Search directory for a pattern in certain types of files
``` bash
grep -rn --include=\*.{sql,py,ipynb} "PATTERN" DIR
```

### Hexdump
Show bytes with interpreted characters, limited to 8 bytes with a starting offset of 0 bytes (repeated lines shown as *)
``` bash
hexdump -C -n 8 -s 0 FILE
```

### Cron
Schedule jobs to run periodically
``` bash
crontab -e
```
Configure the cron file
``` bash
SHELL=/bin/bash
MAILTO=EMAIL
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

# practice command
50 * * * * echo "Hey!" >> ~/cron_temp.txt
```
https://crontab.guru/
https://tecadmin.net/crontab-in-linux-with-20-examples-of-cron-schedule/

### Curl
HTTP CLI client
* **GET**
  * ` curl http://www.google.com -o OUTPUT_FILE `
    * ` -i ` add response header
    * ` -I ` only response header
    * ` -v ` verbose logging
    * ` --location ` use to redirect
* **POST**
  * ` curl --data-urlencode KEY=VALUE http://httpbin.org/post `
    * ` -urlencode ` necessary when data is not encoded
* **PUT**
  * ` curl --upload-file UPLOAD_FILE http://httpbin.org/put `

https://curl.haxx.se/docs/httpscripting.html

### Screen
Process CLI session manager
``` bash
screen -S NAME
# ctrl-a, ctrl-d - leaves the session without terminating it
screen -ls
screen -r NAME
```

### Crypto
* Create new ssh key
  ``` bash
  ssh-keygen -P "" -f ~/.ssh/FILE.pem
  chmod 600 ~/.ssh/FILE.pem
  ```
* Generate public key from private key
  ``` bash
  ssh-keygen -y -f ~/.ssh/FILE.pem > ~/.ssh/FILE.pem.pub
  ```

### SED
* Outputs the (LINE_NUMBER)th line from FILENAME
``` bash
sed -n LINE_NUMBERp FILENAME
```
* Extracts lines from START to END from INPUT_FILE to OUTPUT_FILE, where ENDD is END+1
``` bash
sed -n 'START,ENDp;ENDDq' INPUT_FILE > OUTPUT_FILE
```
* Outputs the first NUMBER of characters from FILENAME
``` bash
cut -c-NUMBER FILENAME
```

### Misc
* Check user info
``` bash
whoami
cat /etc/passwd
```
* shows the location of the specified command
``` bash
which COMMAND
```
* Compute checksums
``` bash
sha256sum FILE
echo -n MESSAGE | md5sum
```
* Make a file executable
``` bash
chmod +x FILE
```
* Redirect **stderr** to **stdout**
``` bash
2>&1
```
* Turn off history substitution (e.g. allowing unescaped passwords with "!")
``` bash
set +H
```
* Run multiple line command
``` bash
cat <<EOF
$(ls /)
$(ls /home)
EOF
```

### Digital Ocean Droplet Setup
1. Receive email from Digital Ocean for droplet DROPLET_NAME with DROPLET_IP_ADDRESS and ROOT_PASSWORD
1. Connect to droplet
``` bash
ssh root@DROPLET_IP_ADDRESS
```
1. Create a new user, give usergroup root privileges, and set user password
``` bash
adduser USER
usermod -aG sudo USER
sudo passwd USER
```
1. Generate ssh rsa key pair
``` bash
ssh-keygen -f ~/.ssh/KEY -t rsa
```
1. Add the following (local) entry to **~/.ssh/config**:
```
Host DROPLET_NAME
  Hostname DROPLET_IP_ADDRESS
  User USER
  IdentityFile ~/.ssh/KEY
```
* (Remember to configure the firewall through "Networking/Firewalls" tab on DigitalOcean's web portal)
* Reference: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04

### HTTPS Certification
1. Add the certbot repo, update and install
``` bash
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
```
1. Add https certification through nginx for the specified URL
``` bash
sudo certbot --nginx -d URL
```
1. Renew certification
``` bash
sudo certbot renew
```
* Remember to add HTTPS firewall rule (port 443) to digital ocean firewall
* Reference: https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04

### Iterate Command Over CSV File
``` bash
#!/bin/bash

COMMAND=$1
INPUT_PATH=$2
LOG_FILE=$3

rm -f $LOG_FILE

while IFS=, read -a csv_line
do
  $COMMAND ${csv_line[0]} >> $LOG_FILE 2>&1
done < $INPUT_PATH
```

### Notes
* Linux Mint was having RAM overflow freezing, which can be remedied by setting the "swappiness" to 0 by adding the following to */etc/sysctl.conf*: "vm.swappiness=0"
