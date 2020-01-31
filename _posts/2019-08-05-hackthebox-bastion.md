---
layout: single
title: 'HackTheBox: Bastion'
date: 2019-08-05T16:16:05-04:00
categories:
  - HackTheBox
---
## Recon

### **Port scan**

<img class="alignnone wp-image-73" src="http://dustinwatts.me/wp-content/uploads/2019/07/2019-07-23_16h26_58-300x97.png" alt="" width="662" height="214" srcset="http://dustinwatts.me/wp-content/uploads/2019/07/2019-07-23_16h26_58-300x97.png 300w, http://dustinwatts.me/wp-content/uploads/2019/07/2019-07-23_16h26_58.png 722w" sizes="(max-width: 662px) 100vw, 662px" /> 

Jumping into this box with a basic NMAP scan shows us a couple of interesting things. First, the SSH server, but that&#8217;s pretty normal for HTB boxes. I tried to log into it with anonymous credentials, but that didn&#8217;t work out. The most important thing to pick up on from the scan is that it&#8217;s likely a Windows machine with NetBIOS and SMB shares, due to the ports 135,139, and 445. Port 135 had several well-known vulnerabilities plague it over the years, so maybe it could be vulnerable to one on this box. Also, there have been vulnerabilities for SMB, like the famous EternalBlue. So we could run some exploit tests against those ports just in case&#8230;

&nbsp;

## Initial Foothold

### **Exploit Checks**

I didn&#8217;t find anything useful for RPC exploits, but did find some for the EternalBlue SMB exploit.

<img class="alignnone wp-image-79" src="http://dustinwatts.me/wp-content/uploads/2019/07/search_eternalblue-300x199.png" alt="" width="660" height="438" srcset="http://dustinwatts.me/wp-content/uploads/2019/07/search_eternalblue-300x199.png 300w, http://dustinwatts.me/wp-content/uploads/2019/07/search_eternalblue.png 735w" sizes="(max-width: 660px) 100vw, 660px" /> 

Then I made another search, this time on &#8220;ms17-010&#8221;, the vulnerability CVE code.

<img class="alignnone wp-image-81" src="http://dustinwatts.me/wp-content/uploads/2019/08/search_ms17-010-300x200.png" alt="" width="657" height="438" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/search_ms17-010-300x200.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/search_ms17-010.png 736w" sizes="(max-width: 657px) 100vw, 657px" /> 

Then I ran the exploit &#8230;

<img class="alignnone wp-image-82" src="http://dustinwatts.me/wp-content/uploads/2019/08/run_ms17-010-300x58.png" alt="" width="657" height="127" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/run_ms17-010-300x58.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/run_ms17-010.png 736w" sizes="(max-width: 657px) 100vw, 657px" /> 

But no dice&#8230; the exploit path is probably a bust, so I moved on from wasting any more time on that approach.

&nbsp;

### SMB Guest Access

Then I tried the easy thing that should have been done first: connect to SMB without credentials.

<img class="alignnone wp-image-84" src="http://dustinwatts.me/wp-content/uploads/2019/08/smb_fileexplorer-300x209.png" alt="" width="657" height="458" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/smb_fileexplorer-300x209.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/smb_fileexplorer-768x534.png 768w, http://dustinwatts.me/wp-content/uploads/2019/08/smb_fileexplorer.png 850w" sizes="(max-width: 657px) 100vw, 657px" /> 

I tried first connecting with the file explorer in kali, but it wouldn&#8217;t accept a connection without credentials. So I tried smbmap next.

<img class="alignnone wp-image-85" src="http://dustinwatts.me/wp-content/uploads/2019/08/smb_smbmap-300x200.png" alt="" width="668" height="445" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/smb_smbmap-300x200.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/smb_smbmap.png 736w" sizes="(max-width: 668px) 100vw, 668px" /> 

That also didn&#8217;t work, but maybe I was just using it wrong. Either way, I moved on to trying smbclient.

<img class="alignnone wp-image-86" src="http://dustinwatts.me/wp-content/uploads/2019/08/smb_smbclient-300x199.png" alt="" width="672" height="446" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/smb_smbclient-300x199.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/smb_smbclient.png 736w" sizes="(max-width: 672px) 100vw, 672px" /> 

Finally I was getting somewhere, I could see the shares listed on the SMB service!

So then I used smbclient to log in and went exploring&#8230;

<img class="alignnone wp-image-87" src="http://dustinwatts.me/wp-content/uploads/2019/08/smb_explore-300x199.png" alt="" width="677" height="449" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/smb_explore-300x199.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/smb_explore.png 737w" sizes="(max-width: 677px) 100vw, 677px" /> 

