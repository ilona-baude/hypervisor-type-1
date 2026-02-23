# hypervisor-type-1


## JOB 1 - Informations

#### Définition: 

#### Type 1 / Type 2:

#### Avantages - Inconvénients :

## JOB 2 - ISOs 

#### ESXi : [ X ]
#### Hyper-V : [ X ]
#### Proxmox VE : [ X ] 
#### XCP-ng :  [ X ]  

## JOB 3 - Hyper V - Windows Server 2022

#### STEP 1: Install Windows Server 2022
- VM: 24GB SDD, 4GB RAM, 4VcPU (2x2)
#### STEP 2: Install Hyper - V

- Oublier son mot de passe admin 
	-> utiliser l'exploit Utilman.exe
- Désactiver Hyper-V sur la machine hôte
	 $bcdedit /set hypervisorlaunchtype off
	 $ shutdown -r -t 0
- VM: Edit Settings -> Hardware -> Processors 
-> Cocher Virtualize Intel VT -x/EPT or AMD-V/RVI

- Ajouter rôles et fonctionalitées 
-> Hyper-V
#### STEP 3: Virtualisation Imbriquée 

- Télécharger ISO Debian
- Création d'une VM Hyper-V
	-> Nom et Emplacement
	-> Spécifier la Génération
	-> Affecter la mémoire
	-> Configurer la mise en réseau
	-> Connecter un disque dur virtuel
	-> Option d'installation
- Configurer le Secure Boot
## JOB 4 - Esxi

#### STEP 1: Création d'une machine virtuelle custom

- Hardware Compatibility: ESXi 7.0
- ISO: ESXi 8.0
- Name: ESXi
- Processors: 2x2 -VcPU
- RAM: 8GB
- Network: NAT
- I/O Controller: Paravirtualized SCSI
- Disk: SCSI - 16 GB
#### STEP 2: Installation VMware ESXi 8.0.0

- Accepter les conditions de la licence en appuyant sur « **F11 »**.
- Sélection clavier
- Mot de passe root
- Appuyer sur « **F11 »** pour démarrer l’installation.
- Appuyer sur « **Entrée »** pour redémarrer le serveur hôte ESXi.
#### STEP 3 - DCUI & Web Interface

- DCUI = Direct Console User interface
	-> personnaliser la configuration réseau en appuyant sur « **F2** »
- Web Interface
	-> Section jaune: "https://[Adresse-IP]"
		-> ctrl c + ctrl v dans un navigateur web
#### STEP 4: Virtualisation Imbriquée

- https://IP
- Create New Datastore
- Upload ISO image to Datastore
- Create/Register VM
	-> CD/Drive connect
		Datastore: select ISO

## JOB 5 - PROXMOX VE

#### STEP 1: Installation VM Proxmox

- ISO: Proxmox VE 9.1
	-> Guest OS: Linux - Debian 12
- Hardware - Processors: Virtualize Intel VT-x/EPT or AMD-V/RVI
- Installation complète: reboot
#### STEP 2: Web Interface

https://[IP]:8006/

- Mise à jour environement Proxmox VE
	-> Update > Repository > Add > No-Subscription
	-> Update > Refresh 
	-> Update > Upgrade
- Reboot
- Importer fichier ISO
	-> node:local >ISO Images > Upload / Upload from URL

#### STEP 3: Virtualisation Imbriquée

- Création VM
	 General: Default
	 OS: ISO - Debian 12
	 System: Default + Quemu Agent (VmwareTools)
     Disk: SCSI - 4GB + Discard
     CPU: 1vCPU
     RAM: 1Go
- Network
	-> bridge: vrmb0
- Start Machine 
	-> Console 
	-> Summary: Live usage CPU/RAM 
## JOB 6 - XCP-ng

#### STEP 1: Installation VM XCP-ng
- Create VM
	 Disk: 50Go (46Go min)
	 ISO: xcp-ng 83
	 Guest OS: VMWare ESXi 8.0
- Network:
	 DHCP
	 Optionnel: eth = trunk -> VLAN requis
	 DNS
- Déconnecter CD/Drive Média + Reboot
- TUI Console > Local Command Shell
	 $ yum update -y
- Reboot

#### STEP 2: Web Interface : XOA

- http://IP
	 +New VM
		 - Template: Debian 12 Bookworm
		 - ISO: xcp-ng guest-tool

#### STEP 3 : Virtualisation imbriquée
## JOB 7 - Migration

### Hyper V -> ESXi

STEP 1: VM PREP 
-> install open-vm tools for better integration with esxi

STEP 2 : Export VM from Hyper V
-> Hyper-V manager > VM > export 

STEP 3: Convert .vhdx to .vmdk
-> $qemu-img convert -f vhdx -O vmdk debian-vm.vhdx debian-vm.vmdk

STEP 4: Import to ESXi
-> web ui : storage > datastore > vm folder > upload .vmdk file
-> ssh: $scp output.vmdk root@esxi-host:/vmfs/volumes/datastore1/debian-vm/

STEP5: Convert to ESXi format
-> ESXi host:
	$cd /vmfs/volumes/DATA/debian-vm/ 
	$vmkfstools -i output.vmdk debian-esxi.vmdk -d thin

STEP 6: Create New VM 
-> Select VMs settings
-> import existing hard disk: debian-vm.vmdk
-> NIC = VMXNET3

STEP 7: Clean-up

-> Fix network interface if necessary
-> Remove Hyper-v tools:
	$sudo apt remove hyperv-daemons linux-image-\*hyperv* 2>/dev/null

### ESXi -> Proxmox 

STEP 1 : Power off VMs
-> cli: $vim-cmd vmsvc/power.off [vmid]
-> prevents disk corruption

STEP 2: Export VMs
-> web ui: VM > export > download .ovf file
-> cli: $ovftool vi://root@[esxi-host-ip]/debian-vm /path/to/output/debian-vm.ovf

STEP 3: Transfer file to Proxmox host
-> ssh : scp debian-vm.ovf root@[proxmox-host-ip]:/var/tmp/

STEP 5: Import into proxmox
cli: 
	qm importovf [vmid] debian-vm.ovf local-lvm
STEP 6: Create New VM + Import Disk
	 **Machine type**: set to `q35` or `i440fx`
	 **Bios**: Match ESXi 
	 **Network**: virtio 

STEP 7: Fix network interface
-> ESXi uses VMXNET3 or e1000 and Proxmox virtio or e1000
-> fix /etc/network/interfaces

+Install quemu agent 
-> $apt install qemu-guest-agent 
-> $systemctl enable --now qemu-guest-agent
### Proxmox -> XCP-NG

### XCP-NG - HYPER - V 
## JOB 8 - Sauvegarde de VMs
## JOB 9 - Proxmox Backup Server
## Bonus: Cluster
