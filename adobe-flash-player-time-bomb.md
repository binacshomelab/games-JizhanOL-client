# Ref: https://gist.github.com/KuromeSan/56d8b724c0696b54f9f81994ae3591d1

In Adobe Flash Player versions newer than 32.0.0.344 they added a "Timebomb" for the EOL.
the player would refuse to run any custom flash content after 12/01/2021,
instead it would just show this ![image](https://camo.githubusercontent.com/30ef3ac9555f29f1d600bf91ae09044a2485dcbf8c0d37dca9d313d7be09f82b/68747470733a2f2f692e696d6775722e636f6d2f465330515a755a2e706e67)

So knowing this, Lets crack it!

I acturally started looking into this before the 12/01/2021 hit, 
but only recently did i acturally discover a way to bypass the killswitch

(also- im aware i was not the first to do this, but i still did do it)

# Recon stuffs
First thing i wanted to know was, so where does flash install to anyway? its a browser plugin right, 
so its not like theres an obvious "Flash.exe" or whatever,

Well it was as simple as googling the answer, this just applies to windows systems but its in
C:\Windows\System32\Macromed\Flash (32 bit version in SysWOW64)
there are three files it uses for different browsers and apis, the NPAPI Firefox one is NPSWF64.DLL, 
the Chromium verison is PepFlashPlayer_<VERSION>.dll and the activeX version for Internet Explorer and desktop apps is Flash.OCX, 

Oh and google is special and have it in %LocalAppData%\Google\Chrome\User Data\PepperFlash\<VERSION>\Pepflashplayer.dll

# Reversing it!
There were a few ways i thought it might work but one thing about the kill screen is that it still said "Adobe Flash Player 32"
when i right clicked, and had the option for global settings and local settings this made me think that the killscreen really is just
a SWF (Flash Movie) file itself, that it'll load instead of whatever is on the site, knowing this i did a very basic search looking
for "CWS" the flash movie magic number inside the DLL, and i found a few results:
![image](https://camo.githubusercontent.com/7ac32772154fdb7a5ef2f790ec01b4ca6ac8538323b7f4de4ffb682d18251b87/68747470733a2f2f692e696d6775722e636f6d2f496f30714d49322e706e67)

so i copied all the bytes until i saw stuff that didnt look like zlib compressed data,
and opened it in the standalone flash projector- but no. this is just the settings menu, 

![image](https://camo.githubusercontent.com/a5a5ddf1b9af3df42a5dc29321c8ba63a679784038f61196b61368c6f8f14b5b/68747470733a2f2f692e696d6775722e636f6d2f72454a52776f382e706e67) 

i still thought that theres a good chance they use a swf for the killscreen, so i just
searched again, found another CWS header that appears to be directly after the first one
which just appeared to be a white screen, not sure what its for.
after going through all the embedded flash SWF's i finally found it, 
the killscreen swf is the last "CWS" in the NPSWF64 file, located at 0x11B9D58 in the latest version

![image](https://camo.githubusercontent.com/8c593903a3c1356e86dfc7b5576f33550f9542ee6f85168f69517bff3c6eff03/68747470733a2f2f692e696d6775722e636f6d2f37357650464b692e706e67)


So after this i tried opening NPFLASH64.dll in Ghidra and seeing what references this embededed flash movie swf-
turns out it takes ghidra (and ida..) a very long time to anaylize a binary like flash player, its a very big file
with thousands of subroutines, after awhile i found that it calls GetSystemTime, and then has there own implementation for converting that into a Unix Epoch time, then just checks if its greater than 1610409600000, theres also some extra checks in there something about "file://" perhaps the killswitch is ignored if its the contents are served locally? and some other stuff i couldnt tell right away,
i assume have to do with enterprise versions of flash and if the url is allowed in mms.cfg. but thats just a guess,

anyway perhaps the most interesting thing about this is that time timestamp compared against was acturally a double value, so to bypass the killswitch all i had to do was change it from 1610409600000 to "Infinity", which means it'll always be before the kill date and so it'll never show the killswitch screen- so thats it,

# Finally,

to remove the killswitch from flash player you simply have to find and replace 00 00 40 46 3E 6F 77 42 with 00 00 00 00 00 00 FF 7F
you have to mess around with windows security settings to get it to allow you to write to the file but thats basically all there is to it.

also the offline installer downloads for flash player are still on adobe server- if you goto the right URL.
which means you have a definitey-not-to-be-malware way of installing and using flash, 
well, atleast until they pull these links offline.. :D

windows:
https://fpdownload.adobe.com/pub/flashplayer/latest/help/install_flash_player.exe - Firefox / NPAPI
https://fpdownload.adobe.com/pub/flashplayer/latest/help/install_flash_player_ax.exe - Internet Explorer / ActiveX 
https://fpdownload.adobe.com/pub/flashplayer/latest/help/install_flash_player_ppapi.exe - Chrome / PPAPI

mac:
https://fpdownload.macromedia.com/pub/flashplayer/latest/help/install_flash_player_osx.dmg - Firefox / NPAPI
https://fpdownload.macromedia.com/pub/flashplayer/latest/help/install_flash_player_osx_ppapi.dmg - Chrome / PPAPI

dont know anywhere to get the linux versions though unfortunately- 

# TL;DR

to remove the killswitch from flash player you simply have to find 00 00 40 46 3E 6F 77 42 and replace with 00 00 00 00 00 00 FF 7F
i also made a patcher program if your lazy and dont want to mess with windows security settings 
https://github.com/KuromeSan/FlashPatcher/tree/master, 
