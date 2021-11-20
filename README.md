# info
- had a rough time installing amd gpu drivers on linux ubuntu server. so i detailed the process for my future reference and to possibly help others
- used for: ubuntu server crypto mining rig
- worked on: november 17 2021
- iso file I used: ubuntu-20.04.3-live-server-amd64.iso
- everything was done from a windows pc using ssh (besides the install of course)
- commands with ```# some description here``` are safe to copy/paste (# = comment)

# download iso
- https://ubuntu.com/#download
- find ubtuntu server (LTS version: long term support)

# burn linux iso to usb drive
- download https://rufus.ie/en/
- use rufus to burn .iso

# installing ubuntu server
- boot server pc with usb plugged in and select install and follow the instructions
- not being very detailed about this step because i'm assuming you have installed linux before
- while on the ubuntu server pc. make note of the ip address since it is needed to ssh in
```
ip a | grep -i net # helps find ip
```

# connect to ubuntu server pc
- from your main pc open a terminal and type: ssh username@ip_address
```
ssh rig1@10.0.0.237
```

# network settings
- if you're using wifi you might have to fuck with the internet settings
- if the internet is working after a fresh install i'd just leave it alone and skip this to save your sanity

```
sudo apt install network-manager # install network-manager
```
```
sudo nano /etc/netplan/*.yaml # open network config file to edit
```
```
netplan generate && netplan apply # apply new network settings
```

example /etc/netplan/*.yaml file
```
  network:
    ethernets:
        enp7s0:
          dhcp4: true
        optional: true
    version: 2
    wifis:
      wlp6s0:
        renderer: NetworkManager
        dhcp4: true
        access-points:
          "your_wifi_name+keep_the_quotes":
            password: "your_wifi_password+keep_the_quotes"   
```
- if there are multiple files in /etc/netplan/ the file chosen will be in filename order
- 001xyz.yaml will be picked over 002xyz.yaml

- troubleshooting
```
ip a # troubleshooting: find device names
```
```
lspci # can't find device name? might be a network driver issue (lspci + google + good luck)
```

# increase swap
- skip this if you have lots of ram
- i only had 2gb of ram but had 100gb ssd sitting doing nothing so i choose to increase my swap space (swap space = hard drive used as ram)
```
sudo swapoff /swap.img # turn swap off
sudo fallocate -l 8G /swap.img # change swap size to 8GB
sudo mkswap /swap.img # mk new swap img
sudo swapon /swap.img # turn swap back on
```

# updates
```
sudo apt update && sudo apt upgrade && sudo reboot # updates system and restarts
```

# amd video driver install
- refer to this if you need extra help https://amdgpu-install.readthedocs.io/en/latest/

```
wget https://repo.radeon.com/amdgpu-install/latest/ubuntu/focal/amdgpu-install-21.40.40500-1_all.deb # download install package from offical amd site
```
```
sudo apt-get install ./amdgpu*.deb && sudo apt-get update # install package and update
```
```
sudo dpkg --add-architecture i386 # prevents error
```
```
amdgpu-install -y --accept-eula --usecase=workstation,rocm,opencl --vulkan=pro --opencl=rocr,legacy --no-32  # installs driver and componets. grab a coffee this command may take a while
```
```
sudo usermod -aG video rig1 # add user to video group (rig1 = your username)
```
finding link in the wget command
- went to https://repo.radeon.com/amdgpu-install/latest/ubuntu/focal/
- right-clicked the link (brings up menu)
- clicked copy link (in firefox at least)

- alternative: if for some reason wget doesn't work can try to download the file on your main pc then scp file transfer over

```
cd c:\what_ever_dir_has_the_file_in_it_probably_the_download_folder_thought
scp amd*.dep rig1@10.0.0.237:/home/rig1
```

- sudo dpkg --add-architecture i386 prevents the below error
```
The following packages have unmet dependencies:
 amdgpu-pro-lib32 : Depends: amdgpu-lib32 (= 21.40.40500-1331380) but it is not going to be installed
                    Depends: libgl1-amdgpu-pro-glx:i386 (= 21.40-1331380) but it is not installable
                    Depends: libegl1-amdgpu-pro:i386 (= 21.40-1331380) but it is not installable
                    Depends: libgles2-amdgpu-pro:i386 (= 21.40-1331380) but it is not installable
                    Depends: libglapi1-amdgpu-pro:i386 (= 21.40-1331380) but it is not installable
                    Depends: libgl1-amdgpu-pro-dri:i386 (= 21.40-1331380) but it is not installable
```
- trobleshooting: uninstalling
- something go wrong? you can try starting over using this
```
amdgpu-uninstall
sudo apt-get purge amdgpu-install
```


# installing mining software
- find and copy the download link of the miner (usually found on github)
- use the wget command to download shit in linux terminal
- then use the tar command to extract the shit that was downloaded
- here is an example for nbminer

```
wget https://github.com/NebuTech/NBMiner/releases/download/v39.7/NBMiner_39.7_Linux.tgz
tar -xf NBMiner*.tgz
ls # if you want to see if it was extracted properly
cd NB # then press tab to auto-complete because you are way to lazy to type out the whole name every time
nano start_ergo.sh # to edit your info
./start_ergo.sh # to start mining
```
