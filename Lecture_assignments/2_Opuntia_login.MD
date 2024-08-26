Instructions for accessing 

##### Mac users
* Unix shell: Terminal, which is pre-installed on MacOSX. You can find it through the Finder by going to Go-> Utilities-> Terminal.

##### Windows users
* PuTTY: A terminal which you can install from [the putty website](https://www.putty.org/)
* MobaXterm: A terminal which you can install from [the MobaXterm website](https://mobaxterm.mobatek.net/)
* Built in Linux subsystem in Windows [Windows tech forum](https://techcommunity.microsoft.com/t5/windows-11/how-to-install-the-linux-windows-subsystem-in-windows-11/m-p/2701207)



**BEFORE WE DO THIS, A CRITICAL NOTE:**

If you are on a Mac and working through the terminal, or on a PC an working through MobaXterm, once you open the terminal you are **working in your computer's directories**. This means that you can accidentally delete/corrupt files on your personal computer, **including your operating system.** Please be extremely careful. I am not responsible for any personal sorrow you suffer in this manner.

#### Introduction to Opuntia, the UH training High Performance Cluster for this course.

In order to access this cluster, we have to 1) be on the UH network and 2) open a terminal window. 

You can be "on the UH network" via either the UH Secure wifi network (Note: UH Guest wifi will NOT work for this) or via the UH VPN. Please install the UH VPN so you may access the cluster when you are not on campus. [Instructions for the VPN can be found here.](https://uh.edu/infotech/services/computing/networks/vpn/)

In order to access the cluster from the terminal, you have to point your computer to the location you want to login to. The address for Opuntia is `opuntia.rcdc.uh.edu `, and to login you specify `cougarnet_accounte@opuntia.rcdc.uh.edu`  Your username is the same as your UH password. You use the ssh command to login. So, to access your account you would use:

```bash
ssh your_username@opuntia.rcdc.uh.edu 
```

Your terminal will then prompt you to provide your login information. Congrats! You are logged into the HPC and ready to SCIENCE!
