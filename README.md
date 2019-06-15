# Install Tomcat 8.x in AWS EC2 Instance (Amazon Linux AMI)

You will find as well the configuration of **Nginx** and **PostgreSQL**. Special thanks to @caden for the contribution!

## Software to install
* Java 8
* Tomcat 8.x

## Check YUM Updates
```
sudo su
yum list installed
yum update
```

## Install Java 8
```
yum install java-1.8.0
yum remove java-1.7.0-openjdk
```

## Install Tomcat 8.x
```
yum install tomcat8 tomcat8-webapps tomcat8-admin-webapps tomcat8-docs-webapp
```

## Start Tomcat Service
```
service tomcat8 start
```

## Tomcat Paths
1 Edit the tomcat-users file
```
cd  /usr/share/tomcat8
vim /usr/share/tomcat8/conf/tomcat-users.xml
```
2 Add this line according your desired configuration (vi commands: "i" for insert mode, "ESC" key to escape the inserting mode, ":wq" for write an quit)
```
<user name="admin" password="YOURPASSWORD" roles="admin,manager,admin-gui,admin-script,manager-gui,manager-script,manager-jmx,manager-status" />
```

## Check Tomcat Installation
* fuser: to display the process id(PID) of every process using the specified files
```
fuser -v -n tcp 8080
```
* netstat: to list out all the network (socket) connections on a system
```
netstat -na | grep 8080
```

## Set Auto Start for Tomcat Service
```
sudo chkconfig --list tomcat8
sudo chkconfig tomcat8 on
```

## Copy JDBC driver to /usr/local/java//jre/lib/ext/ or /usr/lib/jvm/jre-1.7.0-openjdk.x86_64/lib/ext
cp -a ojdbc6.jar /usr/local/java//jre/lib/ext/
cp -a ojdbc6.jar /usr/lib/jvm/jre-1.7.0-openjdk.x86_64/lib/ext
service tomcat8 restart

## Create dbCon.jsp and dbconnection.jsp on /var/lib/tomcat8/webapps/ROOT
# dbCon.jsp
<%@ page language="java" import="java.sql.*" %>
<%
String DB_URL = "jdbc:oracle:thin:@awsdc-rds-prd-sales01.cf89zyffo8dr.ap-northeast-2.rds.amazonaws.com:1521:SALES01";
String DB_USER = "scott";
String DB_PASSWORD = "password";
Connection con = null;
Statement stmt = null;
ResultSet rs = null;
String sql=null;
try
 {
 Class.forName("oracle.jdbc.driver.OracleDriver");
 con = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
 out.println("Oracle Database Connected!");
 }catch(SQLException e){out.println(e);}
%>

# dbconnection.jsp
<%@ page language="java" import="java.sql.*" %>
<%
String DB_URL = "jdbc:oracle:thin:@awsdc-rds-prd-sales01.cf89zyffo8dr.ap-northeast-2.rds.amazonaws.com:1521:SALES01";
String DB_USER = "scott";
String DB_PASSWORD = "external#1234";
Connection con = null;
Statement stmt = null;
ResultSet rs = null;
String sql=null;
try
 {
 Class.forName("oracle.jdbc.driver.OracleDriver");
 con = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
 out.println("Oracle Database Connected!");
 }catch(SQLException e){out.println(e);}
%>
root@ip-10-10-1-226:/var/lib/tomcat8/webapps/ROOT# cat dbconnection.jsp 
<%@ page contentType="text/html; charset=UTF-8" %>
<%@ include file="dbCon.jsp" %> <!-- dbCon.jsp import -->
<%@ page import="java.util.*,java.text.*"%>
<html>
<head>
<title>test</title>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<link title=menustyle href="../adminstyle.css" type="text/css" rel="stylesheet">
<script language="JavaScript">
<!--
 function MM_openBrWindow(theURL,winName,features){
 window.open(theURL,winName,features);
 }
//-->
</script>
</head>
 
 
body bgcolor="#FFFFFF" text="#000000" leftmargin="0" topmargin="0" marginwidth="0" marginheight="0">
<table width="630" border="0" cellspacing="0" cellpadding="0">
 <%
 sql="select ENAME, JOB, MGR, HIREDATE from emp";
 
  stmt = con.createStatement();
  rs = stmt.executeQuery(sql);
 
  while(rs.next()) {
          String name=rs.getString("ENAME");
          String job=rs.getString("JOB");
          String mgr=rs.getString("MGR");
          String hiredate=rs.getString("HIREDATE");
          out.println(" ENAME : "+name+" JOB : "+job+" MRG :"+mgr+" DATE :"+hiredate+" <hr>");
 }
 
    if(rs != null) rs.close();
    if(stmt != null)stmt.close();
    if(con != null)con.close();
 %>
 
</body>
</html>



## Install Nginx
```
yum install nginx
```

## Add Nginx Domain Binding
```
vi /etc/nginx/conf.d/shenhe.org.conf
```

```
server {
    listen       80;
    listen       [::]:80;
    server_name  shenhe.org;
    root         /usr/share/nginx/html;
    location / {
        proxy_connect_timeout 300;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
        proxy_pass http://localhost:8080;
        }
}
```
You can add more bindings. It’s similar.
If you just want nginx to handle static files (like images, javascript, css etc), you just do it like this:
```
server {
    listen       80;
    listen       [::]:80;
    server_name  static.shenhe.org;
    root         /usr/share/nginx/html;
}
```
## Start Nginx
```
service nginx start
```
## Set Auto Star for Nginx Service
```
sudo chkconfig --list nginx
sudo chkconfig nginx on
```

## Install PostgreSQL
```
sudo yum install postgresql postgresql-server
```

## Initialize postgresql DB
```
sudo service postgresql initdb
```

## Start PostgreSQL
```
sudo service postgresql start
```

## Login in with postgres
```
sudo -u postgres psql
```

## Change Password
```
\password postgres
```

## Quit
```
\q
```
## Enable Remote Access
```
sudo vi /var/lib/pgsql9/data/pg_hba.conf
```
Add you IP to trust IP
```
host all all 125.69.29.0/24     trust
```
Also change this line from
```
host    all             all             127.0.0.1/32            ident
```
To
```
host    all             all             127.0.0.1/32            md5
```

```
sudo vi /var/lib/pgsql9/data/postgresql.conf
```
Change 
```
#listen_addresses = 'localhost'
```
To
```
listen_addresses = '*'
```
## Auto Start PostgreSQL
```
sudo chkconfig --list postgresql
sudo chkconfig postgresql on
```

## Add swap to EC2
```
sudo /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=2048
sudo /sbin/mkswap /var/swap.1
sudo chmod 600 /var/swap.1
sudo /sbin/swapon /var/swap.1
```

## Auto mount Swap
To enable it by default after reboot, add this line to /etc/fstab:
```
/var/swap.1  swap        swap    defaults        0   0
```
