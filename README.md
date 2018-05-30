# Unix-Scripts
This folder contains script to install a GUI desktop on a remote server over SSH.
The script also installs google chrome and chromium browesers and configure them to work for root user. 
It also autostart google chrome by default on startup.

## Following setup tools were used:
1. GUI Desktop = LXDE Desktop
2. Remote Desktop Client = Remote Desktop Connection provided by windows.
3. PuTTY for Windows or Windows PowerShell for SSH connection. 

##What does the script do? 
The script can be divided into the following parts based on the tasks it performs 

### [1.][Setting root password]
echo -e "qwe123\nqwe123" |sudo passwd root 

#Sets the password of root user as qwe123.

### [2.] [Installing all the necessary packages] 
sudo apt-get update;
sudo apt-get -y install lxde task-lxde-desktop xrdp chromium; 

#Download and update packages
Downloads the package lists from the repositories and updates them to the newest version of packages and their dependencies.The second command installs LXDE Desktop Environment,a low resource consuming desktop, along with open source implementation of RDP, xrdp, which helps in connecting via a windows desktop and Chromium browser.

### [3.][Starting LXDE Desktop]
echo "exec startlxde" > ~/.xsession;  

#Configures and Starts LXDE Desktop.

### [4.][Configuring Chrome to run as root user]
echo 'export CHROMIUM_FLAGS="$CHROMIUM_FLAGS --user-data-dir --no-sandbox"' > default;
sudo mv default /etc/chromium.d/;

#These commands configures chromium for root user. For running chromium as root user, we need to run it with "--no-sandbox" flag. 
Since all the files in chromium.d folder run before starting chromium,adding a file that contains 'export CHROMIUM_FLAGS="$CHROMIUM_FLAGS --user-data-dir --no-sandbox"' does the work as it starts chromium with "--no-sandbox" flag.

### [5.][Installing Google Chrome]
sudo sed -i -e '$a\deb http://dl.google.com/linux/chrome/deb/ stable main' /etc/apt/sources.list;
sudo wget https://dl-ssl.google.com/linux/linux_signing_key.pub;
sudo apt-key add linux_signing_key.pub;
sudo apt-get update;
sudo apt-get -y install google-chrome-stable;

#These commands install google-chrome-stable. 
Since there is no source listed to download google chrome so first we have to save the source "http://dl.google.com/linux/chrome/deb/" in sources.list using 'sed' command and running as super user.The next step is to get Google's signing key and adding it to our keyring so that the packet manager can verify the integrity of Google Chrome package which is accomplished by using 'wget' and 'apt-key' commands,which are run as super user.After that we update local package index and install stable version of Google Chrome.

### [6.][Configuring Google Chrome as root user]
sudo sed -i 's/"$@"/"$@" --user-data-dir --no-sandbox/g' /opt/google/chrome/google-chrome;

#This command configures chrome for root user by running it with "--no-sandbox" flag.
A small trick that I used to configure google-chrome executable is finding '"$@"', which comes exactly once in the file and replacing it with '"$@" --user-data-dir --no-sandbox',which allows it to run as a root user.

### [7.][Google Chrome Autostart]
sudo sed -i -e "\$a@/opt/google/chrome/google-chrome" /etc/xdg/lxsession/LXDE/autostart;

#Autostart google chrome.
It is achieved by providing the path of google-chrome executable in autostart file of LXDE desktop with @ which was done using sed command. 

### [8.][Restarting XRDP service]
sudo service xrdp restart;

# You can now connect to your remote server via Remote Desktop app on windows system or by rdesktop on linux. 
