---
description: Last updated in 2018
---

# Bitcoin and Ethereum Wallet Cracking

## Ethereum Wallet Cracking

This guide will show you how an Ethereum wallet \(JSON keystore file\) is broken down, how it’s mapped to a hashcat compatible format, and finally how to crack the wallet to recover the private key.

### The Keystore File

For demonstrating this attack, I will generate a wallet using [myEtherWallet](https://www.myetherwallet.com/) and choose `P@ssw0rd1!` for a password. The generated JSON file will have a name in the following format: `UTC--<date_created>--<wallet_address>`. To learn about keysotre files and how your Ethereum private key is computed from your Ethereum keystore file read he article “[what is an Ethereum keystore file](https://medium.com/@julien.m./what-is-an-ethereum-keystore-file-86c8c5917b97)[?](https://medium.com/@julien.m./what-is-an-ethereum-keystore-file-86c8c5917b97)”. In a nutshell, the Ethereum keystore file is an encrypted version \(aes-128-ctr\) of your unique Ethereum private key that you will use to sign your transactions.

Fortunately we don't need to break AES in order to recover the private key; the keystore file includes a MAC \(SHA3-256\) computer from the XOR of the cyphertext \(private key\) and the decryption-key \(the password after applying the “Scrypt” Key Derivation Function\). If the MAC computed by Hashcat matches the one from the keystore file, bingo!

### ethereum2john.py

How to turn the keystore file into a Hashcat compatible format? This can be done by hand, but a python scrypt called [ethereum2john.py](https://github.com/magnumripper/JohnTheRipper/blob/bleeding-jumbo/run/ethereum2john.py) \(yes john, not hashcat\) makes the task much easier, lets take for example the keystore file that I've already created:

$ python ethereum2john.py ~/UTC--2018-04-19T09-34-23.783Z--9a475c94351710d82d1611fe9e81817cbaf5833c

UTC--2018-04-19T09-34-23.783Z--9a475c94351710d82d1611fe9e81817cbaf5833c:$ethereum$s\*8192\*8\*1\*f855b4083dd962a59f62eaca1b8af4c1557b11c982572c05a1466793d6fc3aab\*9b7411344eea9af8fa83d5684d4bec52fd30aeec99e5700a977eabea9c7b9e4f\*a53185c2660340eca77da5db3667fc72d07e42eefc647d59bcc04a73b2937a27

The string that we are interested in starts at `$ethereum$` until the end of the line. The following table shows the [example hashes](https://hashcat.net/wiki/doku.php?id=example_hashes) for Ethereum and their respective Hash-Mode numbers.

| Hash-Mode | Hash-Name | Example |
| :--- | :--- | :--- |
| 15600 | Ethereum Wallet, PBKDF2-HMAC-SHA256 | $ethereum$p\*262144\*3238383137313130353438343737383736323437353437383831373034343735\*06eae7ee0a4b9e8abc02c9990e3730827396e8531558ed15bb733faf12a44ce1\*e6d5891d4f199d31ec434fe25d9ecc2530716bc3b36d5bdbc1fab7685dda3946 |
| 15700 | Ethereum Wallet, SCRYPT | $ethereum$s\*262144\*1\*8\*3436383737333838313035343736303637353530323430373235343034363130\*8b58d9d15f579faba1cd13dd372faeb51718e7f70735de96f0bcb2ef4fb90278\*8de566b919e6825a65746e266226316c1add8d8c3d15f54640902437bcffc8c3 |

### Cracking with Hashcat

We can now start cracking by specifying the hash-mode \(using the flag **`-m15700`** for Ethereum\) followed by the output of ethereum2john:

Linux:

./hashcat64.bin -m 15700 -a 0 -w 3 -D1 --force /home/ubuntu/crackme.txt /home/ubuntu/tryme.txt -r rules/best64.rule --status --status-timer=10

./hashcat64.bin -m 15700 -a 0 -w 3 -D2 --gpu-temp-disable --force -s 40000 --scrypt-tmto=3 /home/ubuntu/crackme.txt /home/ubuntu/tryme.txt -r rules/OneRuleToRuleThemAll.rule --status --status-timer=30 --session=ethereum -o "ethereumsession.txt" --outfile-format=3

./hashcat64.bin -m 15700 -a 0 -w 3 -D1 --gpu-temp-disable --force --scrypt-tmto=1 /home/ubuntu/crackme.txt /home/ubuntu/words.txt --status --status-timer=30 -o "crackmeifyoucan" --outfile-format=3 -s 40000

Windows:

hashcat64.exe -m15700 $ethereum$s\*8192\*8\*1\*f855b4083dd962a59f62eaca1b8af4c1557b11c982572c05a1466793d6fc3aab\*9b7411344eea9af8fa83d5684d4bec52fd30aeec99e5700a977eabea9c7b9e4f\*a53185c2660340eca77da5db3667fc72d07e42eefc647d59bcc04a73b2937a27 C:\Users\cmedici785\Desktop\shared\_vm\opt\wordlists\fasttrack.txt -w3 -r rules/best64.rule

hashcat \(v4.0.1\) starting...

OpenCL Platform \#1: NVIDIA Corporation  
======================================  
\* Device \#1: Quadro M2000M, 1024/4096 MB allocatable, 5MCU

OpenCL Platform \#2: Intel\(R\) Corporation  
========================================  
\* Device \#2: Intel\(R\) HD Graphics 530, skipped.  
\* Device \#3: Intel\(R\) Core\(TM\) i7-6820HQ CPU @ 2.70GHz, skipped.

\[..SNIP..\]

Dictionary cache hit:  
\* Filename..: C:\Users\cmedici785\Desktop\shared\_vm\opt\wordlists\fasttrack.txt  
\* Passwords.: 222  
\* Bytes.....: 2006  
\* Keyspace..: 17094

\[s\]tatus \[p\]ause \[r\]esume \[b\]ypass \[c\]heckpoint \[q\]uit =&gt;

Approaching final keyspace - workload adjusted.

$ethereum$s\*8192\*8\*1\*f855b4083dd962a59f62eaca1b8af4c1557b11c982572c05a1466793d6fc3aab\*9b7411344eea9af8fa83d5684d4bec52fd30aeec99e5700a977eabea9c7b9e4f\*a53185c2660340eca77da5db3667fc72d07e42eefc647d59bcc04a73b2937a27:P@ssw0rd1!

Session..........: hashcat  
Status...........: Cracked  
Hash.Type........: Ethereum Wallet, SCRYPT  
Hash.Target......: $ethereum$s\*8192\*8\*1\*f855b4083dd962a59f62eaca1b8af4...937a27  
Time.Started.....: Thu Apr 19 16:48:23 2018 \(8 mins, 50 secs\)  
Time.Estimated...: Thu Apr 19 16:57:13 2018 \(0 secs\)  
Guess.Base.......: File \(C:\Users\cmedici785\Desktop\shared\_vm\opt\wordlists\fasttrack.txt\)  
Guess.Mod........: Rules \(rules/best64.rule\)  
Guess.Queue......: 1/1 \(100.00%\)  
Speed.Dev.\#1.....:        6 H/s \(858.00ms\)  
Recovered........: 1/1 \(100.00%\) Digests, 1/1 \(100.00%\) Salts  
Progress.........: 3085/17171 \(17.97%\)  
Rejected.........: 0/3085 \(0.00%\)  
Restore.Point....: 40/223 \(17.94%\)  
Candidates.\#1....: P@55w0rd -&gt; sqlsqlsqlsql  
HWMon.Dev.\#1.....: Temp: 49c Util:100% Core:1137MHz Mem:2505MHz Bus:16

As you can be seen above Hashcat is running at a mind-boggling speed of 6 hashes/second!

### CPU vs GPU

If Hashcat crashes or hangs your system, you will need to crack using a CPU. SCRYPT is an anti-GPU algorithm and depending on the SCRYPT parameters \(N, r and p\) there’s a fair chance you’ll have to resort to CPU cracking, for more details on this please refer to this excellent [blog post](https://stealthsploit.com/2018/01/04/ethereum-wallet-cracking-pt-2-gpu-vs-cpu/). In a nutshell, your CPU likely won't have the amount of RAM necessary to handle the required parallel computations, so you need to manually tell Hashcat to use your CPU. In order to do that, you first find out your devices' information by running **`hashcat -I`** which you can use to identify the CPU\(s\):

hashcat-4.0.1&gt;hashcat64.exe -I  
hashcat \(v4.0.1\) starting...

OpenCL Info:

Platform ID \#1  
  Vendor  : NVIDIA Corporation  
  Name    : NVIDIA CUDA  
  Version : OpenCL 1.2 CUDA 8.0.0

  Device ID \#1  
    Type           : GPU  
    Vendor ID      : 32  
    Vendor         : NVIDIA Corporation  
    Name           : Quadro M2000M  
    Version        : OpenCL 1.2 CUDA  
    Processor\(s\)   : 5  
    Clock          : 1137  
    Memory         : 1024/4096 MB allocatable  
    OpenCL Version : OpenCL C 1.2  
    Driver Version : 382.16

Platform ID \#2  
  Vendor  : Intel\(R\) Corporation  
  Name    : Intel\(R\) OpenCL  
  Version : OpenCL 2.1

  Device ID \#2  
    Type           : GPU  
    Vendor ID      : 8  
    Vendor         : Intel\(R\) Corporation  
    Name           : Intel\(R\) HD Graphics 530  
    Version        : OpenCL 2.1  
    Processor\(s\)   : 24  
    Clock          : 1050  
    Memory         : 2047/13055 MB allocatable  
    OpenCL Version : OpenCL C 2.0  
    Driver Version : 22.20.16.4836

  Device ID \#3  
    Type           : CPU  
    Vendor ID      : 8  
    Vendor         : Intel\(R\) Corporation  
    Name           : Intel\(R\) Core\(TM\) i7-6820HQ CPU @ 2.70GHz  
    Version        : OpenCL 2.1 \(Build 2\)  
    Processor\(s\)   : 8  
    Clock          : 2700  
    Memory         : 8164/32659 MB allocatable  
    OpenCL Version : OpenCL C 2.0  
    Driver Version : 7.5.0.2

The output above shows that we need to use the CPU with platform ID `#2` and device ID `#3` which we can specify in hashcat adding the flags **`-D 2 -d 3`** \(Upper D is for platform and lower d for device\). If you are using the [Hashcat GUI](https://hashkiller.co.uk/hashcat-gui.aspx), you can simply check the checkbox called “CPU only”.

Unfortunately this does not speed up the cracking in any way. I haven't found any ways to increase the cracking rate substantially, I can only force to use a second GPU by adding the **`--force`** flag, this way the number of Hashes/second add up:

Speed.Dev.\#1.....:        6 H/s \(859.43ms\)  
Speed.Dev.\#2.....:        4 H/s \(6101.20ms\)  
Speed.Dev.\#\*.....:        9 H/s

88048

#### Resources \(especially for setting up Intel OpenCL drivers\):

[https://hashcat.net/forum/thread-7293.html](https://hashcat.net/forum/thread-7293.html)  
[https://software.intel.com/en-us/articles/opencl-drivers](https://software.intel.com/en-us/articles/opencl-drivers)  
[https://hashcat.net/forum/thread-6079.html](https://hashcat.net/forum/thread-6079.html)  
[http://registrationcenter-download.intel.com/akdlm/irc\_nas/11396/SRB5.0\_intel-opencl-installation.pdf](http://registrationcenter-download.intel.com/akdlm/irc_nas/11396/SRB5.0_intel-opencl-installation.pdf) -- installation manual  
[https://hashcat.net/forum/archive/index.php?thread-6648-2.html](https://hashcat.net/forum/archive/index.php?thread-6648-2.html)  
[https://scottlinux.com/2017/01/31/how-to-use-hashcat-on-cpu-only/](https://scottlinux.com/2017/01/31/how-to-use-hashcat-on-cpu-only/)

#### Installing drivers - Metod 1 \(not recommended\)

wget [http://registrationcenter-download.intel.com/akdlm/irc\_nas/12556/opencl\_runtime\_16.1.2\_x64\_rh\_6.4.0.37.tgz](http://registrationcenter-download.intel.com/akdlm/irc_nas/12556/opencl_runtime_16.1.2_x64_rh_6.4.0.37.tgz)  
tar -xzvf opencl\_runtime\_16.1.2\_x64\_rh\_6.4.0.37.tgz  
cd opencl\_runtime\_16.1.2\_x64\_rh\_6.4.0.37/  
./install.sh  
wget [https://hashcat.net/files/hashcat-4.1.0.7z](https://hashcat.net/files/hashcat-4.1.0.7z)  
7z x hashcat-4.1.0.7z

[Installing drivers - Method 2 \(Recommended\)](Cybersecurity--Vulnerabilities--Wallets--Bitcoin-Litecoin--Wallet_cracking--automate_install_intel_CPU_drivers.html)  
link above..

## Bitcoin/Litecoin

The wallet format for Bitcoin and Litecoin is identical, they are both stored in a file called wallet.dat which we can use to crack the passphrase used to protect the user's private key.

### Wallet.dat

The original Bitcoin client stores private key information in a file named **wallet.dat** following the so called ["bitkeys"](https://bitcointalk.org/index.php?topic=4448.0) format.

The wallet.dat file is located in the [Bitcoin data directory](https://en.bitcoin.it/wiki/Data_directory) and may be [encrypted with a password](https://en.bitcoin.it/wiki/Wallet_encryption).

The format of this file is Berkeley DB. Tools that can manipulate wallet files include [pywallet](https://en.bitcoin.it/wiki/Pywallet).

### Bitcoin2john.py

In order to convert a wallet.dat file into a string of text usable by hashcat we need to use [bitcoin2john.py](https://github.com/magnumripper/JohnTheRipper/blob/bleeding-jumbo/run/bitcoin2john.py), which is based on jackjack's [pywallet](https://en.bitcoin.it/wiki/Pywallet).

The usage is very simple:

python bitcoin2john.py wallet.dat  
$bitcoin$................

Take the line that starts with `$bitcoin` and place it in a file called `hash.txt` in the working directory.

The format will be like the one below, taken from the hashcat [example hashes](https://hashcat.net/wiki/doku.php?id=example_hashes) page:

| Hash-Mode | Hash-Name | Example |
| :--- | :--- | :--- |
| 11300 | Bitcoin/Litecoin wallet.dat | $bitcoin$96$d011a1b6a8d675b7a36d0cd2efaca32a9f8dc1d57d6d01a58399ea04e703e8bbb44899039326f7a00f171a7bbc854a54$16$1563277210780230$158555$96$628835426818227243334570448571536352510740823233055715845322741625407685873076027233865346542174$66$625882875480513751851333441623702852811440775888122046360561760525 |

NOTE: Cracking this hash will not allow them to access your Bitcoins unless they also have access to your wallet.dat file.

First, we are going to run a straight-up dictionary attack. This means that password has to be found in your wordlist exactly - with correct case, special characters, etc.

\# Try it this way first, with some hardware optimization parameters:  
hashcat -a 0 -m 11300 ./hash.txt ./wordlist.txt -O -w 3

\# If that doesn't work, try this:  
hashcat -a 0 -m 11300 ./hash.txt ./wordlist.txt

Press the `S` key at any time to see that status of your cracking session.

### Creating more work for full speed

If hashcat is running really slow it might be because it is not making full use of the parallel processing power of your GPU, for detailed information please read [this hashcat FAQ](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_to_create_more_work_for_full_speed). In short, after applying the fix suggested in the FAQ hashcat went from 20 to 40,000 hashes per minute! The trick is simply to feed Hashcat base words through a pipe, gor example:

hashcat64.exe --stdout wordlist.txt -r rules/best64.rule \| hashcat64.exe -a 0 -m 11300 -w 3 btcHash.txt

Since these are slow hashes and will take a long time to crack, I recommend adding the following parameters to save the session for another time in case of interruption:

hashcat64.exe --stdout wordlist.txt -r rules\OneRuleToRuleThemAll.rule \| hashcat64.exe -a 0 -m 11300 --session=ltcwallet -w 3 -o "C:\Users\foobar\sessionfile.txt" --outfile-format=3 -D 2 -d 3 $bitcoin$96$................

## decrypting wallet

`openssl enc -d -aes-256-cbc -in /path/to/wallet.dat -out ~/wallet-decrypt.dat`

## How to create more work for full speed?

This is a really important topic when working with Hashcat. Let me explain how Hashcat works internally, and why this is so important to understand.

GPUs are not magic superfast compute devices that are thousands of times faster than CPUs – actually, GPUs are quite slow and dumb compared to CPUs. If they weren't, we would even use CPUs anymore; CPUs would simply be replaced with GPU architectures. What makes GPUs fast is the fact that there are thousands of slow, dumb cores \(shaders.\) This means that **in order to make full use of a GPU, we have to parallelize the workload so that each of those slow, dumb cores have enough work to do.** Password cracking is what is known as an “embarrassingly parallel problem” so it is easy to parallelize, but we still have to structure the attack \(both internally and externally\) to make it amenable to acceleration.

For most hash algorithms \(with the exception of very slow hash algorithms\), it is not sufficient to simply send the GPU a list of password candidates to hash. Generating candidates on the host computer and transferring them to the GPU for hashing is an order of magnitude slower than just hashing on the host directly, due to PCI-e bandwidth and host-device transfer latency \(the PCI-e copy process takes longer than the actual hashing process.\) To solve this problem, we need some sort of workload amplifier to ensure there's enough work available for our GPUs. In the case of password cracking, generating password candidates on the GPU provides precisely the sort of amplification we need. In Hashcat, we accomplish this by splitting attacks up into two loops: a “base loop”, and a “mod\(ifier\) loop.” The base loop is executed on the host computer and contains the initial password candidates \(the “base words.”\) The mod loop is executed on the GPU, and generates the final password candidates from the base words on the GPU directly. The mod loop is our amplifier – this is the source of our GPU acceleration.

What happens in the mod loop depends on the attack mode. For brute force, a portion of the mask is calculated in the base loop, while the remaining portion of the mask is calculated in the mod loop. For straight mode, words from the wordlist comprise the base loop, while rules are applied in the mod loop \(the on-GPU rule engine that executes in the mod loop is our amplifier.\) For hybrid modes, words from the wordlist comprise the base loop, while the brute force mask is processed in the mod loop \(generating each mask and appending it to base words is our amplifier.\)

Without the amplifier, there is no GPU acceleration for fast hashes. If the base or mod loop keyspace is too small, you will not get full GPU acceleration. So the trick is providing enough work for full GPU acceleration, while not providing too much work that the job will never complete. For straight mode against fast hashes, your wordlist should have at least 10 million words and you should supply at least 1000 rules.

Now, we mentioned above that this advice is for most hash algorithms, with the exception of very slow hash algorithms. Slow hash algorithms use some variety of compute-hardening techniques to make the hash computation more resource-intensive and more time-consuming. For slow hash algorithms, we do not need \(nor oftentimes do we want\) an amplifier to keep the GPU busy, as the GPU will be busy enough with computing the hashes. Using attacks without amplifiers often provide the best efficiency.

Because we are very limited in the number of guesses we can make with slow hashes, you're often working with very small, highly targeted wordlists. However, sometimes this can have an inverse effect and result in a wordlist being too small to create enough parallelism for the GPU. There are two solutions for this:

Use rules, but not as an amplifier. Basically this means you feed Hashcat base words through a pipe:  
$ ./hashcat64.bin --stdout wordlist.txt -r rules/best64.rule \| ./hashcat64.bin -m 2500 test.hccapx  
Use princeprocessor. Same processes over a pipe:  
$ ./pp64.bin wordlist.txt \| ./hashcat64.bin -m 2500 test.hccapx  
Sometimes it can make sense to use maskprocessor. Note this should be used only for very small keyspaces!  
$ ./mp64.bin ?d?d?d?d \| ./hashcat64.bin -m 2500 test.hccapx  
Note: pipes work in Windows the same as they do in Linux.

Those attack modes are usually already built into Hashcat, so why should we use a pipe? The reason is, as explained above, masks are split in half internally in Hashcat, with one half being processed in the base loop, and the other half processed in the mod loop, in order to make use of the amplification technique. But this reduces the number of base words, and for small keyspaces, reduces our parallelism, thus resulting in reduced performance.

Is piping a wordlist slower than reading from file?  
No, piping is usually equally fast.

However, most candidate generators are not fast enough for hashcat. For fast hashes such as MD5, it is crucial to expand the candidates on the GPU with rules or masks in order to achieve full acceleration. However be aware that different rulesets are not producing constant speeds. Especially big rulesets can lead to a significant speed decrease. The increase from using rules as amplifier can therefor cancel itself out depending how complicated the rules are.

Why is my attack so slow?  
To find out about what maximum speeds you can expect from your system, run a benchmark How can I perform a benchmark? Note: Benchmarks are a “best case” scenario, i.e. single hash brute force. Real-world speed can vary depending on the number of hashes and attack mode.  
In most of the cases of “slow speeds” you simply did not create enough work for hashcat. Read How to create more work for full speed?  
You can add more pressure on the GPU using the -w 3 parameter. Note: this will cause your desktop to lag because the GPU is so busy it can not compute desktop changes like mouse movement.  
Your GPUs are overheating. If this happens \(typically around 90c\) the GPU bios automatically downclocks the GPU = slower speed  
The more hashes are in your hashlist, the slower the speed gets. The biggest difference is between one or more hashes because for single hashes hashcat can use special optimitations which only can be used when cracking just a single hash  
Some hashes are designed to run slow, like bcrypt, scrypt or bitcoin wallet. Deal with it.  
Why does hashcat says it has only 2% GPU utilization?  
Not having 100% GPU utilization is usually an indicator for a too-small keyspace \(meaning, not enough work to be done\).

Read How to create more work for full speed?

How is it possible that hashcat does not utilize all GPUs?  
If the number of base-words is so small that it is smaller than the GPU power of a GPU, then there is simply no work left that a second, or a third, or a fourth GPU could handle.

Read How to create more work for full speed?

## Blockchain, My Wallet, V1, V2

### Hashcat

| click me | click me | click me |
| :--- | :--- | :--- |
| 12700 | Blockchain, My Wallet | $blockchain$288$5420055827231730710301348670802335e45a6f5f631113cb1148a6e96ce645ac69881625a115fd35256636d0908217182f89bdd53256a764e3552d3bfe68624f4f89bb6de60687ff1ebb3cbf4e253ee3bea0fe9d12d6e8325ddc48cc924666dc017024101b7dfb96f1f45cfcf642c45c83228fe656b2f88897ced2984860bf322c6a89616f6ea5800aadc4b293ddd46940b3171a40e0cca86f66f0d4a487aa3a1beb82569740d3bc90bc1cb6b4a11bc6f0e058432cc193cb6f41e60959d03a84e90f38e54ba106fb7e2bfe58ce39e0397231f7c53a4ed4fd8d2e886de75d2475cc8fdc30bf07843ed6e3513e218e0bb75c04649f053a115267098251fd0079272ec023162505725cc681d8be12507c2d3e1c9520674c68428df1739944b8ac |

| click me | click me | click me |
| :--- | :--- | :--- |
| 15200 | Blockchain, My Wallet, V2 | $blockchain$v2$5000$288$06063152445005516247820607861028813ccf6dcc5793dc0c7a82dcd604c5c3e8d91bea9531e628c2027c56328380c87356f86ae88968f179c366da9f0f11b09492cea4f4d591493a06b2ba9647faee437c2f2c0caaec9ec795026af51bfa68fc713eaac522431da8045cc6199695556fc2918ceaaabbe096f48876f81ddbbc20bec9209c6c7bc06f24097a0e9a656047ea0f90a2a2f28adfb349a9cd13852a452741e2a607dae0733851a19a670513bcf8f2070f30b115f8bcb56be2625e15139f2a357cf49d72b1c81c18b24c7485ad8af1e1a8db0dc04d906935d7475e1d3757aba32428fdc135fee63f40b16a5ea701766026066fb9fb17166a53aa2b1b5c10b65bfe685dce6962442ece2b526890bcecdeadffbac95c3e3ad32ba57c9e |

## passphrases

Most wallet generators now enforce minimum passsword length of 9 characters, and educate users about using passphrases instead of passwrds. If you are cracking a wallet “for a friend” but have no idea about the passphrase length/charset, you should try first the wordlist provided by [passphrase-cracker](https://github.com/initstring/passphrase-cracker): “This is a project to build a massive wordlist using passphrases for cracking password hashes. Most of the wordlists I know of \(rockyou, exploitin, crackstation, etc\) contain single-word passwords. People are getting smarter and using phrases instead...I'd recommend combining with Hashcat rules like [Hob0](https://github.com/praetorian-inc/Hob0Rules) or [OneRule](https://github.com/NotSoSecure/password_cracking_rules).”

## Automation script to install intel CPU drivers

`sudo apt-get update  
sudo apt-get install unzip  
wget http://registrationcenter-download.intel.com/akdlm/irc_nas/11396/SRB5.0_linux64.zip  
unzip SRB*  
mkdir intel-opencl  
tar -C intel-opencl -Jxf intel-opencl-r5.0-63503.x86_64.tar.xz  
tar -C intel-opencl -Jxf intel-opencl-devel-r5.0-63503.x86_64.tar.xz  
tar -C intel-opencl -Jxf intel-opencl-cpu-r5.0-63503.x86_64.tar.xz  
sudo cp -R intel-opencl/* /  
sudo ldconfig  
sudo apt install p7zip-full  
wget https://hashcat.net/files/hashcat-4.1.0.7z  
7z x hashcat-4.1.0.7z  
cd hashcat-4.1.0/rules/  
wget https://raw.githubusercontent.com/NotSoSecure/password_cracking_rules/master/OneRuleToRuleThemAll.rule`  