After looking at all the folders, the only interesting one is WindowsImageBackup. Also there is a warning about downloading the entire backup file:

<img class="alignnone wp-image-88" src="http://dustinwatts.me/wp-content/uploads/2019/08/smb_warning-300x199.png" alt="" width="681" height="452" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/smb_warning-300x199.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/smb_warning.png 737w" sizes="(max-width: 681px) 100vw, 681px" /> 

After digging into the &#8220;Backups&#8221; share, I found a WindowsImageBackup with two VHD files.

<img class="alignnone wp-image-89" src="http://dustinwatts.me/wp-content/uploads/2019/08/backup_vhd_listing-300x201.png" alt="" width="681" height="456" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/backup_vhd_listing-300x201.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/backup_vhd_listing-768x514.png 768w, http://dustinwatts.me/wp-content/uploads/2019/08/backup_vhd_listing.png 835w" sizes="(max-width: 681px) 100vw, 681px" /> 

I don&#8217;t often listen to signs well, so I proceeded to download the VHDs anyway, but it disconnected the session pretty quickly.

<img class="alignnone wp-image-90" src="http://dustinwatts.me/wp-content/uploads/2019/08/backup_disconnect-300x202.png" alt="" width="686" height="462" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/backup_disconnect-300x202.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/backup_disconnect-768x516.png 768w, http://dustinwatts.me/wp-content/uploads/2019/08/backup_disconnect.png 835w" sizes="(max-width: 686px) 100vw, 686px" /> 

&nbsp;

## Search For User

### Getting Into the VHD

Reading just a little in the forums led to a hint from L4mpje that the VHD files can be opened remotely.

Initial googling led to opening VHD files with native windows disk management, but that wound require making a windows VM, or opening my host machine to a network full of hackers, so I decided to keep searching&#8230;

Eventually found an article explaining how to open VHD files remotely from linux, perfectly fitting the task at hand  
<https://medium.com/@klockw3rk/mounting-vhd-file-on-kali-linux-through-remote-share-f2f9542c1f25>

<span class="lang:sh decode:true crayon-inline">mount -t cifs //10.10.10.134/Backups /mnt/remote -o ro</span>

That is the command explained in the article above, and it allowed me to mount the remote SMB share to a local folder&#8230;

<img class="alignnone wp-image-91" src="http://dustinwatts.me/wp-content/uploads/2019/08/backups_mounted-300x22.png" alt="" width="696" height="51" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/backups_mounted-300x22.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/backups_mounted.png 722w" sizes="(max-width: 696px) 100vw, 696px" /> 

&#8230; where I could see and access the VHD files directly! Awesome way to do things and I&#8217;ll definitely be keeping notes on this method.

Another cool technique is to seperately mount the VHD files to another folder and explore inside, using the local shell!

I first opened the smaller VHD, but it turned out to be a system boot partition, so then I opened the larger one to find the next clue.

<img class="alignnone wp-image-92" src="http://dustinwatts.me/wp-content/uploads/2019/08/backups_vhd_opened-300x58.png" alt="" width="693" height="134" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/backups_vhd_opened-300x58.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/backups_vhd_opened.png 722w" sizes="(max-width: 693px) 100vw, 693px" /> 

&nbsp;

### Extracting the Secret Sauce

From here I just explored for a looooong time, looking and prodding at everything.

Eventually looked into opening the Registry and extracting passwords.

Tried mimikatz, but it didn&#8217;t run in kali well.  
Tried a tool to open the registry like a filesystem and poke around, that didn&#8217;t work.  
Eventually found pwdump from the creddump package that works the best:

<img class="alignnone wp-image-93" src="http://dustinwatts.me/wp-content/uploads/2019/08/vhd_pwdump-300x53.png" alt="" width="696" height="123" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/vhd_pwdump-300x53.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/vhd_pwdump.png 720w" sizes="(max-width: 696px) 100vw, 696px" /> 

I knew there were some programs for cracking these hashes with dictionary files, hashcat being one I&#8217;ve used before and also John the Ripper.

From reading some of the forums to get hints previously, I noticed people talking about “John”, so I guessed JtR would be the way to go. I wasn&#8217;t sure what type of hash to put in, and instead of letting john go through them all I looked up the possibilities and tried some until I got it right.

To get the available formats for John, use:

<span class="lang:sh decode:true crayon-inline">john &#8211;list=formats</span>

This is the result of the password cracking:

