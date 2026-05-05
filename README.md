# Litex-install-Ubuntu-25.10-and-Vivado-2018.2

Installation instructions to install Litex SOC builder on Ubuntu 25.10 and Vivado 2018.2    

## Why?   

I wanted to get a stable virtual environment to act as a base for Litex work.   
Using Virtualbox meant it could be run from Windows when needed.   

## Useful links   

https://www.controlpaths.com/2022/01/17/building-soc-litex/   
https://github.com/enjoy-digital/litex/tree/master  
https://github.com/enjoy-digital/litex/tree/master/litex/soc/software/demo   
https://github.com/trabucayre/openFPGALoader   
https://trabucayre.github.io/openFPGALoader/guide/install.html   

## Versions

- VirtualBox 7.1.6    
- Ubuntu 25.10    
- Vivado 2018.2    

## VirtualBox install
Install VirtualBox 7.1.6    
Create new VM – “Vivado 2018.2”    
Settings:    
- Ubuntu 64 bit    
- 6 CPU    
- 8GB RAM    
- 60GB storage    

For this keyboard set AltGr or RIGHT ALT as home key    
In Settings / General / Advanced set bi-directional clipboard    

## Ubuntu install
Download Ubuntu image ubuntu-25.10-desktop-amd64    

Menu option Storage, Controlled: IDE add ISO image for Ubuntu.    

Start the VM    
Continue the Ubuntu install process    
Set up a user    
Log in    

## VirtualBox guest additions    
In Devices menu option    
Insert Guest Additions CD image...    

```
  cd /media/paul/VBox_GAs_7.1.6
  ./VBoxLinuxAdditions.run.
```

Set shared folders in ```d:\VirtualBoxShare``` mounted to ```/mnt/VirtBoxShare```   

## Vivado 2018.2 install    
Download 2018.2 from AMD website – Vivado Design Suite – HLx Editions – 2018.2 Full Product Installation    
Xilinx_Vivado_SDK_2018.2_0614_1954.tar.gz (17.11 GB)    

Copy the downloaded Xilinx install to /mnt/VirtBox/Installs    
Ensure we do a symbolic link for the two missing libraries    

```
  ln -s /usr/lib/x86_64-linux-gnu/libtinfo.so.6    /usr/lib/x86_64-linux-gnu/libtinfo.so.5
  ln -s /usr/lib/x86_64-linux-gnu/libncurses.so.6  /usr/lib/x86_64-linux-gnu/libcurses.so.5
```
Then do the install    

```
  cd /home/paul
  mkdir Xilinx
  cd Xilinx

  tar xvf /mnt/VirtBox/Installs/Xilinx_Vivado_SDK_2018.2_0614_1954.tar.gz 

  cd Xilinx_Vivado_SDK_2018.2_0614_1954
  ./xsetup
```

Use /opt/Xilinx as the install location    
Set options - only Artix 7 and Spartan 7    
SDK compiler toolsets - none    

```
   cd /opt/Xilinx/Vivado/2018.2
   source settings64.sh
```

## Litex install

```
   sudo apt-get install git pip
   sudo apt install python3.13-venv

   mkdir my_litex
   cd my_litex

   python3 -m venv venv
   source venv/bin/activate


   wget https://raw.githubusercontent.com/enjoy-digital/litex/master/litex_setup.py
   chmod +x litex_setup.py

   ./litex_setup.py --init --install
```

## RISC V toolchain install

```
   pip3 install meson ninja
   sudo ./litex_setup.py --gcc=riscv

```

## OpenFPGALoader
```
   sudo apt-get install openfpgaloader
```

