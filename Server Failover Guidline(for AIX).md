Server Failover Guidline（ for AIX）
==================================

1. ###Stop service####
	* Netbackup service
	> $/usr/openv/netbackup/bin/goodies/netbackup stop 
	
	* Offhost agent
	>$ ps –ef | grep –I offhost.pl   
	*Kill the offhost agent with the PID*   
	$ kill -9  \<PID\>
	
	* Control-M
	>RG915# su – controlm 		
	RG915-controlm [1] ctm_menu  
	Choose 1- CONTROL-M Manager   
	Choose 1 - Check All   
	Choose 6 - Stop All   
	Choose q – Quit   
	RG915-controlm [2] exit


2. ###Umount SAN disks###
> *Find LDEV list for each TC group in Horcm file*   
$ cat /etc/horcm0.conf   
*Find disk number of each LDEV with HDLM*   
$ /usr/DynamicLinkManager/bin/dlnkmgr view -path  
*find the corresponding VG name with the disk number(s)*      
$ lspv   
*find the involved LV*
$ lsvg –l <vg_name>   
*Check the mount point*		
$ df -k   	
*Umount the filesystem*		
$ umount < dev_name| mount point>


3. ###Check SAN disks status (TC should be in PAIR)###
>$ pairdisplay -IH0 -g \<TC_group\> -l -fxce
￼
4. ###Split TC (TC should be in PSUS)ok###
>*Normally, it can be performed by SAN team*   
$ pairsplit -IH0 -g <TC_group>   
*After split completed, double check the TC status*   
$ pairdisplay -IH0 -g <TC_group> -l -fxce

5. ###Call SAN Team to mount up target side’s SAN disks##

6. ###IP configuration###
	* Remove PG915SV from local side (RG915)
	>smitty tcpip -> Further Configuration ->Network Interfaces -> Network Interface Selection -> Configure Aliases -> Remove an IPV4 Network Alias

	* Assign PG915V to target side (PG915)
	>smitty tcpip -> Further Configuration ->Network Interfaces -> Network Interface Selection -> Configure Aliases -> Remove an IPV4 Network Alias

7. ###Scan disks###
>*Create script in /tmp (cfgmgr.sh)*   
$ vi /tmp/cfgmgr.sh   
*Export below*   
\#!/usr/bin/ksh   
export ODMDIR=/etc/objrepos    
cfgmgr –v    
￼￼￼￼￼$ chmod +x cfgmgr.sh    
$ ./cfgmgr.sh   
	
	* Import VG    
	>￼￼￼￼￼$ importvg –y <vg_name>
	
	* Mount SAN disks
	>￼￼￼￼￼￼￼￼$ mount <dev_name> <mount_point>   

8. ###Start service###
	* Netbackup service
	>$ /usr/openv/netbackup/bin/goodies/netbackup start
	* Offhost agent
	>$ vi /etc/inittab
	*Uncomment the below line*     
	**vrtsoffhost:2:respawn:/usr/openv/ecs/bin/nbu_offhost.pl -server -display> /dev/null 2> &1**   
	$ init q   
	
	* Control-M   
	>PG915# su – controlm    
	PG915-controlm [1] ctm_menu    
	Choose 1- CONTROL-M Manager    
	Choose 2 – Start All      
	Choose 1 - Check All
	Choose q – Quit   
	PG915-controlm [2] exit  
	 
9. ###Reverse sync with TC (PG->RG)    
*Do make sure the failover success, and the backup can be run in the target site without problem, then ask SAN team to reverse sync TC pair.*