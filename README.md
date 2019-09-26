#!/bin/bash 

############### START DESCRIPTION #################
# Script Purpose : Install Web and App Components on CentOS 7 
# 
#
#
# Exit States
# 0 - SUCCESS
# 1 - Executed by Non-Root User / if user input is missing
# 2 - Script Failure
############### END DESCRIPTION ###################

### Global Variables
LOG=/tmp/stack.log 
rm -f $LOG
R="\e[31m"
RB="\e[1;31m"
N="\e[0m"
YB="\e[1;33m"
GB="\e[1;32m"
C="\e[36m"
MB="\e[1;35m"
APPUSER="student"

### Functions

## This function is to proint an error message
Error() {
  #echo -e "\t\t\t${YB}>>>>>>>>>>>>>>>ERROR<<<<<<<<<<<<<<<<${N}"
  #echo -e "${R}$1${N}"
  echo -e "${C}$(date +%F-%T) ${MB}$COMPONENT${N} ${RB}ERROR:${N} $1"
}

Success() {
  echo -e "${C}$(date +%F-%T) ${MB}$COMPONENT${N} ${GB}SUCCESS:${N} $1"
}

Head() {
  echo -e "\n\t\t\t${YB}>>>>>>>>>>>>>>  $COMPONENT SETUP  <<<<<<<<<<<<<<<<<${N}\n"
}

Status_Check() {
  case $1 in 
    0) 
      Success "$2" 
      ;;
    *) 
      Error "$2"
      echo -e "\t Refer Logfile : $LOG for error\n"
      exit 2 
      ;;
  esac
}

### Main Program



## Check whether the user running the script is a root user or not 
USER_ID=$(id -u)
if [ "$USER_ID" -ne 0  ]; then 
  Error "You should be a root user to execute this script"
  exit 1
fi 

### Config Web Server 
COMPONENT=WEBSERVER
Head  ## TO print heading 
yum install httpd -y &>>$LOG
Status_Check $? "Installing Web Server"

echo 'ProxyPass "/student" "http://localhost:8080/student"
ProxyPassReverse "/student"  "http://localhost:8080/student"' >/etc/httpd/conf.d/app-proxy.conf  
Status_Check $? "Setup Application Proxy Config"

curl -s https://s3-us-west-2.amazonaws.com/studentapi-cit/index.html -o /var/www/html/index.html &>>$LOG 
Status_Check $? "Setup Default Application Web Page"

systemctl enable httpd &>/dev/null 
systemctl restart httpd &>>$LOG 
Status_Check $? "Start Web Server"


### Config App Server 
COMPONENT=APPSERVER
Head # To print heading 
yum install java -y &>>$LOG 
Status_Check $? "Install Java"

id $APPUSER &>>$LOG 
if [ $? -eq 0 ]; then   
  Status_Check "0" "Add Application User"
else 
  useradd  &>>$LOG 
  Status_Check "$?" "Add Application User"
fi 



