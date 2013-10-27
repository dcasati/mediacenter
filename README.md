FreeBSD as a Media Center
=========================

The idea in a nutshell:
------------------------------------------------------------------------------
To have a Plex media server running on FreeBSD and streaming to a Raspberry Pi. 


The building blocks:
------------------------------------------------------------------------------
FreeBSD is known from it's stability, cutting-edge features and rock-solid
performance. We will make use of all of them. 

FreeBSD supports an operating system-level virtualization mechanism known as
Jails, which aims into creating a segregated instance of FreeBSD. 

Plex is a popular media center solution available to many OSes, but up until
recently it wasn't available to any of the BSDs. Now you can enjoy the 
capabilities of Plex on a FreeBSD 9.1 or newer running on amd64. This is a 
very interesting idea as one can use the some of FreeBSD's powerful features
such as ZFS and jails to run this solution. 

Raspberry Pi is a popular small amr based computer that will be used as a
client, streaming the content from the Plex Server running on a FreeBSD Jail.
The interesting side of the Raspberry Pi are it's low price, low power
consumption, ethernet and HDMI support.


Creating the environment:
------------------------------------------------------------------------------
The inital concept involves setting up a Jail and from there install the Plex
media server. A handy tool to install and manage Jails is ezjail that can be
installed from the ports system. A more detailed and friendly version, from which 
this document was based, can be found on the tutorial notes from the BSD Now 
Podcast at http://www.bsdnow.tv/tutorials/jails

# cd /usr/ports/sysutils/ezjail
# make install clean

After the install, enable it by appending the string ezjail_enable=YES to
/etc/rc.conf. After that proceed with the following set of commands in order to
create the plexserver Jail.

# ezjail-admin install 

When creating a Jail, ezjail allows you to bind an IP address to a physical
interface. On the example server we have three interfaces, em0, em1 and xl0. We
will bind the new Jail to the em1 interface using the IP address 10.5.11.20.

# ezjail-admin create plexserver 'em1|10.5.11.20'

Once the Jail gets created we need to start it: 

# ezjail-admin start plexserver

If everything worked you can verify that the Jail is running by using the 
command jls or ezjail-admin list. The final touches will involve enabling
access to the Internet from this Jail. The reason being that the Plex server
will fetch information such as movie covers, imdb descriptions and other meta
tags. Copy of create your resolv.conf to the Jail directory (in this setup this
directory lies at /usr/jails/plexmediaserver).

# cp /etc/resolv.conf /usr/jails/plexserver/etc/

As an extra requirement we will enable the OpenSSH server on this Jail. You can do
this from the Jail console itself:

root@plexmediaserver:/root # ezjail-admin console plexserver
root@plexmediaserver:/root # grep ssh /etc/defaults/rc.conf >> /etc/rc.conf

After this you must edit the /etc/rc.conf and enable the sshd server by changing
sshd_enable="NO" to sshd_enable="YES". You can now exit from the Jail by
issuing 'exit'.


Installing the Plex Media Server
------------------------------------------------------------------------------
We will make use of the ports tree in order to install the Plex Media Server.
The following ezjail command will install a copy of the tree that can then be
used by plexserver Jail:

# ezjail-admin install -P
With the ports tree now in place we can log back into the plexserver Jail and
install it.

# ezjail-admin console plexserver
root@plexserver:/root # cd /usr/ports/multimedia/plexmediaserver/
root@plexserver:/usr/ports/multimedia/plexmediaserver # make install clean

And finally enable and start the server:

root@plexserver:/root # echo 'plexmediaserver_enable="YES" >> /etc/rc.conf
root@plexserver:/root # /usr/local/etc/rc.d/plexmediaserver start

The server should now be accessible on: http://10.5.11.5:32400/web/index.html


Configuring and adding media to the server
------------------------------------------------------------------------------
You should now share with the Jail your media files, which could be done in a
plethora of ways such as by mounting external hard drives or sharing files
sitting on a ZFS zpool. The rest of the Plex Serverr configuration can be done
entirelly via it's web interface on http://10.5.11.5:32400/web/index.html. For
more information please visit http://www.plexapp.com/


Configuring the Plex Player
------------------------------------------------------------------------------
For the client side we will be using a Linux distribution called Rasplex that
can be found at raplex.com.

In order to have an enjoyable experience, make sure you have a class 10+ SD
card. The performance penalty for running anything below that could ruin this
experience. If connecting the Raspberry Pi to a TV set with HDMI you can use
the TV's remote control to browse and control the Plex interface. Also, if using
a USB wifi dongle, check first if it is fully supported under Linux. A good list
can be fetched here:
http://elinux.org/RPi_VerifiedPeripherals

There are a few options to download it on a Mac OSX/Linux and Windows. The
instructions are fairly straight forward and they can be found at
http://rasplex.com/get-started/download-rasplex.html

Another possibility is to install it manually by grabbing the file at:
http://sourceforge.net/projects/rasplex/files/release/. At this time I'm using
the file rasplex-0.3.1.img.gz

$ md5 rasplex-0.3.1.img.gz
MD5 (rasplex-0.3.1.img.gz) = dba9a3f124225aaf9775ca63ecccce0b

After downloading the img.gz file you need to burn it to the SD card, there are
many ways of doing this such as by using dd. Once that step is done you can
hook the raspberry pi to your TV.

In general, at this stage, you should be set as the Plex Player running on the
Raspberry  Pi should be able to detect the presence of a Plex server on your
network. If this however is not your case, you can configure the Raspberry Pi
to look for the Plex Server by navigating into the Settings section and then
into the Manage server list menu. For more information please consult the Plex
Support at http://www.plexapp.com/help/

At this point you should be able to enjoy your new media center setup! This
setup could be expanded or changed on both, the client side by using different 
Plex Players (iOS devices, Android, Smart TVs) or on the server side by installing 
extra components such as the firefly media server that could serve music to your iTunes.
