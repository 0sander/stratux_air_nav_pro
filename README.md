# Getting Stratux to work with AirNavigation Pro

## The Problem
[Xample](https://www.airnavigation.aero/), the makers of Air Navigation Pro seem to have no interest in supporting the open source [Stratux](http://stratux.me/)  ADS-B, AHRS and GPS device. Currently they only support iLevil and FLARM devices as traffic sources. 

## The Solution

To make Air Navigation Pro use Stratux as a traffic, AHRS and GPS source we need to make the Stratux behave like one of the supported devices. iLevil actually uses the GDL-90 protocol that also Stratux uses. The only difference is, that iLevil uses a non-standard UDP port and the Air Navigation Pro seems to expect the UDP packages to come from the IP that iLevil devices normally use. iLevil uses the IP **169.254.1.1** and UDP port **43211**. So lets go ahead and change the IP and port settings in Stratux. For this tutorial I will be using Stratux [1.4r3](https://github.com/cyoung/stratux/tree/v1.4r3).

## Change the IP

You first need to SSH into your Stratux. On Windows you can use putty, on a Mac you can simply use your terminal:

    ssh -l pi 169.254.1.1

Enter the password "raspberry" and get root by entering

    sudo su

Then open the network settings with your favourite editor. I will use *nano* in this tutorial.

    nano /etc/network/interfaces
  Now set the IP by changing
  

    iface wlan0 inet static
      address 192.168.10.1
      netmask 255.255.255.0
      post-up /usr/sbin/stratux-wifi.sh
 
 to
 
    iface wlan0 inet static
      address 169.254.1.1
      netmask 255.255.255.0
      post-up /usr/sbin/stratux-wifi.sh
 
 To save and exit nano press *Ctrl+X* and *Y*
 
 Then change the DHCP server settings

     nano /etc/dhcp/dhcpd.conf
Edit the IP range by changing

    subnet 192.168.10.0 netmask 255.255.255.0 {
            range 192.168.10.10 192.168.10.50;
            option broadcast-address 192.168.10.255;
            option routers 192.168.10.1;
            default-lease-time 12000;
            max-lease-time 12000;
            option domain-name "stratux.local";
            option domain-name-servers 4.2.2.2;
    }

to

    subnet 169.254.1.0 netmask 255.255.255.0 {
            range 169.254.1.10 169.254.1.50;
            option broadcast-address 169.254.1.255;
            option routers 169.254.1.1;
            default-lease-time 12000;
            max-lease-time 12000;
            option domain-name "stratux.local";
            option domain-name-servers 4.2.2.2;
    }
 
 To save and exit nano press *Ctrl+X* and *Y*


## Change the port
To change the UDP port open the stratux configuration

    nano /etc/stratux.conf 
and find *"Port":4000* and change it to *"Port":43211*:

 To save and exit nano press *Ctrl+X* and *Y*

Finally reboot the Stratux

    shutdown -r now
and reconnect to Stratux. The web interface can now be found at:
http://169.254.1.1/#/

## Setting up Air Navigation Pro
Open Air Navigation Pro and connect your device to the Stratux WiFi. Then open the Configuration:
![enter image description here](https://github.com/0sander/stratux_air_nav_pro/raw/master/E3EC0B2D-E4E5-4803-AB5B-34AFAFD3D3A0.png)

Select Sensors:

![enter image description here](https://github.com/0sander/stratux_air_nav_pro/raw/master/91A17359-02C2-47E5-B0EA-82E5CA4574A7.png)

Select Levil AHRS G Mini:
![enter image description here](https://github.com/0sander/stratux_air_nav_pro/raw/master/18B1CD58-0BE5-487A-B2CA-85D81704CA26.png)

And change the settings as shown above. The sensors should now turn green and you should have working GPS, AHRS and traffic in Air Navigation Pro.

## Note
Changing the port to anything but the 4000 default is no longer officially supported by Stratux as companies like Xample should support Stratux / GDL90 instead. 
