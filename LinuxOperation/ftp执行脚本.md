
#!/bin/sh
#设定自己的花名
MUSER="tianyuan"
echo "Your nickname is: "$MUSER
read -p "Your password is: " -s MPASSWD
if [ $# == "3" ];then
    DIR=$3
else
    DIR="/"
fi
ftp -niv <<EOF
open ftp.rabbit.org
user $MUSER $MPASSWD
binary
cd $DIR
prompt
$1 $2
close
bye
EOF


#!/bin/sh
#设定自己的花名
MUSER="pubftp"
echo "Your nickname is: "$MUSER
read -p "Your password is: " -s MPASSWD
if [ $# == "3" ];then
    DIR=$3
else
    DIR="/"
fi
ftp -niv <<EOF
open ip
user $MUSER $MPASSWD
binary
cd $DIR
prompt
$1 $2
close
bye
EOF
