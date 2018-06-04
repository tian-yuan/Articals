一、mysql 连接 
	
	mysql -u root -p
	
二、php 连接 mysql

	<html>
	<head>
	<title>Connecting MySQL Server</title>
	</head>
	<body>
	<?php
		$dbhost = 'localhost:3306';  //mysql服务器主机地址
		$dbuser = 'guest';      //mysql用户名
		$dbpass = 'guest123';//mysql用户名密码
		$conn = mysql_connect($dbhost, $dbuser, $dbpass);
		if(! $conn )
		{
			die('Could not connect: ' . mysql_error());
		}
		echo 'Connected successfully';
		mysql_close($conn);
		?>
	</body>
	</html>	
	
	解决 mysql_connect(): No such file or directory 问题
	If yours is /tmp/mysql.sock but no /var/mysql/mysql.sock you should:
		cd /var 
		mkdir mysql
		cd mysql
		ln -s /tmp/mysql.sock mysql.sock
	If you have /var/mysql/mysql.sock but no /tmp/mysql.sock then:
	
		cd /tmp
		ln -s /var/mysql/mysql.sock mysql.sock

