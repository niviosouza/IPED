# Tutorial de compilação do IPED

Siga os passos para preparação e compilação do IPED.

## Instalação do Sistema Operacional

A compilação foi executada em um Linux Mint 21.3, mas pode ser executada, quase sem diferenças, em qualquer distribuição derivada do Debian.

## Preparação do Sistema Operacional

### Escale para root

```
sudo su -
```

### Atualize o sistema

```
apt update
apt dist-upgrade -y
```

### Baixe a chave do software JFX indicado pela equipe do IPED

```
wget -c -t0 -T5 https://download.bell-sw.com/pki/GPG-KEY-bellsoft -O /etc/apt/trusted.gpg.d/GPG-KEY-bellsoft.asc
echo "deb [arch=amd64] https://apt.bell-sw.com/ stable main" > /etc/apt/sources.list.d/bellsoft.list
```

### Atualize a base de índices de pacotes

```
apt update
```

### Instale os pacotes necessários

```
apt install ant automake automake1.11 autopoint bellsoft-java11-full build-essential ewf-tools git graphviz imagemagick libafflib-dev libesedb-utils libevt-utils libevtx1 libevtx-dev libevtx-utils libewf-dev libjavafxsvg-java libmsiecf-utils libopenjfx-java libopenjfx-java-doc libopenjfx-jni libparse-win32registry-perl libpff1 libreoffice-gtk2 libreoffice-java-common libscca-utils libssl-dev libtool libvhdi-dev libvmdk-dev libvslvm-dev m4 make maven mplayer mplayer-gui openjdk-11-dbg openjdk-11-jdk openjdk-11-jre openjdk-11-source openjfx pkg-config python3.10-venv python3-evtx python3-libevtx python3-libewf python3-pip python3-vlc rifiuti2 smbclient tesseract-ocr tesseract-ocr-eng tesseract-ocr-por tesseract-ocr-script-latn vim vim-addon-manager vim-runtime vim-scripts vlc
```

**OBSERVAÇÃO: esta instalação vai demorar de 15 a 20 minutos.**

### Facilite o uso do vim

```
vim /etc/vim/vimrc
[...]
set number
set background=dark
[...]
```

## Instalação de outras dependências

### Instale um módulo do Python

```
pip install numpy
```

### Insira uma variável de ambiente no sistema

```
vim /etc/profile

[...]
#export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export JAVA_HOME=/usr/lib/jvm/bellsoft-java11-full-amd64
```

### Carregue a variável de ambiente

```
source /etc/profile
```

### Instale outro módulo do Python

```
pip install jep==4.0.3
```

### Insira atalho para a biblioteca

```
ln -svf /usr/local/lib/python3.10/dist-packages/jep/libjep.so /usr/lib/libjep.so
```

### Insira outra variável de ambiente no sistema 

```
vim /etc/profile
[...]
export LD_LIBRARY_PATH=/usr/local/lib/python3.10/dist-packages/jep/
```

### Carregue a variável de ambiente

```
source /etc/profile
```

### Reinicie o sistema operacional

```
reboot
```

## Compilação e instalação de outras dependências

### Escale para root

```
sudo su -
```

### Entre no diretório de códigos-fonte

```
cd /usr/src
```

### Clone o sleuthkit

```
git clone -b 4.12.0_iped_patch https://github.com/sepinf-inc/sleuthkit
```

### Entre no diretório do código-fonte do sleuthkit

```
cd sleuthkit/
```

### Prepare e configure o código-fonte do sleuthkit para compilação

```
./bootstrap
./configure --prefix=/usr
```

### O resultado final esperado

```
Building:
   openssl support:                       yes

   afflib support:                        yes
   libewf support:                        yes
   zlib support:                          yes

   libbfio support:                       yes
   libvhdi support:                       yes
   libvmdk support:                       yes
   libvslvm support:                      yes
Features:
   Java/JNI support:                      yes
   Multithreading:                        yes
```

