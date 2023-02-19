# Sliver Snap

**Sliver** is an open source cross-platform adversary emulation/red team framework, it can be used by organizations of all sizes to perform security testing. Sliver's implants support C2 over Mutual TLS (mTLS), WireGuard, HTTP(S), and DNS and are dynamically compiled with per-binary asymmetric encryption keys.

### Features

* Dynamic code generation
* Compile-time obfuscation
* Multiplayer-mode
* Staged and Stageless payloads
* [Procedurally generated C2](https://github.com/BishopFox/sliver/wiki/HTTP(S)-C2#under-the-hood) over HTTP(S)
* [DNS canary](https://github.com/BishopFox/sliver/wiki/DNS-C2#dns-canaries) blue team detection
* [Secure C2](https://github.com/BishopFox/sliver/wiki/Transport-Encryption) over mTLS, WireGuard, HTTP(S), and DNS
* Fully scriptable using [JavaScript/TypeScript](https://github.com/moloch--/sliver-script) or [Python](https://github.com/moloch--/sliver-py)
* Windows process migration, process injection, user token manipulation, etc.
* Let's Encrypt integration
* In-memory .NET assembly execution
* COFF/BOF in-memory loader
* TCP and named pipe pivots
* Much more!

***

## About the Snap
This is the best and most advanced snap I've developed so far. The download size of **Sliver** snap is ~129MB and the installed size is ~200MB, *including* MinGW compiler! While the Kali Linux provided **apt** package comes with a download size of 170MB and installed size of 250MB, ***excluding*** MinGW compiler(which is more than 1GB in size!)! Also, snap version of **Sliver** is more advanced when it comes to user security, ease of usage and configuration(which I'll discuss shortly) than the official release. I enjoyed building this snap and hope you'll have fun using it. Happy Hacking!

> **Note:-** This'll be my last public snap release as I've decided to quit building snaps due to lack of appreciation and feedback from the community, despite getting thousands of installs worldwide! I think I could've done better if I've invested the time in some other lucrative skill, but still *a skill learnt is a talent earned.* 🙂


![image](https://user-images.githubusercontent.com/83544797/219951854-d2558612-e0c6-4423-a613-8801debb931d.png)

***

## Installation
To install Sliver snap, use the following command:-
```
snap install --edge sliver
```
> **Note:-** The initial release of Sliver snap will be in the edge channel, which is why you need the `--edge` flag.


## Usage
There are two commands available in **Sliver** snap, i.e. `sliver` and `sliver.server` for client and server respectively. By default, the `sliver.server` is run using **root** user. **Sliver** only allows the config files created by the ***user running the server*** to be used by other users. As snaps cannot access anything outside the user's HOME folder, we'll use **root**/**sudo** privileges to create a config file inside `/root` folder. 



The first time you run `sliver` command as a **normal** user, you get an error due to no config file generated for the user. You can generate a config file and import it to start `sliver`. Use the following commands:-

### Creating the config file
```
sudo sliver.server operator --name $USER --lhost localhost --lport 31337 --save /root/$USER.cfg
````

> **Note:-** If the user doesn't have **sudo** privileges, then use a root shell.

### Importing the config file
```
sudo cp /root/$USER.cfg .; sudo chown $USER ./$USER.cfg; sliver import $USER.cfg
```

![image](https://user-images.githubusercontent.com/83544797/219951929-f6c8011e-288f-4f8c-83ee-04d10585fdb0.png)

> **Note:-** I can get permissions to write the config files inside every user's HOME folder, which is what the official **Sliver** install script does using **root** privileges, but I choose not to as it'd be a breach of user's privacy. Btw, it's a good practice to know how to create config files. Also, I'm going to discuss a better alternative shortly where we'll run **Sliver** server without **root** privilege.

### Sliver Server
You can run the **Sliver** server as a normal user using the following command:-
```
sliver.server
```

![image](https://user-images.githubusercontent.com/83544797/219951957-b939a08e-bfbb-442c-8eea-4af0b0a978ce.png)

### Managing the Sliver service
The **Sliver** snap comes with it's own built-in service. To show all running snap services use the following command:-
```
snap services
```


![image](https://user-images.githubusercontent.com/83544797/219951992-da27e909-9d59-4c65-975b-ec4c8f0b5c76.png)

You can stop the service using the following command:-
```
sudo snap stop sliver.service
```
To disable **Sliver** service from running at startup, use the following command:-
```
sudo snap stop --disable sliver.service
```
To Start and Enable **Sliver** service, use the following command:-
```
sudo snap start --enable sliver.service
```


![image](https://user-images.githubusercontent.com/83544797/219952017-68aa901f-b388-4546-9fcc-8aac86d1278e.png)

### Managing Sliver configuration
Currently, there is only one configuration option available to users through which they can set the `SLIVER_DARWIN_CC_64` [sliver environment variable](https://github.com/BishopFox/sliver/wiki/Environment-Variables). If you want to add your own environment variable, then you can open a PR submitting the code. Adding a snap configuration option will require basic **Bash** scripting knowledge and to get started you can read [this guide](https://snapcraft.io/docs/adding-snap-configuration).



![image](https://user-images.githubusercontent.com/83544797/219952051-e724bca9-64cf-4162-84cb-79d22da51ee9.png)

As you can see from the image above, the `darwin/amd64` variable is set to an arbitrary value suggesting a need for change. The `windows/386` and  `windows/amd64` compilers are built into the snap and I didn't add any configuration for it. Also, as mentioned in [official wiki](https://github.com/BishopFox/sliver/wiki/Environment-Variables) due to license restrictions I couldn't include the compiler inside the snap. So, to get the existing config use the following command:-
```
sudo snap get sliver
```
To set a configuration option, in this case the `SLIVER_DARWIN_CC_64` environment variable use the following command:-
```
sudo snap set sliver darwin-cc=/home/phoenix/osxcross/target/bin/amd64-apple-darwin21.2-clang`
```



![image](https://user-images.githubusercontent.com/83544797/219952085-f842169d-87be-4f85-86ad-7c13001d1228.png)


Note, even if you change the config option outside the **Sliver** session, the variable `darwin/amd64` will get updated immediately inside an existing **Sliver** session.


![image](https://user-images.githubusercontent.com/83544797/219952109-130e6e20-7dc8-4aba-ad5a-788fae8385f9.png)

> **Note:-** As this snap doesn't have access outside user's HOME folder, the path to the compiler should be both inside the user's HOME folder and not inside a hidden(starting with a `.`) folder. Also, there is no environment variable mentioned in official Sliver wiki for  `darwin/arm64` compiler. I think the wiki needs updating.



## Using Sliver snap without root(Recommended)
If you're running **Sliver** in a cloud server for remote access, then running the `sliver-server` as **root** can be risky in case a potential vulnerability is found in **Sliver**. Which is why, I recommend running **Sliver** as a normal user and inside a security confinement. By default, the **Sliver** snap runs on port 31337 with **root** privilege(although still within a security confinement). But, you can start `sliver.server` on a different port using a normal user without **sudo**/**root** privileges. To do this run the following command as a normal user:-
```
sliver.server daemon --lhost 10.10.10.1 --lport 1337 &
```

> **Note:-** Basic unix restrictions for privileged port access still applies to snaps and to open a port below 1024 you'll need sudo/root privilege.

![image](https://user-images.githubusercontent.com/83544797/219952154-9323c3ce-7664-4692-b867-0a986f95d588.png)

As you can see, now `sliver-server` is running on ports 1337 and 31337. Now, let's create a configuration without **sudo**/**root** privilege and import it. 
```
sliver.server operator --name phoenix --lhost 10.10.10.1 --lport 1337 --save .
sliver import phoenix_10.10.10.1.cfg
```

![image](https://user-images.githubusercontent.com/83544797/219952185-012c2177-58f4-454f-a862-c737860b072d.png)

As you can see in the image above **Sliver** now works without **root** and full multiplayer mode support. Now, you can completely disable the `sliver-server` running on port 31337 with the following command:-
```
sudo snap stop --disable sliver.service
```
## Metasploit Framework Integration

As Sliver snap can't read outside user's HOME folder, it can't access the **Metasploit Framework** installed via `apt`. But you can install **Metasploit Framework** snap provided by me to let Sliver access the Metasploit commands. The question maybe how **Sliver** can access **Metasploit Framework** snap when it can't read outside HOME folder? Well, Snap Developers are provided [a facility](https://snapcraft.io/docs/content-interface#heading--dev-details) where they can make snaps automatically talk to each other, if they're are the publishers of both the snaps. As I maintain both **Metasploit Framwork** and **Sliver** snaps, I was able to make this happen. 🙂

![image](https://user-images.githubusercontent.com/83544797/219954892-4a4ca416-8d53-4b9c-a02a-9a78bea5b76d.png)


***
## About this Repo
This repo provides the necessary files to create a snap package for [Sliver](https://github.com/BishopFox/sliver). It'll work on Debian, Fedora, Ubuntu and all other major Linux distributions that have snapd installed.


&nbsp;
&nbsp;
&nbsp;
&nbsp;
***
**Snap created for Linux** ~~with love~~ with a keyboard by **Jitendra Patro**
