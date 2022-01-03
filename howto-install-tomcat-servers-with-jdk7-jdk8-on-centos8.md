# How to Install Tomcat Servers with JDK 1.7 and JDK 1.8 on CentOS 8

## Install JDK 1.7 and 1.8

Download JDK 1.7 and 1.8, and extract them to `/opt`:

```sh
cd /usr/src
tar zxvf jdk-7u80-linux-x64.tar.gz
tar zxvf jdk-8u311-linux-x64.tar.gz

mv jdk1.7.0_80/ /opt
mv jdk1.8.0_311/ /opt
```

Export env variables for system commands:

```sh
cat >> ~/.bashrc <<'EOF'

export JAVA_HOME=/opt/jdk1.8.0_311
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
EOF

source ~/.bashrc
```

Verify `java` command for JDK 1.8 and `java7` command for JDK 1.7:

```
[root@localhost ~]# java -version
java version "1.8.0_311"
Java(TM) SE Runtime Environment (build 1.8.0_311-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.311-b09, mixed mode)
```

## Install Apache Tomcat 8 (Compatiable with JDK 1.7+)

Download Apache Tomcat 8, and extract them to `/opt`:

```sh
cd /usr/src
wget https://mirrors.bfsu.edu.cn/apache/tomcat/tomcat-8/v8.5.73/bin/apache-tomcat-8.5.73.tar.gz
tar zxvf apache-tomcat-8.5.73.tar.gz
mv apache-tomcat-8.5.73/ /opt
```

Create a copy of Tomcat with JRE 7:

```sh
cp -a /opt/apache-tomcat-8.5.73 /opt/apache-tomcat-8.5.73_jre7

useradd -m -s /sbin/nologin -U tomcat
chown -R tomcat:tomcat /opt/apache-tomcat-8*
```

Config systemd service:

```sh
cat > /etc/systemd/system/tomcat.service <<'EOF'
[Unit]
Description=Tomcat 8.5
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment=JAVA_HOME=/opt/jdk1.8.0_311/jre
Environment=CATALINA_BASE=/opt/apache-tomcat-8.5.73_jre8
Environment=CATALINA_HOME=/opt/apache-tomcat-8.5.73_jre8
Environment=CATALINA_PID=/opt/apache-tomcat-8.5.73_jre8/temp/tomcat.pid
Environment=CATALINA_OPTS=-Xms1g -Xmx1g -server

ExecStart=/opt/apache-tomcat-8.5.73/bin/catalina.sh start
ExecStop=/opt/apache-tomcat-8.5.73/bin/catalina.sh stop

[Install]
WantedBy=multi-user.target
EOF

cat > /etc/systemd/system/tomcat_jre7.service <<'EOF'
[Unit]
Description=Tomcat 8.5 with JRE 7
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment=JAVA_HOME=/opt/jdk1.7.0_80/jre
Environment=CATALINA_BASE=/opt/apache-tomcat-8.5.73_jre7
Environment=CATALINA_HOME=/opt/apache-tomcat-8.5.73_jre7
Environment=CATALINA_PID=/opt/apache-tomcat-8.5.73_jre7/temp/tomcat.pid
Environment=CATALINA_OPTS=-Xms1g -Xmx1g -server

ExecStart=/opt/apache-tomcat-8.5.73_jre7/bin/catalina.sh start
ExecStop=/opt/apache-tomcat-8.5.73_jre7/bin/catalina.sh stop

[Install]
WantedBy=multi-user.target
EOF
```

Change port number in **Tomcat with JRE 7** server config`/opt/apache-tomcat-8.5.73_jre7/conf/server.xml`, from `8005`, `8080`, and `8443`  to `8006`, `8081`, and `8444`.

Enable and start up tomcat servers:

```sh
systemctl daemon-reload
systemctl enable tomcat.service
systemctl start tomcat.service
systemctl enable tomcat_jre7.service
systemctl start tomcat_jre7.service
```
