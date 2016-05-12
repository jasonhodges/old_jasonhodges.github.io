---
layout: post
title: Windows User Batch
categories: blog
permalink: /blog/Windows_User_Batch
tags: [Windows, Server, user config]
comments: true
excerpt: Batch file for configuring general settings for a Windows user
seo__desc: Windows batch file for configuring general user settings
seo__key: Windows, batchfile, user config
---

A recent work project required me to migrate users over to a new terminal server. During the setup process for each user of the new server I had to configure various Windows Explorer options, Microsoft Access options, set up the user data source for ODBC, map a network drive and copy a set of shortcuts out to the users Desktop. Manually stepping through each one of these tasks was time consuming and left room for error. My solution was to write a batch script to handle the process so that all I would need to do after having the user log in is run the batch file, ask the user to perform a test and then provide the user a brief overview. 

I know some people may read this and quickly think "that could be done through a logon script or Group Policy", and I agree, but this box is being shared by other Users that do not need the same configuration. With that in mind it just made since to have a solution that could be manually launched once following the first login of each migrated user. I had no previous experience with writing batch files that modify Windows registry so as soon as I got a taste of what was possible I was confident I could achieve the result I was going for. 

Just to recap, these are the steps I wanted to accomplish -

+ Set Windows Explorer Folder Options for:
    - Show Full Path
    - Uncheck 'Hide Extension of Known File Types'
    - Uncheck 'Remember View'
^
+ Configure Microsoft Access Settings for:
    - Security
    - Keyboard settings
    - Menu settings
+ Copy a set of shortcuts to the Users Desktop
+ Set the User Data Source for ODBC connections
+ Map a network drive 

The first part of the batch script turns `@ECHO ON` so the commands will be displayed within the CMD Prompt window. Next, I map the network drive the User will need access to. The space after the drive letter is key for this to work. `persistent:yes` is used to set the mapped drive to reconnect each time the User logs in. _example drive letter and directory provided for security_

{% highlight winbatch %}
@ECHO ON

@ECHO MAPPING Z DRIVE
net use Z: \\server\directory /persistent:yes
{% endhighlight %}

The main shortcuts each user would need was placed in a general directory under **Public** so they could easily be copied each time the batch script is executed. Remember I said this server is shared with other Users and we do not want them to have the same shortcuts that is why I am copying for each migrated User I set up. The following command copies any shortcut, `.ink` file type, in the specified directory to the second directory specified in the command. By utilizing `%username%` in the directory path, the command will use the currently logged in User. 

{% highlight winbatch %}
@ECHO INSTALLING SHORTCUTS TO DESKTOP
start xcopy "\Users\Public\Documents\Public Desktop\*.ink" "\Users\%username%\Desktop"
{% endhighlight %}

Since the Users mainly work within a specific directory on the mapped network drive, I created a Windows Explorer shortcut that takes them straight there. This shortcut gets copied to the desktop along with all the other shortcuts in the previous step. As a helpful feature, I pin this shortcut to the toolbar for the Users convenience. This command handles that step:

{% highlight winbatch %}
@ECHO COPYING EXPLORER TO TOOLBAR
start xcopy "\Users\%username%\Desktop\Windows Explorer.ink" "\Users\%username%\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\"
{% endhighlight %}

The next setting I needed to make is the ODBC File Data Source which allows connection to a data provider. Adding this piece into the batch script saved several clicks since the normal process would be to go into Administrator Tools located in the Control Panel and open ODBC, click on the File DSN tab and then set the directory. I found the registry key that holds this setting located under `HKEY_CURRENT_USER\Software\ODBC\`. I exported the individual key to the same directory I have my batch script in and added the following command to the batch. 

{% highlight winbatch %}
@ECHO SET DATA SOURCE FOR ODBC
regedit.exe /s \directory\FileDSN.reg
{% endhighlight %}

The command will call the registry file, therefore adding it to the registry. `/s` is used to silence any prompts Windows may throw due to registry modification. 

There are a few Microsoft Access settings I needed to make for each User and this part became a bit of trial and error. Logged into the server as myself I experimented with making adjustments to Access and looking for changes in the registry keys located under `HKEY_CURRENT_USER\Software\Microsoft\Office\11.0\Access`. After determining that settings I needed to make were held within the keys in this location I set Access up the way it needed to be and exported the entire Access registry directory to a registry file named AccessSetting.reg to my working directory with the batch script. The following command executes the registry file, adding the settings to the Users registry and in return configuring Access. Great!

{% highlight winbatch %}
@ECHO SET ACCESS SETTINGS
regedit.exe /s \directory\AccessSettings.reg
{% endhighlight %}

The remainder of the settings are for adjusting Windows Explorer Folder Options to make it more work friendly for the Users. These settings include *showing extensions of known file types*, *showing the full path in the address bar*, and *unchecking Remember View* so that the settings will remain the same each time the User logs in. First figuring out where in the registry these settings are stored was a mediocre task but finding any good information about how to get the settings to stick was brutal. I searched and searched for a decent source on the web that explained how to make the Explorer adjustments I wanted and all I found were posts on what registry keys held the settings, nothing on how to actually script them in. After much testing, settings checking, registry modifying… it clicked! I had my steps out of order. I learned through experimentation that the Explorer folder settings would not take if Explorer was running. So the sensible thing to do was have the batch script kill Explorer, then load the registry keys and restart Explorer. The keys I had exported from the registry were 
`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced` and `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\CabinetState`. 
The following are the final commands to finish off the migrated User setup. Kill Explorer, load Explorer registry settings, restart Explorer. 

{% highlight winbatch %}
@ECHO KILL EXPLORER
taskkill /f /im explorer.exe

@ECHO SET EXPLORER SETTINGS
regedit.exe /s \directory\Explorer_Advanced.reg

regedit.exe /s \directory\Explorer_CabinetState.reg

@ECHO RESTART EXPLORER
explorer.exe

pause
{% endhighlight %}

The `pause` at the end will force the CMD Prompt window to stay open so the status of each step in the batch script process can be viewed, in case of any problems. 

The whole file looks like this:

{% highlight winbatch %}
@ECHO ON

@ECHO MAPPING Z DRIVE
net use Z: \\server\directory /persistent:yes

@ECHO INSTALLING SHORTCUTS TO DESKTOP
start xcopy "\Users\Public\Documents\Public Desktop\*.ink" "\Users\%username%\Desktop"

@ECHO COPYING EXPLORER TO TOOLBAR
start xcopy "\Users\%username%\Desktop\Windows Explorer.ink" "\Users\%username%\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\"

@ECHO SET DATA SOURCE FOR ODBC
regedit.exe /s \directory\FileDSN.reg

@ECHO SET ACCESS SETTINGS
regedit.exe /s \directory\AccessSettings.reg

@ECHO KILL EXPLORER
taskkill /f /im explorer.exe

@ECHO SET EXPLORER SETTINGS
regedit.exe /s \directory\Explorer_Advanced.reg

regedit.exe /s \directory\Explorer_CabinetState.reg

@ECHO RESTART EXPLORER
explorer.exe

pause
{% endhighlight %}

If you are reading this and you haven't ever created a batch script before the process is simple. Just open Notepad, add your commands and save the file with a `.bat` file extension. Hopefully someone out there will stumble upon this and find it helpful. 

*modifying Windows registry settings can DESTROY your operating system and I take no responsibility if **you** screw yours up*