<img class="alignnone wp-image-94" src="http://dustinwatts.me/wp-content/uploads/2019/08/vhd_john-300x89.png" alt="" width="701" height="208" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/vhd_john-300x89.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/vhd_john.png 721w" sizes="(max-width: 701px) 100vw, 701px" /> 

Awesome, a user password for L4mpje!!!  bureaulampje

###<img class="alignnone wp-image-95" src="http://dustinwatts.me/wp-content/uploads/2019/08/ssh_user-300x89.png" alt="" width="701" height="208" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/ssh_user-300x89.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/ssh_user.png 722w" sizes="(max-width: 701px) 100vw, 701px" /> 

Using the credentials found in the Registry, I logged into the SSH and the flag is in the Desktop folder.

&nbsp;

## The Search for Root

### First Clues

With the user flag found and a live login to the box, I searched around the files and folders for an hour or more. Just poking and prodding.

When reading the forum for clues, several people said to look for a program that seems out of place:

<img class="alignnone wp-image-98" src="http://dustinwatts.me/wp-content/uploads/2019/08/ssh_program_files-300x161.png" alt="" width="706" height="379" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/ssh_program_files-300x161.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/ssh_program_files.png 726w" sizes="(max-width: 706px) 100vw, 706px" /> 

Nothing really out of place there, how about the x86 program files:

<img class="alignnone wp-image-99" src="http://dustinwatts.me/wp-content/uploads/2019/08/ssh_x86_program_files-300x150.png" alt="" width="708" height="354" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/ssh_x86_program_files-300x150.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/ssh_x86_program_files.png 720w" sizes="(max-width: 708px) 100vw, 708px" /> 

Bingo! mRemoteNG looks pretty out of place! I&#8217;ve used this software before, so I know that it stores connections to things, which necessarily includes the passwords. Maybe there&#8217;s some passwords we can retrieve.

### mRemoteNG

This is the directory listing for mRemoteNG:

<pre class="lang:zsh highlight:0 decode:true">l4mpje@BASTION C:\Program Files (x86)\mRemoteNG&gt;dir                             
 Volume in drive C has no label.                                                
 Volume Serial Number is 0CB3-C487                                              

 Directory of C:\Program Files (x86)\mRemoteNG                                  

22-02-2019  15:01    &lt;DIR&gt;          .                                           
22-02-2019  15:01    &lt;DIR&gt;          ..                                          
18-10-2018  23:31            36.208 ADTree.dll                                  
18-10-2018  23:31           346.992 AxInterop.MSTSCLib.dll                      
18-10-2018  23:31            83.824 AxInterop.WFICALib.dll                      
18-10-2018  23:31         2.243.440 BouncyCastle.Crypto.dll                     
18-10-2018  23:30            71.022 Changelog.txt                               
18-10-2018  23:30             3.224 Credits.txt                                 
22-02-2019  15:01    &lt;DIR&gt;          cs-CZ                                       
22-02-2019  15:01    &lt;DIR&gt;          de                                          
22-02-2019  15:01    &lt;DIR&gt;          el                                          
22-02-2019  15:01    &lt;DIR&gt;          en-US                                       
22-02-2019  15:01    &lt;DIR&gt;          es                                          
22-02-2019  15:01    &lt;DIR&gt;          es-AR                                       
22-02-2019  15:01    &lt;DIR&gt;          Firefox                                     
22-02-2019  15:01    &lt;DIR&gt;          fr                                          
18-10-2018  23:31         1.966.960 Geckofx-Core.dll                            
05-07-2017  01:31         4.482.560 Geckofx-Core.pdb                            
18-10-2018  23:31           143.728 Geckofx-Winforms.dll                        
05-07-2017  01:31           259.584 Geckofx-Winforms.pdb                        
22-02-2019  15:01    &lt;DIR&gt;          Help                                        
22-02-2019  15:01    &lt;DIR&gt;          hu                                          
22-02-2019  15:01    &lt;DIR&gt;          Icons                                       
18-10-2018  23:31           607.088 Interop.MSTSCLib.dll                        
18-10-2018  23:31           131.440 Interop.WFICALib.dll                        
22-02-2019  15:01    &lt;DIR&gt;          it                                          
22-02-2019  15:01    &lt;DIR&gt;          ja-JP                                       
22-02-2019  15:01    &lt;DIR&gt;          ko-KR                                       
07-10-2018  13:21            18.326 License.txt                                 
18-10-2018  23:31           283.504 log4net.dll                                 
18-10-2018  23:31           412.528 MagicLibrary.dll                            
18-10-2018  23:31         1.552.240 mRemoteNG.exe                               
07-10-2018  13:21            28.317 mRemoteNG.exe.config                        
18-10-2018  23:30         2.405.888 mRemoteNG.pdb                               
22-02-2019  15:01    &lt;DIR&gt;          nb-NO                                       
22-02-2019  15:01    &lt;DIR&gt;          nl                                          
18-10-2018  23:31           451.952 ObjectListView.dll                          
22-02-2019  15:01    &lt;DIR&gt;          pl                                          
22-02-2019  15:01    &lt;DIR&gt;          pt                                          
22-02-2019  15:01    &lt;DIR&gt;          pt-BR                                       
07-10-2018  13:21           707.952 PuTTYNG.exe                                 
07-10-2018  13:21               887 Readme.txt                                  
18-10-2018  23:31           415.088 Renci.SshNet.dll                            
22-02-2019  15:01    &lt;DIR&gt;          ru                                          
22-02-2019  15:01    &lt;DIR&gt;          Schemas                                     
22-02-2019  15:01    &lt;DIR&gt;          Themes                                      
22-02-2019  15:01    &lt;DIR&gt;          tr-TR                                       
22-02-2019  15:01    &lt;DIR&gt;          uk                                          
18-10-2018  23:31           152.432 VncSharp.dll                                
18-10-2018  23:31           312.176 WeifenLuo.WinFormsUI.Docking.dll            
18-10-2018  23:31            55.152 WeifenLuo.WinFormsUI.Docking.ThemeVS2003.dll
18-10-2018  23:31           168.816 WeifenLuo.WinFormsUI.Docking.ThemeVS2012.dll
18-10-2018  23:31           217.968 WeifenLuo.WinFormsUI.Docking.ThemeVS2013.dll
18-10-2018  23:31           243.056 WeifenLuo.WinFormsUI.Docking.ThemeVS2015.dll
22-02-2019  15:01    &lt;DIR&gt;          zh-CN                                       
22-02-2019  15:01    &lt;DIR&gt;          zh-TW                                       
              28 File(s)     17.802.352 bytes                                   
              28 Dir(s)  11.376.021.504 bytes free                              

