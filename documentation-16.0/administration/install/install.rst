Thrive Bureau ERP

For installation in VPS server use these three lines, in the end of the installation process, when asking for GitHub user id: then input your GitHub email then input your password, it will install thrive_bureau_erp.

login to vps terminal then insert three lines then it will start automatically start the installation process

step 1)
sudo wget https://raw.githubusercontent.com/u16052642/thrive_erp_install_script/main/thrive1669_install.sh

step 2)
sudo chmod +x thrive1669_install.sh

step 3)
sudo ./thrive1669_install.sh


after finishing installation you will see the (yourvpsip):8069 then you should provide the master password then use database name and user id and password

when need manually start restart or stop then use this command

sudo /etc/init.d/thrive1669-server restart
sudo /etc/init.d/thrive1669-server stop
sudo /etc/init.d/thrive1669-server start
