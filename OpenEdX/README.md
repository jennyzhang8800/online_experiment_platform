edx在Ubuntu12.04 64上的部署
======

安装oraclejdk
======
第一步：区分32位还是64位操作系统
先确定你的ubuntu linux是32位还是64位的，方法很多，这里介绍一种即可。

    $uname -a

第二步：卸载OpenJDK
先执行如下命令看是否安装了OpenJDK，如果已经安装，会显示java的信息。

    $java -version

如果安装了OpenJDK，可用如下方法全部卸载：

    $sudo apt-get purge openjdk-\*

第三步：下载Oracle JDK版本
注：这里以Oracle 1.6 64位的为例
进入 http://www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-javase6-419409.html
，选择"Java SE Development Kit 6u45"，然后在新打开的页面点选接受"Accept License Agreement". 接受之后，选择" jdk-6u45-linux-x64.bin"即可进行下载。如果你没有登录，系统会先转到登录页面，输入你在Oracle网站注册的用户名和密码即可。

第三步：安装Oracle JDK

(1) 
创建java目录

    $ sudo mkdir -p /usr/local/java
    
将你下载的jdk-6u45-linux-x64.bin拷贝至/usr/local/java目录

    $ cd /usr/local/java
    $ sudo cp /home/dennis/Downloads/jdk-6u45-linux-x64.bin .

(2) 
解压bin文件

    $ sudo chmod +x jdk-6u45-linux-x64.bin
    $ sudo ./jdk-6u45-linux-x64.bin
    $ sudo rm -rf jdk-6u45-linux-x64.bin

第四步：配置Orache JDK

(1) 
配置JAVA_HOME和PATH环境变量

    $ sudo vi /etc/profile
    
在该文件的末尾加上如下部分：

    JAVA_HOME=/usr/local/java/jdk1.6.0_45
    PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
    export JAVA_HOME
    export PATH

(2) 
配置ubuntu的JDK和JRE的位置

    $ sudo update-alternatives --install "/usr/bin/java" "java" "/usr/local/java/jdk1.6.0_45/bin/java" 1
    $ sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/local/java/jdk1.6.0_45/bin/javac" 1
    $ sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/local/java/jdk1.6.0_45/bin/javaws" 1

(3) 
配置Oracle为系统默认JDK/JRE

    $ sudo update-alternatives --set java /usr/local/java/jdk1.6.0_45/bin/java
    $ sudo update-alternatives --set javac /usr/local/java/jdk1.6.0_45/bin/javac
    $ sudo update-alternatives --set javaws /usr/local/java/jdk1.6.0_45/bin/javaws

配置完成后，执行如下命令使其立即生效。

    $ . /etc/profile
    
再次执行"java -version"显示如下：

    dennis@dubuntu1404:~$ java -version
    java version "1.6.0_45"
    Java(TM) SE Runtime Environment (build 1.6.0_45-b06)
    Java HotSpot(TM) 64-Bit Server VM (build 20.45-b01, mixed mode)

开始部署edx
======
(1)
更新包

    sudo apt-get update -y
    sudo apt-get upgrade -y
    sudo reboot

(2)
安装所需的软件,pip为包管理系统;使用virtualenv为python开发创建一个隔离的环境

    sudo apt-get install -y build-essential software-properties-common python-software-properties curl git-core libxml2-dev      libxslt1-dev libfreetype6-dev python-pip python-apt python-dev
    sudo pip install --upgrade pip
    sudo pip install --upgrade virtualenv

(3)
从github下载配置包

    cd /var/tmp
    git clone -b release https://github.com/edx/configuration
    
为方便ssh访问，
将 configuration/playbooks/roles/common/defaults/main.yml 文件中的变量 COMMON_SSH_PASSWORD_AUTH 更改为 “yes”。

(4)
由于后续的使用ansible的安装过程中elasticsearch可能安装失败，所以在此提前安装elasticsearch

    wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-0.90.3.tar.gz 

    tar xvf elasticsearch-0.90.3.tar.gz   

进入config文件夹，编辑elasticsearch.yml，修改下面两行配置 

    node.name: "name of node"  
    node.master: true

进入到bin目录 

    ./elasticsearch

访问http://host:9200 如果可以访问，说明elasticsearch已经启动。

(5)
利用pip安装包

    cd /var/tmp/configuration
    sudo pip install -r requirements.txt

(6)
利用ansible进行安装,此处耗时可能较长

    cd /var/tmp/configuration/playbooks && sudo ansible-playbook -c local ./edx_sandbox.yml -i "localhost,"
    
可能有某个工具安装失败,如果出现则手动安装该工具后再重新安装

(7)
安装完成后,在本地的80端口访问lms,18010端口访问studio
