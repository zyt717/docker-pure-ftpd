
Docker Pure-ftpd Server
============================

Pull down with docker:
```bash
docker pull zyt717/docker-pure-ftpd
```

**Often needing to run as `sudo`, e.g. `sudo docker pull zyt717/docker-pure-ftpd`**

----------------------------------------

**My advice is to extend this image to make any changes.**  
This is because rebuilding the entire docker image via a fork can be slow as it rebuilds the entire pure-ftpd package from source. 

Instead you can create a new project with a `DOCKERFILE` like so:

```
FROM zyt717/docker-pure-ftpd

# e.g. you could change the defult command run:
CMD /run.sh -c 30 -C 5 -l puredb:/etc/pure-ftpd/pureftpd.pdb -E -j -R 
```

*Then you can build your own image, `docker build --rm -t my-pure-ftp .`, where my-pure-ftp is the name you want to build as*

----------------------------------------

Starting it 
------------------------------

`docker run -d --name ftpd_server -p 21:21 -p 30000-30009:30000-30009 -e "PUBLICHOST=localhost"  -e "PASVPORT=30000:30009"  zyt717/docker-pure-ftpd`

*Or for your own image, replace zyt717/docker-pure-ftpd with the name you built it with, e.g. my-pure-ftp*

Operating it
------------------------------

`docker exec -it ftpd_server /bin/bash`

Example usage once inside
------------------------------

Create an ftp user: `e.g. bob with chroot access only to /home/ftpusers/bob`
```bash
pure-pw useradd bob -m -u ftpuser -d /home/ftpusers/bob
```
*No restart should be needed.*

More info on usage here: https://download.pureftpd.org/pure-ftpd/doc/README.Virtual-Users


Test your connection
-------------------------
From the host machine:
```bash
ftp -p localhost 21
```

----------------------------------------

Our default pure-ftpd options explained
----------------------------------------

```
/usr/sbin/pure-ftpd # path to pure-ftpd executable
-c 50 # --maxclientsnumber (no more than 50 people at once)
-C 10 # --maxclientsperip (no more than 10 requests from the same ip)
-l puredb:/etc/pure-ftpd/pureftpd.pdb # --login (login file for virtual users)
-E # --noanonymous (only real users)
-j # --createhomedir (auto create home directory if it doesnt already exist)
-R # --nochmod (prevent usage of the CHMOD command)
-P $PUBLICHOST # IP/Host setting for PASV support, passed in your the PUBLICHOST env var
-p 30000:30009 # PASV port range
```

For more information please see `man pure-ftpd`, or visit: https://www.pureftpd.org/

Why so many ports opened?
---------------------------
This is for PASV support, please see: [#5 PASV not fun :)](https://github.com/stilliard/docker-pure-ftpd/issues/5)

----------------------------------------

Keep user database in a volume
------------------------------
You may want to keep your user database through the successive image builds. It is possible with Docker volumes.

Create a named volume:
```
docker volume create --name my-db-volume
```

Specify it when running the container:
```
docker run -d --name ftpd_server -p 21:21 -p 30000-30009:30000-30009 -e "PUBLICHOST=localhost" -v my-db-volume:/etc/pure-ftpd/passwd zyt717/docker-pure-ftpd
```

When an user is added, you need to use the password file which is in the volume:
```
pure-pw useradd bob -f /etc/pure-ftpd/passwd/pureftpd.passwd -m -u ftpuser -d /home/ftpusers/bob
```
(Thanks to the -m option, you don't need to call *pure-pw mkdb* with this syntax).

----------------------------------------
Development (via git clone)
```bash
# Clone the repo
git clone https://github.com/stilliard/docker-pure-ftpd.git
cd docker-pure-ftpd
# Build the image
make build
# Run container in background:
make run
# enter a bash shell insdie the container:
make enter
```

Credits
-------------
Thanks for the help on stackoverflow with this!
https://stackoverflow.com/questions/23930167/installing-pure-ftpd-in-docker-debian-wheezy-error-421


Development sponsored by [ecommerce.co.uk](https://www.ecommerce.co.uk/?utm_source=docker-pureftpd&utm_medium=referral&utm_campaign=open-source)

