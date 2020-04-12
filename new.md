---
layout: default
---

## Build a New SWG Server

This guide will help you build a new SWG Server VM from scratch. I recommend following this guide's every detail at least once before attempting to build a server with different settings or anything.

### Prerequisites

You need to have [Oracle VirtualBox](https://www.virtualbox.org/) installed on your host machine before you begin.

You should also download the installation media for Linux Mint 19.3 from [here](https://www.linuxmint.com/edition.php?id=278). This guide uses Linux Mint Xfce because I like it.

### Prepare your VM for installation

In VirtualBox, Select `Machine -> New...`. You will be prompted with a window to begin creating your VM. Enter Expert Mode if you're not already there. Then give your VM a name, choose where it will be saved, and select `Type: Linux` and `Version: Ubuntu (64-bit)`. Choose how much RAM you will give your VM and be sure that you have the `Create a virtual hard disk now` option selected.

![](assets/images/new/001.PNG)

>A note on memory size: To run the _entire_ game, you will need at least 16 GB of RAM in your VM. If you don't have that much, just choose an amount as high as you can go while still leaving enough for the host to be comfortable running a SWG client. In this example, my host has only 12 GB of RAM, so I will give 8 GB to the VM and keep 4 GB for the host. Individual mileage may vary.

After clicking `Create`, you will get a window asking you for details about your Virtual Hard Disk. Defaults should be fine, except that you'll want to give your VM more than the default 10 GB of disk space. 100 GB should be plenty, and as long as you've got dynamic allocation selected, you don't even loose that space until you actually need to use it.

![](assets/images/new/002.PNG)

Click create, and your VM will show up in VirtualBox.

Before booting it up, we'll need to change a few more options. With your VM selected, click `Settings` to bring up the settings window.

+ For `System -> Processor -> Processor(s)`, give the VM pretty much as many CPUs as you can without bringing the slider into the red.
+ For `Display -> Screen -> Video Memory`, bring the slider all the way to 128 MB.
+ Select `Storage -> Storage Devices -> Controller: IDE -> Empty`. Select the Disk icon to the right of the dropdown menu under `Attributes -> Optical Drive`, select `Choose Virtual Optical Disk File...` and navigate to and open the Linux Mint Xfce iso file you downloaded earlier.
+ (Optional:) If you are running your VM from a solid-state drive, click on your vdi under `Storage Devices` and select the `Solid-state Drive` checkbox under `Attributes`.
+ For `Network -> Adapter 1 -> Attached to:`, select `Bridged Adapter`.

That's it for settings! Click `OK` to close the window.

With your VM still selected, click the big green arrow that says `Start`. The VM will boot and bring you to Linux Mint's desktop. Double click the `Install Linux Mint` icon to begin the installation.

The default keyboard layout is fine, so press `Continue`.

On the next page, select both checkboxes and click `Continue`.

On the next page, the default selection should be `Erase disk and install Xubuntu`. This is good, so click `Install Now`. A confirmation window will appear, so click `Continue`.

The `Where are you` screen will try to guess at where you are. If it's wrong, make a new selection manually. This is for localization settings, time zone, etc. When you're ready, click `Continue`.

Next is the `Who are you` screen. Input `swg` for everything and select `Log in automatically`. Click `Continue`.

![](assets/images/new/003.PNG)

The installer will run for a while now and you definitely don't want to skip anything, so feel free to go make a sandwich or something. When it's done, it'll ask you to restart. Go ahead and click `Restart Now`. It'll ask you to remove the installation media then press ENTER. VirtualBox actually removes the installation media automatically, so just go ahead and press ENTER. After it finishes rebooting, you should arrive at your desktop. After a few moments, a window should appear that offers to install updates for you. This is good, so click `Install Now` and let it run. When it's done, press `OK`.

Before we start installing server software, we want to prevent the OS from locking up or turning off automatically. Go to `Start -> Settings -> Power Manager`. Click the `Display` tab, move all the sliders to `Never`, and then just turn off `Display power management`. Click `Close` when you're done.

![](assets/images/new/004.PNG)

We also want to install VirtualBox Guest Additions. In the VM window, select `Devices -> Insert Guest Additions CD image..`. The CD will be mounted and a file explorer should open automatically. In the file explorer, click `File -> Open Terminal Here`. At the command line, run the following commands: (Note: The sudo password is `swg`)

```
sudo apt install build-essential dkms linux-headers-generic -y
sudo ./VBoxLinuxAdditions.run
```

When the script finishes, close the terminal and file explorer. Eject the Guest Additions CD by right clicking its icon on the desktop and selecting `Eject Volume`. You will now be able to maximize the VM window and the desktop will resize automatically. You should now reboot the VM by clicking the power icon in the start menu and selecting `Restart`.

>Check the clock on your Xubuntu desktop. If installing Guest Additions makes your clock incorrect, you shouldn't continue. The script that comes in the next step needs your clock to be cooperating. I had an issue while writing this guide where I had daylight savings set incorrectly on my host and then my time set manually for some reason. VirtualBox didn't like that at all. Anyways, just make sure your clock isn't screwed up. Hopefully it works for you without any hassle.

After rebooting, we want our server to have a static IP address. First, it's helpful to know what our current IP address is. Open a terminal and run this command:

```
ifconfig
```

You'll get an output that looks like this. Pay attention to the numbers that correspond with the ones circled in red. Yours will be different. Once you've noted this IP address, you may close the terminal.

![](assets/images/new/018.PNG)

Now, click the network icon (up and down arrows) on the right side of the taskbar and click `Edit Connections...`. In the window that appears, select `Wired connection 1` and click the gear icon at the bottom of the window. Click on the tab called `IPv4 settings`. For the Method, select `Manual`. Then under addresses, click `Add`. For your address, enter an address that has the same first three numbers as the IP address you noted earlier, but substitute the fourth number for `250`. In the example, I started with the IP address `192.168.1.76`, so I will enter `192.168.1.250`. This should almost always be safely outside your DHCP address pool. If you press Tab when you're done, the Netmask will be populated automatically. The Gateway should be the IP address of your router. (If you're not sure, check for a sticker on your router or maybe the manual. You can also run `route -n` from a terminal.) For your DNS servers, enter `1.1.1.1,8.8.8.8,8.8.4.4`. Click `Save` when you're done and close the Network Connections window.

![](assets/images/new/019.PNG)

Now, click the network icon again and select `Disconnect`. Then click the icon again and select `Wired connection 1`. This will cause your machine to switch to the new address.

>As a sanity check, you can open a terminal and rerun the `ifconfig` command from earlier. You should see your new IP address in the place of your old one.

We should now add our static IP address to the hosts file. Open a terminal and enter this command:

```
sudo nano /etc/hosts
```

Remove `127.0.1.1` from the second line and replace it with your static IP Address.

![](assets/images/new/011.PNG)

Press `CTRL + X` to exit. At the prompt, press `y` to indicate that you want to save, and press enter to accept the same filename.

Let's next install Oracle Database 18. The following commands will run a script to get the system ready for installation, including downloading the software. This may take a while depending on your internet connection.

```
wget https://raw.githubusercontent.com/tekaohswg/oinit/master/oinit.sh
chmod +x oinit.sh
sudo ./oinit.sh
```

Once the script has finished, we're ready to run the actual installer. (Note: When you're prompted for it, the password for oracle is `swg`.)

