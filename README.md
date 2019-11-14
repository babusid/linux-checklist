# Brentwood High School Linux Checklist
## Notes

**If a command errors or fails, try it again with `sudo` (or `sudo !!` to save typing)**

**Google anything and everything. If you don't know or understand something, google it**

When you see the syntax `$word`, do not type it verbatim, but instead substitute the appropriate word (usually referenced in a previous command).

When the order of steps does not matter, bullet points have been used instead of ordinals.

To edit files, run `gedit`, a graphical editor akin to notepad; `nano`, a simple 
command-line editor; or `vim`, a powerful  but less intuitive command-line editor. Note that vim may need to be installed with `apt-get install vim`.

## Checklist

1. Read the readme
	* **The readme is the highest authority on the image scenario. If it tells you to leave something, even if you believe it to be the worlds biggest security flaw, LEAVE IT. And if it tells you to remove something, even if it is incredibly beneficial, REMOVE IT.**
	
	* Note down which ports/users are allowed.
	
	* Note down which services are necessary/needed.
		* Some of the following steps are made under the assumption that those services are needed. **USE YOUR HEAD.** If it's not needed, these steps might **NOT APPLY**.

1. Install locate package, it makes finding things ridiculously easy.
	1. `sudo apt-get install locate`
	2. `locate [blank]`

1. **Do Forensics Questions**
	
	You may destroy the requisite information if you work on the checklist! Tip: The majority of the time, the forensics questions (at least one of them) will ask you to find a certain type of file, or a certain file on the system. To print the entirety of the filetree, and reveal every file as well as its path, use `sudo ls -aR`. Pipe the output of this to `grep""`and you will be able to find most any type of file, or specific file. (grep is essentially ctrl+F for the terminal)
	
	Ex: `sudo ls -aR|grep"example.mp3"`

	Ex: `sudo locate ".mp3"`

1. Secure ssh installation (if ssh is needed as it commonly is; if ssh is specified as not needed in readme **DELETE IT**, but it may be unspecified. **USE YOUR HEAD**)

	1. set `PermitRootLogin no` in `/etc/ssh/sshd_config` to disable root logins
	1. set `PermitEmptyPasswords no` in `/etc/ssh/sshd_config` to disable empty passwords
	1. set `PermitUserEnvironment no` in `/etc/ssh/sshd_config` 

1. Secure Users
	1. Disable the guest user.
	
		Go to `/etc/lightdm/lightdm.conf` and add the line
		
		`allow-guest=false`

		Then restart your session with `sudo restart lightdm`. This will log you out, so make sure you are not executing anything important.

	1. Open up `/etc/passwd` and check which users
		* Are uid 0
		* Can login
		* Are allowed in the readme
	1. Delete unauthorized users:
		
		`sudo userdel -r $user`

		`sudo groupdel $user`
	1. Check `/etc/sudoers.d` and make sure only members of group sudo can sudo.
	1. Check `/etc/group` and remove non-admins from sudo and admin groups.
	1. Check user directories.
		1. cd `/home`
		1. `sudo ls -aR *`
		1. Look in any directories which show up for media files/tools and/or "hacking tools."
	1. Enforce  Password Requirements.
		1. Add or change password expiration requirements to `/etc/login.defs`.
			
			```
			PASS_MIN_DAYS 7
			PASS_MAX_DAYS 90
			PASS_WARN_AGE 14
			```
		1. Add a minimum password length.
			1. Open `/etc/pam.d/common-password` with sudo.
			2. Add `minlen=8` to the end of the line that has `pam_unix.so` in it.
		1. Add a password history
			1. Open `/etc/pam.d/common-password` with sudo.
			2.Add `remember=5` to the end of the line that has `pam_unix.so` in it.
		2.Add a minimum password complexity
			1. Open `/etc/pam.d/common-password`.
			2. Locate the line that has pam.cracklib.so in it.
				If you cannot find that line, run sudo apt-get install libpam-cracklib to install cracklib, 				    close the file and open it again with sudo.
			3.Add `ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-` to the end of the line.
		1. Implement an account lockout policy.
			1. Open `/etc/pam.d/common-auth`.
			2. Add `deny=5 unlock_time=1800` to the end of the line with `pam_tally2.so` in it.
		1. Change all passwords to satisfy these requirements.
			
			Use `chpasswd` to change the passwords of accounts from the terminal, alternatively, you can change them individiually through **Settings-->Users**, unlocking the settings and then individually typing in new passwords for accounts where they do not meet complexity requirements
			
			***Note: You will have to manually change any account that has a weak password, however, these will only be part of the admin accounts. Check the given admin passwords and determine which ones are weak and change those first.***

1. Enable automatic updates
	
	In the GUI set Update Manager->Settings->Updates->Check for updates:->Daily.

1. Secure ports
	1. `sudo ss -ln`
	1. If a port has `127.0.0.1:$port` in its line, that means it's connected to loopback and isn't exposed. Otherwise, there should only be ports which are specified in the readme open (but there probably will be tons more).
	1. For each open port which should be closed:
		1. `sudo lsof -i :$port`
		1. Copy the program which is listening on the port.
		`whereis $program`
		1. Copy where the program is (if there is more than one location, just copy the first one).
		`dpkg -S $location`
		1. This shows which package provides the file (If there is no package, that means you can probably delete it with `rm $location; killall -9 $program`).
		`sudo apt-get purge $package`
		1. Check to make sure you aren't accidentally removing critical packages before hitting "y".
		1. `sudo ss -l` to make sure the port actually closed.

1. Secure network
	1. Enable the firewall  
	Install gufw (A graphical frontend for the pre-existing ubuntu firewall):  
	`sudo apt-get install gufw`  
	 If the system returns that gufw is already installed, run   
	 `sudo gufw`
	1. Enable syn cookie protection
	
		`sysctl -n net.ipv4.tcp_syncookies`

1. Install Updates

	Start this before half-way.

	* Do general updates.
		1. `sudo apt-get update`.
		1. `sudo apt-get upgrade`.

	* Update services specified in readme.
		1. Google to find what the latest stable version is.
		1. Google "ubuntu install service version".
		1. Follow the instructions.
	
	* Ensure that you have points for upgrading the kernel, each service specified in the readme, and bash if it is [vulnerable to shellshock](https://en.wikipedia.org/wiki/Shellshock_%28software_bug%29).
	

1. Configure services
	1. Check service configuration files for required services.
		Usually a wrong setting in a config file for sql, apache, etc. will be a point.
	1. Ensure all services are legitimate.
		
		`service --status-all`


1. Check the installed packages for "hacking tools," such as password crackers.

1. Run other (more comprehensive) checklists. This is checklist designed to get most of the common points, but it may not catch everything.
 
## Tips

* Netcat is installed by default in ubuntu. You will most likely not get points for removing this version.
* Some services (such as `ssh`) may be required even if they are not mentioned in the readme. Others may be points even if they are explicitly mentioned in the readme


## Acknowledgments
Sean "Forty-Bot" Anderson, whose checklist this was originally forked from and then iterated upon. (His checklist can be found here:https://github.com/Forty-Bot/linux-checklist)

[![Creative Commons License][image-1]][1]  
 This checklist is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License][1].
 
 [1]:    http://creativecommons.org/licenses/by-sa/4.0/
 
 [image-1]:    https://i.creativecommons.org/l/by-sa/4.0/88x31.png