### Compile e instale o sleuthkit

```
PROC=$(cat /proc/cpuinfo | grep processor | wc -l)
PROC=$((PROC+1))
make -j${PROC}
make install
```

### Entre no diretório de códigos-fonte

```
cd /usr/src/
```

### Clone a biblioteca libagdb

```
git clone https://github.com/libyal/libagdb.git
```

### Entre no diretório do código-fonte da biblioteca

```
cd libagdb/
```

### Prepare, compile e instale

```
./synclibs.sh
./autogen.sh
./configure --prefix=/usr --enable-wide-character-type
make -j${PROC}
make install 
ldconfig
```

## Compilação do IPED

### Reinicie o sistema operacional

```
reboot
```

### Escale para root

```
sudo su -
```

### Entre no diretório de códigos-fonte

```
cd /usr/src/
```

### Clone o IPED

```
git clone https://github.com/sepinf-inc/IPED.git
```

### Entre no diretório do IPED

```
cd IPED
```

### Compile o IPED

```
mvn clean install
```

**OBSERVAÇÃO: em um máquina virtual com 4 VCPUs e 16 GB RAM, a compilação demorou por volta de 45 minutos.**

## Passos pós-compilação

### Permita o acesso à interface gráfica ao usuário 'root' por meio dos comandos 'su' e 'sudo'

#### Insira a seguinte linha no fim dos arquivos

```
vim /etc/pam.d/su
[...]
session optional pam_xauth.so

vim /etc/pam.d/sudo
[...]
session optional pam_xauth.so
```

### Configuração básica do IPED

```
vim /usr/src/IPED/target/release/iped-4.2-snapshot/LocalConfig.txt
[...]
locale = pt-BR
[...]
tskJarPath = /usr/share/java/sleuthkit-4.12.0.jar
[...]
```

### Criação de diretórios do sistema

```
mkdir -p -m 0755 /var/lib/samba/usershares
```

### Crição de uma pasta para o caso

```
mkdir -p /home/<usuario>/caso1
```

### Criação de um HD virtual para testes

```
dd if=/dev/zero of=/home/<usuario>/caso1/teste.img bs=100M count=1
```

#### Verifique o arquivo de loop

```
losetup -a
/dev/loop0: [2050]:3015005 (/home/<usuario>/caso1/teste.img)
```

#### Formate a partição

```
mkfs.ext4 /dev/loop0
```

#### Monte a partição do HD virtual

```
mount -t ext4 /dev/loop0 /mnt
```

#### Verifique a montagem

```
mount | grep loop0
/dev/loop0 on /mnt type ext4 (rw,relatime)
```

#### Copie conteúdos para o ponto de montagem /mnt para simular um HD qualquer

```
cp -a <origem> /mnt/
```

#### Desmonte

```
umount /mnt
```

#### Remova o loop

```
losetup -d /dev/loop0 
```

#### Saia do root

```
exit
```

## Resumo

O processo de preparação, compilação e pós-compilação levou em torno de 90 minutos!

## Execução do IPED para indexação

```
sudo java -jar /usr/src/IPED/target/release/iped-4.2-snapshot/iped.jar -d /home/<usuario>/caso1/teste.img -o /home/<usuario>/caso1/indexado
```

## Execução do IPED para análise

```
sudo java -jar /home/<usuario>/caso1/indexado/iped/lib/iped-search-app.jar 
```

## Referências

* https://github.com/sepinf-inc/IPED/wiki/User-Manual#python-modules
* https://github.com/sepinf-inc/IPED/wiki/Beginner's-Start-Guide
* https://github.com/sepinf-inc/IPED/
* https://github.com/iped-docker/iped
* https://github.com/sepinf-inc/sleuthkit
* https://github.com/sepinf-inc/IPED/wiki/Linux
* https://bell-sw.com/pages/repositories/#apt
* https://github.com/ninia/jep/wiki/Linux