```
xhost +
su - oracle
/u01/app/oracle/product/18/dbhome_1/runInstaller
```

The graphical installer will load. A warning will appear to let you know that the operating system is not supported. Just click `yes` to continue.

![](assets/images/new/005.PNG)

We want the option `Set Up Software Only`. Selected that and click `Next` to continue.

![](assets/images/new/006.PNG)

The default options on the next two screens, `Single instance database installation` and `Enterprise Edition`, are suitable. You may just click `Next` to continue.

The default `Oracle base` and `Inventory Directory` are also good. Just click `Next` for these screens.

The `Operating System groups` are also fine at their defaults. Click `Next` again for this page.

Finally, you'll be presented with a summary screen. Everything should be good, so click `Install`.

The installer will run for several minutes. Finally, you'll be asked to run a couple of scripts to complete the installation.

![](assets/images/new/007.PNG)

_Without_ closing the window or clicking `OK`, open a **NEW** terminal (don't close the old one) and run the commands with `sudo`:

```
sudo /u01/app/oraInventory/orainstRoot.sh
sudo /u01/app/oracle/product/18/dbhome_1/root.sh
```

The second one will ask for input twice. Just press `Enter` both times to keep the defaults. Then, close the terminal you just opened and press `OK` on the window that opened up before. If everything went well, you'll get confirmation that the installation was successful. Click `Close` to finish the installer.

![](assets/images/new/008.PNG)

Back to the first terminal that has been open this whole time, run the following commands to run a post-install script. This is needed before trying to create our database.

```
wget https://raw.githubusercontent.com/tekaohswg/oinit/master/postinst.sh
chmod +x postinst.sh
./postinst.sh
```

Still in the same terminal, use this command to launch the Oracle Net Configuration Assistant.

```
netca
```

The default option is `Listener Configuration`. This is what we want, so click `Next`.

On the next window, the only available option is `Add`, so just click `Next`.

Up next, the default name `LISTENER` is good, so click `Next`.

Nothing to change this page either. Just click `Next`.

Standard port `1521` is good, so click `Next` again.

Leave the next option as `No` and press `Next`.

`Listener configuration complete!` Press `Next`. Then, press `Finish` to close this window.

Up next is the Database Configuration Assistant. Again in the same terminal run:

```
dbca
```

When the application launches, the default selection will be `Create a database`. That's exactly what we want, so click `Next`. We want to do the `Advanced configuration`, so select that option and click `Next`.

![](assets/images/new/009.PNG)

The next page has nothing we want to change, so click `Next` again.

Here is where we'll start changing settings. Set `Global database name` and `SID` to `swg`. Deselect `Create as Container database` and click `Next`.

![](assets/images/new/010.PNG)

The next four pages are fine, so click `Next` four more times.

When you get to the page with several tabs, we'll change a few things again. First, in the `Memory` tab, set your `SGA Size` to `3072` and your `PGA Size` to `1024`.

![](assets/images/new/013.PNG)

In the `Character sets` tab, change the `National character set` to `UTF8`. That's it here, so click `Next`.

![](assets/images/new/014.PNG)

Uncheck the `Enterprise Manager` option and click `Next`.

![](assets/images/new/015.PNG)

Select the option to use the same password for everything, and set this password to `swg`.

![](assets/images/new/016.PNG)

When you click `Next`, you'll get a warning that your password kind of sucks. That's fine, so click `Yes`.

![](assets/images/new/017.PNG)

On the next page, the default checkbox is `Create database`. That's good, so click `Next`.

Finally, you'll get your confirmation summary. Everything should be good, so click `Finish` to let the application create your database. This will take quite a while. You'll probably think it's stuck. It's not. Just give it time. When it's finally done, click `Close`.

You can finally close the terminal window you've been using for the Oracle installation.

After setting up the database, it will already be running in the background. However, we'll also want Oracle Database to load automatically whenever the VM is booted up. We can make that happen by starting a fresh terminal and running the following commands:

```
wget https://raw.githubusercontent.com/tekaohswg/oinit/master/install-service.sh
chmod +x install-service.sh
sudo ./install-service.sh
```

The last thing we have to do with Oracle is prepare it to be accessible by the SWG server. In a terminal, run the following commands to download a file we're going to need and then to open SQL Developer.:

```
wget https://raw.githubusercontent.com/tekaohswg/oinit/master/swgusr.sql
/u01/app/oracle/product/18/dbhome_1/sqldeveloper/sqldeveloper.sh
```

As SQL Developer loads, it will ask you if you want to import your preferences. Click `No`. After loading some more, it will ask you about reporting your usage to Oracle. Uncheck the box to opt out and click `OK`.

We want to add some new connections. Click the green plus icon on the left-hand side under `Connections`. First, enter these inputs and click `Save`.

```
Connection name: system@swg
Username: system
Password: swg
[x] Save Password
SID: swg
```

![](assets/images/new/012.PNG)

Create another connection with these inputs and click save again: (You won't actually use this connection as part of the tutorial, but you might find it nice to have later.)

```
Connection name: swg@swg
Username: swg
Password: swg
[x] Save Password
SID: swg
```

![](assets/images/new/020.PNG)

With both these connections created, close this window. Your two new connections will now appear on the list on the left-hand side. Double-click on `system@swg` to connect as `system`. Then, click on `File -> Open...` and in the file explorer, click `Home` and open the file `swgusr.sql` that you downloaded earlier. This will load a series of SQL statements into SQL Developer. Press `F5` to run the script, and click `OK` on the confirmation window to confirm that you want to use the `system@swg` connection.

![](assets/images/new/021.PNG)

After a moment, the script will be completed. You can now close SQL Developer.

Check back soon for instructions to install SWG! (Or just go ahead if you already know how)

We are finally ready to install Star Wars Galaxies! Go ahead and install your favorite SWG platform, if you have one, or keep reading to install the Tekaoh SWG platform.

```
sudo apt install git -y
git clone https://github.com/tekaohswg/swg.git
cd swg
./swgtool.sh install
```

The install script will need a little bit of user input to get started. Just enter your IP address and press `Enter`. Then enter a name for your galaxy and press `Enter` again. If the script asks for a sudo password, enter that as well. Once the script is done asking these questions, it will download everything you need and compile the source code. This will probably take a good long time.

After the compile stage, the script might ask for the sudo password again before building the chat server. It will also ask for input again when it wants to build the database. Use the following replies:
```
Enter value for max_characters_per_account: 10
Enter value for max_characters_per_cluster: 10
Enter value for character_slots: 10
Enter value for cluster_name: (Use whatever you named your cluster at the beginning of the installation.)
Enter value for host: 127.0.0.1
```

After the install script finally finishes, you can start your SWG server:
```
./swgtool.sh start
```

That's it! Congratulations on building your SWG server from scratch! I knew you could do it.