## USB Device
(https://trabucayre.github.io/openFPGALoader/guide/install.html)    

```
   sudo vi /etc/udev/rules.d/99-ftdi.rules
```

Entry for original FT232: 

```
   ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6010", MODE="0666"
```

or (not tested but openFPGALoader install instructions)    

```
   wget https://github.com/trabucayre/openFPGALoader/blob/master/99-openfpgaloader.rules

   grep -w plugdev /etc/group

   sudo groupadd --system plugdev # only required if plugdev is absent
   sudo cp 99-openfpgaloader.rules /etc/udev/rules.d/
   sudo udevadm control --reload-rules && sudo udevadm trigger # force udev to take new rule
   sudo usermod -a $USER -G plugdev # add user to plugdev group
```


## First build and load
(See https://trabucayre.github.io/openFPGALoader/guide/first-steps.html)    
For the Arty A7 board (a7-100)    

```
   source /opt/Xilinx/Vivado/2018.2/settings64.sh

   cd ~/my_litex/litex-boards/litex_boards/targets
   ./digilent_arty.py --variant a7-100 --build --load
```

Could also be:   

```
   source /opt/Xilinx/Vivado/2018.2/settings64.sh

   python3 -m litex_boards.targets.digilent_arty –build --load
```



## Load   

```
   cd ~/my_litex/litex-boards/litex_boards/targets/build/digilent_arty/gateware
   openFPGALoader -b arty digilent_arty.bin
```

## Terminal
```
   litex_term /dev/ttyUSB1
```

## Full run for Nexys4 DDR
Load up the Ubuntu VM and log in    

```
   cd my_litex
   source venv/bin/activate

   cd /opt/Xilinx/Vivado/2018.2
   source settings64.sh

   cd ~/my_litex/litex-boards/litex_boards/targets

   ./digilent_nexys4ddr.py --build –load

   cd ~/my_litex/litex-boards/litex_boards/targets/build/digilent_nexys4ddr/gateware
   openFPGALoader -b nexys_a7_100 digilent_nexys4ddr.bit

   litex_term /dev/ttyUSB1
```



## Generate the demo program
Load the Ubuntu VM and log in    

Create the SOC    

```
   cd my_litex
   source venv/bin/activate

   cd /opt/Xilinx/Vivado/2018.2
   source settings64.sh

   cd ~/my_litex/litex-boards/litex_boards/targets
   ./digilent_nexys4ddr.py --build –load
```

Then the demo program    

```
   cd ~/my_litex/litex/litex/soc/software/demo/

   export BUILD_DIR=~/my_litex/litex-boards/litex_boards/targets/build/digilent_nexys4ddr/
   mkdir build 
   cp * build
   cd build
   make

   cd ~/my_litex/litex-boards/litex_boards/targets/build/digilent_nexys4ddr/gateware
   openFPGALoader -b nexys_a7_100 digilent_nexys4ddr.bit

   litex_term /dev/ttyUSB1 --kernel  demo.bin
```

Alternatively using the ```demo.py``` script.

```
   cd ~/my_litex/litex/litex/soc/software/demo
   python3 demo.py --build-path=/home/paul/my_litex/litex-boards/litex_boards/targets/build/digilent_nexys4ddr/

   openFPGALoader -b nexys_a7_100 /home/paul/my_litex/litex-boards/litex_boards/targets/build/digilent_nexys4ddr/gateware/digilent_nexys4ddr.bit

   litex_term /dev/ttyUSB1 --kernel  demo.bin
```

## Replace the BIOS with the demo   

```
   cd my_litex
   source venv/bin/activate

   cd /opt/Xilinx/Vivado/2018.2
   source settings64.sh

   cd ~/my_litex/litex/litex/soc/software/demo

   python3 demo.py --build-path=/home/paul/my_litex/litex-boards/litex_boards/targets/build/digilent_nexys4ddr/ --mem=rom

   python3 -m litex_boards.targets.digilent_nexys4ddr --integrated-rom-init=demo.bin --build 

   openFPGALoader -b nexys_a7_100 \
      ~/my_litex/litex-boards/litex_boards/targets/build/digilent_nexys4ddr/gateware/digilent_nexys4ddr.bit

   litex_term /dev/ttyUSB1 
```

Or   
```
   cd my_litex
   cd ~/my_litex/litex/litex/soc/software/demo

   python3 demo.py \
       --build-path=/home/paul/my_litex/litex-boards/litex_boards/targets/build/digilent_nexys4ddr/ \
       --mem=rom

   cd ~/my_litex/litex-boards/litex_boards/targets

   ./digilent_nexys4ddr.py --build \
      --integrated-rom-init=/home/paul/my_litex/litex/litex/soc/software/demo/demo.bin 

   openFPGALoader -b nexys_a7_100 \
      ~/my_litex/litex-boards/litex_boards/targets/build/digilent_nexys4ddr/gateware/digilent_nexys4ddr.bit

   litex_term /dev/ttyUSB1 
```