</pre>

After looking at the mRemoteNG.exe.config, there wasn&#8217;t anyting obviously useful to me. So I looked in the program data folders.

<pre class="lang:zsh highlight:0 decode:true" title="program data folder">l4mpje@BASTION C:\Program Files (x86)\mRemoteNG&gt;dir C:\ProgramData              
 Volume in drive C has no label.                                                
 Volume Serial Number is 0CB3-C487                                              

 Directory of C:\ProgramData                                                    

16-07-2016  15:23    &lt;DIR&gt;          Comms                                       
22-02-2019  13:36    &lt;DIR&gt;          regid.1991-06.com.microsoft                 
16-07-2016  15:23    &lt;DIR&gt;          SoftwareDistribution                        
25-04-2019  06:08    &lt;DIR&gt;          ssh                                         
12-09-2016  13:37    &lt;DIR&gt;          USOPrivate                                  
12-09-2016  13:37    &lt;DIR&gt;          USOShared                                   
16-04-2019  12:18    &lt;DIR&gt;          VMware                                      
               0 File(s)              0 bytes                                   
               7 Dir(s)  11.374.854.144 bytes free</pre>

Nope, nothing there either.

And in AppData\Local\mRemoteNG there is only one file, user.config

<pre class="lang:xhtml decode:true" title="user.config">l4mpje@BASTION C:\Users\L4mpje\AppData\Local\mRemoteNG\mRemoteNG.exe_Url_pjpxdeh
xpaaorqg2thmuhl11a34i3ave\1.76.11.40527&gt;type user.config                        
&lt;?xml version="1.0" encoding="utf-8"?&gt;                                          
&lt;configuration&gt;                                                                 
    &lt;userSettings&gt;                                                              
        &lt;mRemoteNG.Settings&gt;                                                    
            &lt;setting name="MainFormLocation" serializeAs="String"&gt;              
                &lt;value&gt;-8, -8&lt;/value&gt;                                           
            &lt;/setting&gt;                                                          
            &lt;setting name="MainFormSize" serializeAs="String"&gt;                  
                &lt;value&gt;1040, 744&lt;/value&gt;                                        
            &lt;/setting&gt;                                                          
            &lt;setting name="MainFormState" serializeAs="String"&gt;                 
                &lt;value&gt;Maximized&lt;/value&gt;                                        
            &lt;/setting&gt;                                                          
            &lt;setting name="MainFormKiosk" serializeAs="String"&gt;                 
                &lt;value&gt;False&lt;/value&gt;                                            
            &lt;/setting&gt;                                                          
            &lt;setting name="DoUpgrade" serializeAs="String"&gt;                     
                &lt;value&gt;False&lt;/value&gt;                                            
            &lt;/setting&gt;                                                          
            &lt;setting name="LoadConsFromCustomLocation" serializeAs="String"&gt;    
                &lt;value&gt;False&lt;/value&gt;                                            
            &lt;/setting&gt;                                                          
            &lt;setting name="FirstStart" serializeAs="String"&gt;                    
                &lt;value&gt;False&lt;/value&gt;                                            
            &lt;/setting&gt;                                                          
            &lt;setting name="ResetPanels" serializeAs="String"&gt;                   
                &lt;value&gt;False&lt;/value&gt;                                            
            &lt;/setting&gt;                                                          
            &lt;setting name="NoReconnect" serializeAs="String"&gt;                   
                &lt;value&gt;False&lt;/value&gt;                                            
            &lt;/setting&gt;                                                          
            &lt;setting name="ExtAppsTBVisible" serializeAs="String"&gt;              
                &lt;value&gt;False&lt;/value&gt;                                            
            &lt;/setting&gt;                                                          
            &lt;setting name="ExtAppsTBShowText" serializeAs="String"&gt;             
                &lt;value&gt;True&lt;/value&gt;                                             
            &lt;/setting&gt;                                                          
            &lt;setting name="ExtAppsTBLocation" serializeAs="String"&gt;             
                &lt;value&gt;3, 25&lt;/value&gt;                                            
            &lt;/setting&gt;                                                          
            &lt;setting name="ExtAppsTBParentDock" serializeAs="String"&gt;           
                &lt;value&gt;Bottom&lt;/value&gt;                                           
            &lt;/setting&gt;                                                          
            &lt;setting name="QuickyTBVisible" serializeAs="String"&gt;               
                &lt;value&gt;True&lt;/value&gt;                                             
            &lt;/setting&gt;                                                          
            &lt;setting name="QuickyTBLocation" serializeAs="String"&gt;              
                &lt;value&gt;3, 24&lt;/value&gt;                                            
            &lt;/setting&gt;                                                          
            &lt;setting name="QuickyTBParentDock" serializeAs="String"&gt;            
                &lt;value&gt;Top&lt;/value&gt;                                              
            &lt;/setting&gt;                                                          
            &lt;setting name="ResetToolbars" serializeAs="String"&gt;                 
                &lt;value&gt;False&lt;/value&gt;                                            
            &lt;/setting&gt;                                                          
            &lt;setting name="CheckForUpdatesAsked" serializeAs="String"&gt;          
                &lt;value&gt;True&lt;/value&gt;                                             
            &lt;/setting&gt;                                                          
            &lt;setting name="CheckForUpdatesLastCheck" serializeAs="String"&gt;      
                &lt;value&gt;02/22/2019 13:01:46&lt;/value&gt;                              
            &lt;/setting&gt;                                                          
            &lt;setting name="UpdatePending" serializeAs="String"&gt;                 
                &lt;value&gt;False&lt;/value&gt;                                            
            &lt;/setting&gt;                                                          
            &lt;setting name="ThemeName" serializeAs="String"&gt;                     
                &lt;value&gt;vs2015light&lt;/value&gt;                                      
            &lt;/setting&gt;                                                          
            &lt;setting name="PuttySavedSessionsPanel" serializeAs="String"&gt;       
                &lt;value&gt;General&lt;/value&gt;                                          
            &lt;/setting&gt;                                                          
            &lt;setting name="EncryptionEngine" serializeAs="String"&gt;              
                &lt;value&gt;AES&lt;/value&gt;                                              
            &lt;/setting&gt;                                                          
            &lt;setting name="EncryptionBlockCipherMode" serializeAs="String"&gt;     
                &lt;value&gt;GCM&lt;/value&gt;                                              
            &lt;/setting&gt;                                                          
            &lt;setting name="LogFilePath" serializeAs="String"&gt;                   
                &lt;value&gt;C:\Users\L4mpje\AppData\Roaming\mRemoteNG\mRemoteNG.log&lt;/
