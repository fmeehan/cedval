# Installation steps for Ubuntu Server 18.04 on Intel Nuc

## Boot from usb key containing ISO image

Configure Ethernet manually

see image:

![IP static config](/images/Set_IP_at_install.JPG)


## Fix timezone

```
sudo timedatectl set-timezone America/Toronto
```

## Update Repository list

```
vi /etc/apt/sources.list
```
Content should look like this:
```
deb http://archive.ubuntu.com/ubuntu bionic universe main
deb http://archive.ubuntu.com/ubuntu bionic-security main
deb http://archive.ubuntu.com/ubuntu bionic-updates main
deb-src http://archive.ubuntu.com/ubuntu bionic universe
deb-src http://archive.ubuntu.com/ubuntu bionic multiverse
deb http://cz.archive.ubuntu.com/ubuntu bionic main
```

Then run:
```
sudo add-apt-repository "deb http://archive.ubuntu.com/ubuntu $(lsb_release -sc) universe"
```

# Refresh repositories and update apps

```
sudo apt update
sudo apt upgrade
```

## Switch to zsh

```
sudo apt install zsh
sudo apt-get install git-core
wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | zsh
```

Swtich shell
```
chsh -s `which zsh`
```
Logout and back in.

### zsh-autosuggestions an autocompletion

```
mkdir -p ~/.zsh/completion

git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-

curl -L https://raw.githubusercontent.com/docker/compose/1.23.1/contrib/completion/zsh/_docker-compose > ~/.zsh/completion/_docker-compose
```

Add the following lines to your .zshrc:

```
source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh
fpath=(~/.zsh/completion $fpath)
autoload -Uz compinit && compinit -i
```

Reload your shell

```
exec $SHELL -l
```

## Download latest CedVal repos

```
cd ~
git clone https://github.com/fmeehan/cedval.git
```



# Package Install
```
sudo apt install tmux cowsay mc git
```
## Additional stuff


### mdv markdown on zsh
https://github.com/axiros/terminal_markdown_viewer

## Postgresql

### Modify config file

Add following lines to sudo vi

 /etc/postgresql/11/main/pg_hba.conf

```
#Giving acces to Docker containers
host    all             all             172.17.0.1/16           md5

Change the following lines in /etc/postgresql/10/main/postgresql.conf

listen_addresses = '*'
ssl = off
```

### Create DBA user
```
sudo -i -u postgres
createuser  --interactive  # enter dba as user name, make it a super user

sudo adduser dba #to set password
```
### Test

```
sudo -i -u dba
psql -d postgres #if psql prompt appears, it’s all good
```


### To exit Psql
```
\q
```
### Install Postqres DB admin app  pgadmin

https://www.pgadmin.org/

```
docker pull dpage/pgadmin4

docker run -p 70:80 \
-e "PGADMIN_DEFAULT_EMAIL=fmeehan@cedvalinfo.com" \
-e "PGADMIN_DEFAULT_PASSWORD=nx9010" \
-d dpage/pgadmin4
```

## Docker for Linux Setup and Tips

### Docker CE install

Get the Docker install script by going to [docker](https://get.docker.com/)
Copy the code in the page and create installation file.

```
vi ~/install_docker.sh #Past your stuff

chmod a+x install_docker

./install_docker
```

### Docker-compose


Run this command to download the latest version of Docker Compose:


```
sudo curl -L "https://github.com/docker/compose/releases/download/latest/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose


sudo chmod +x /usr/local/bin/docker-compose
```
### Docker-Machine

Docker machine [github](http://github.com/docker/machine/releases)  


Run following command
```
base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```

Verify your versions

```
docker version

docker-compose version

docker-machine version
```

#### Give your users access to Docker stuff
```
sudo usermod -aG docker fmeehan
```

#### Command completion for zch

References: [Docker Compose](https://docs.docker.com/compose/completion/)

```
$ mkdir -p ~/.zsh/completion
$ curl -L https://raw.githubusercontent.com/docker/compose/1.23.1/contrib/completion/zsh/_docker-compose > ~/.zsh/completion/_docker-compose
```
Include the directory in your $fpath by adding in ~/.zshrc:

```
fpath=(~/.zsh/completion $fpath)
```

Make sure compinit is loaded or do it by adding in ~/.zshrc:

```
autoload -Uz compinit && compinit -i
```

Then reload your shell:
```
exec $SHELL -l
```

# DNS resolution dilemma

Not sure about this /run/systemd/resolve/resolv.conf

```
nameserver 192.168.0.105
nameserver 198.168.0.105
nameserver 8.8.4.4
search cedval.int
```
Must read [Nice post](https://unix.stackexchange.com/questions/304050/how-to-avoid-conflicts-between-dnsmasq-and-systemd-resolved)