value&gt;                                                                          
            &lt;/setting&gt;                                                          
            &lt;setting name="MultiSshToolbarLocation" serializeAs="String"&gt;       
                &lt;value&gt;3, 0&lt;/value&gt;                                             
            &lt;/setting&gt;                                                          
            &lt;setting name="MultiSshToolbarParentDock" serializeAs="String"&gt;     
                &lt;value&gt;Top&lt;/value&gt;                                              
            &lt;/setting&gt;                                                          
            &lt;setting name="MultiSshToolbarVisible" serializeAs="String"&gt;        
                &lt;value&gt;False&lt;/value&gt;                                            
            &lt;/setting&gt;                                                          
            &lt;setting name="MainFormRestoreSize" serializeAs="String"&gt;           
                &lt;value&gt;1040, 610&lt;/value&gt;                                        
            &lt;/setting&gt;                                                          
            &lt;setting name="MainFormRestoreLocation" serializeAs="String"&gt;       
                &lt;value&gt;0, 156&lt;/value&gt;                                           
            &lt;/setting&gt;                                                          
        &lt;/mRemoteNG.Settings&gt;                                                   
    &lt;/userSettings&gt;                                                             
&lt;/configuration&gt;                                                        
</pre>

That gave me the path of the log file, but nothing was really interesting in the log file itself. However, other things were in the AppData folder:

<pre class="lang:zsh highlight:0 decode:true" title="appdata">l4mpje@BASTION C:\Users\L4mpje\AppData\Roaming\mRemoteNG&gt;dir                    
 Volume in drive C has no label.                                                
 Volume Serial Number is 0CB3-C487                                              

 Directory of C:\Users\L4mpje\AppData\Roaming\mRemoteNG                         

22-02-2019  15:03    &lt;DIR&gt;          .                                           
22-02-2019  15:03    &lt;DIR&gt;          ..                                          
22-02-2019  15:03             6.316 confCons.xml                                
22-02-2019  15:02             6.194 confCons.xml.20190222-1402277353.backup     
22-02-2019  15:02             6.206 confCons.xml.20190222-1402339071.backup     
22-02-2019  15:02             6.218 confCons.xml.20190222-1402379227.backup     
22-02-2019  15:02             6.231 confCons.xml.20190222-1403070644.backup     
22-02-2019  15:03             6.319 confCons.xml.20190222-1403100488.backup     
22-02-2019  15:03             6.318 confCons.xml.20190222-1403220026.backup     
22-02-2019  15:03             6.315 confCons.xml.20190222-1403261268.backup     
22-02-2019  15:03             6.316 confCons.xml.20190222-1403272831.backup     
22-02-2019  15:03             6.315 confCons.xml.20190222-1403433299.backup     
22-02-2019  15:03             6.316 confCons.xml.20190222-1403486580.backup     
22-02-2019  15:03                51 extApps.xml                                 
22-02-2019  15:03             5.217 mRemoteNG.log                               
22-02-2019  15:03             2.245 pnlLayout.xml                               
22-02-2019  15:01    &lt;DIR&gt;          Themes                                      
              14 File(s)         76.577 bytes                                   
               3 Dir(s)  11.441.676.288 bytes free</pre>

confCons.xml looked particularly interesting. It was clearly important since it had all those backups.

<pre class="lang:xhtml decode:true" title="confCons">l4mpje@BASTION C:\Users\L4mpje\AppData\Roaming\mRemoteNG&gt;type confCons.xml      
&lt;?xml version="1.0" encoding="utf-8"?&gt;                                          
&lt;mrng:Connections xmlns:mrng="http://mremoteng.org" Name="Connections" Export="f
alse" EncryptionEngine="AES" BlockCipherMode="GCM" KdfIterations="1000" FullFile
Encryption="false" Protected="ZSvKI7j224Gf/twXpaP5G2QFZMLr1iO1f5JKdtIKL6eUg+eWkL
5tKO886au0ofFPW0oop8R8ddXKAx4KK7sAk6AA" ConfVersion="2.6"&gt;                      
    &lt;Node Name="DC" Type="Connection" Descr="" Icon="mRemoteNG" Panel="General" 
Id="500e7d58-662a-44d4-aff0-3a4f547a3fee" Username="Administrator" Domain="" Pas
sword="aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7em
f7lWWA10dQKiw==" Hostname="127.0.0.1" Protocol="RDP" PuttySession="Default Setti
ngs" Port="3389" ConnectToConsole="false" UseCredSsp="true" RenderingEngine="IE"
 ICAEncryptionStrength="EncrBasic" RDPAuthenticationLevel="NoAuth" RDPMinutesToI
dleTimeout="0" RDPAlertIdleTimeout="false" LoadBalanceInfo="" Colors="Colors16Bi
t" Resolution="FitToWindow" AutomaticResize="true" DisplayWallpaper="false" Disp
layThemes="false" EnableFontSmoothing="false" EnableDesktopComposition="false" C
acheBitmaps="false" RedirectDiskDrives="false" RedirectPorts="false" RedirectPri
nters="false" RedirectSmartCards="false" RedirectSound="DoNotPlay" SoundQuality=
"Dynamic" RedirectKeys="false" Connected="false" PreExtApp="" PostExtApp="" MacA
ddress="" UserField="" ExtApp="" VNCCompression="CompNone" VNCEncoding="EncHexti
le" VNCAuthMode="AuthVNC" VNCProxyType="ProxyNone" VNCProxyIP="" VNCProxyPort="0
" VNCProxyUsername="" VNCProxyPassword="" VNCColors="ColNormal" VNCSmartSizeMode
="SmartSAspect" VNCViewOnly="false" RDGatewayUsageMethod="Never" RDGatewayHostna
me="" RDGatewayUseConnectionCredentials="Yes" RDGatewayUsername="" RDGatewayPass
word="" RDGatewayDomain="" InheritCacheBitmaps="false" InheritColors="false" Inh
eritDescription="false" InheritDisplayThemes="false" InheritDisplayWallpaper="fa
lse" InheritEnableFontSmoothing="false" InheritEnableDesktopComposition="false" 
InheritDomain="false" InheritIcon="false" InheritPanel="false" InheritPassword="
false" InheritPort="false" InheritProtocol="false" InheritPuttySession="false" I
nheritRedirectDiskDrives="false" InheritRedirectKeys="false" InheritRedirectPort
s="false" InheritRedirectPrinters="false" InheritRedirectSmartCards="false" Inhe
ritRedirectSound="false" InheritSoundQuality="false" InheritResolution="false" I
nheritAutomaticResize="false" InheritUseConsoleSession="false" InheritUseCredSsp
="false" InheritRenderingEngine="false" InheritUsername="false" InheritICAEncryp
tionStrength="false" InheritRDPAuthenticationLevel="false" InheritRDPMinutesToId
leTimeout="false" InheritRDPAlertIdleTimeout="false" InheritLoadBalanceInfo="fal
se" InheritPreExtApp="false" InheritPostExtApp="false" InheritMacAddress="false"
 InheritUserField="false" InheritExtApp="false" InheritVNCCompression="false" In
heritVNCEncoding="false" InheritVNCAuthMode="false" InheritVNCProxyType="false" 
InheritVNCProxyIP="false" InheritVNCProxyPort="false" InheritVNCProxyUsername="f
alse" InheritVNCProxyPassword="false" InheritVNCColors="false" InheritVNCSmartSi
zeMode="false" InheritVNCViewOnly="false" InheritRDGatewayUsageMethod="false" In
heritRDGatewayHostname="false" InheritRDGatewayUseConnectionCredentials="false" 
InheritRDGatewayUsername="false" InheritRDGatewayPassword="false" InheritRDGatew
ayDomain="false" /&gt;                                                             
    &lt;Node Name="L4mpje-PC" Type="Connection" Descr="" Icon="mRemoteNG" Panel="Ge
neral" Id="8d3579b2-e68e-48c1-8f0f-9ee1347c9128" Username="L4mpje" Domain="" Pas
sword="yhgmiu5bbuamU3qMUKc/uYDdmbMrJZ/JvR1kYe4Bhiu8bXybLxVnO0U9fKRylI7NcB9QuRsZV
vla8esB" Hostname="192.168.1.75" Protocol="RDP" PuttySession="Default Settings" 
Port="3389" ConnectToConsole="false" UseCredSsp="true" RenderingEngine="IE" ICAE
ncryptionStrength="EncrBasic" RDPAuthenticationLevel="NoAuth" RDPMinutesToIdleTi
meout="0" RDPAlertIdleTimeout="false" LoadBalanceInfo="" Colors="Colors16Bit" Re
solution="FitToWindow" AutomaticResize="true" DisplayWallpaper="false" DisplayTh
emes="false" EnableFontSmoothing="false" EnableDesktopComposition="false" CacheB
itmaps="false" RedirectDiskDrives="false" RedirectPorts="false" RedirectPrinters
="false" RedirectSmartCards="false" RedirectSound="DoNotPlay" SoundQuality="Dyna
mic" RedirectKeys="false" Connected="false" PreExtApp="" PostExtApp="" MacAddres
s="" UserField="" ExtApp="" VNCCompression="CompNone" VNCEncoding="EncHextile" V
NCAuthMode="AuthVNC" VNCProxyType="ProxyNone" VNCProxyIP="" VNCProxyPort="0" VNC
ProxyUsername="" VNCProxyPassword="" VNCColors="ColNormal" VNCSmartSizeMode="Sma
rtSAspect" VNCViewOnly="false" RDGatewayUsageMethod="Never" RDGatewayHostname=""
 RDGatewayUseConnectionCredentials="Yes" RDGatewayUsername="" RDGatewayPassword=
"" RDGatewayDomain="" InheritCacheBitmaps="false" InheritColors="false" InheritD
escription="false" InheritDisplayThemes="false" InheritDisplayWallpaper="false" 
InheritEnableFontSmoothing="false" InheritEnableDesktopComposition="false" Inher
itDomain="false" InheritIcon="false" InheritPanel="false" InheritPassword="false
" InheritPort="false" InheritProtocol="false" InheritPuttySession="false" Inheri
tRedirectDiskDrives="false" InheritRedirectKeys="false" InheritRedirectPorts="fa
lse" InheritRedirectPrinters="false" InheritRedirectSmartCards="false" InheritRe
directSound="false" InheritSoundQuality="false" InheritResolution="false" Inheri
tAutomaticResize="false" InheritUseConsoleSession="false" InheritUseCredSsp="fal
se" InheritRenderingEngine="false" InheritUsername="false" InheritICAEncryptionS
trength="false" InheritRDPAuthenticationLevel="false" InheritRDPMinutesToIdleTim
eout="false" InheritRDPAlertIdleTimeout="false" InheritLoadBalanceInfo="false" I
nheritPreExtApp="false" InheritPostExtApp="false" InheritMacAddress="false" Inhe
ritUserField="false" InheritExtApp="false" InheritVNCCompression="false" Inherit
VNCEncoding="false" InheritVNCAuthMode="false" InheritVNCProxyType="false" Inher
itVNCProxyIP="false" InheritVNCProxyPort="false" InheritVNCProxyUsername="false"
 InheritVNCProxyPassword="false" InheritVNCColors="false" InheritVNCSmartSizeMod
e="false" InheritVNCViewOnly="false" InheritRDGatewayUsageMethod="false" Inherit
RDGatewayHostname="false" InheritRDGatewayUseConnectionCredentials="false" Inher
itRDGatewayUsername="false" InheritRDGatewayPassword="false" InheritRDGatewayDom
ain="false" /&gt;                                                                  
&lt;/mrng:Connections&gt;                            
</pre>

On close inspection I could see there was a connection defined for L4mpje, and another for Administrator. The Administrator password is what we want, but it&#8217;s encrypted.

We&#8217;re given encryption details at the beginning of the file. One possible option is to take those details and try to decrypt it. But I don&#8217;t want to do that if we can get mRemoteNG to do the decryption for us.

The forum posts suggested Google had the answers once we discovered which software was out of place. I found that an external command can be made  that will display the decrypted password!!!

If you download the Zip version instead of installer, everything stays in the same folder, so it&#8217;ll be a little easier to manipulate. To get the connection details into mRemoteNG, copy the text that was extracted from the SSH session into an xml file and save it as confCons.xml.

After copying the text, saving it, and opening mRemoteNG, I got an error:

<img class="alignnone wp-image-100" src="http://dustinwatts.me/wp-content/uploads/2019/08/mremoteng_error-300x180.png" alt="" width="495" height="297" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/mremoteng_error-300x180.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/mremoteng_error.png 568w" sizes="(max-width: 495px) 100vw, 495px" />  
Turns out a straight copy of the text from the shell resulted in data not formed well, so I had to clean up the XML and reload. After that it worked great.

Then the external tools script could be put in to extract the password.

<img class="alignnone wp-image-101 size-full" src="http://dustinwatts.me/wp-content/uploads/2019/08/mremoteng_showpass.png" alt="" width="1087" height="759" srcset="http://dustinwatts.me/wp-content/uploads/2019/08/mremoteng_showpass.png 1087w, http://dustinwatts.me/wp-content/uploads/2019/08/mremoteng_showpass-300x209.png 300w, http://dustinwatts.me/wp-content/uploads/2019/08/mremoteng_showpass-768x536.png 768w, http://dustinwatts.me/wp-content/uploads/2019/08/mremoteng_showpass-1024x715.png 1024w" sizes="(max-width: 1087px) 100vw, 1087px" /> 

Admin password found!! With that you can log into the SSH with Administrator and grab the root flag!