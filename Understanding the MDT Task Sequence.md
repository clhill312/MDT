# Understanding the MDT Task Sequence

## Initialization
### 1. Format and partition disk
Formats the disk
### 2. Use Toolkit package
Initializes the Zero Touch deployment process. Setting some base
variables etc.
### 3. Check pre­reqs
Checks if your computer is capable of running the ts and complies with
minimum requirements. Stops the processing if not
### 4. Gather local only
Gathers deployment configuration settings from local sources that
apply to the target computer [ZTIGather.wsf] This step is making your deployment dynamic. [See here](http://blogs.technet.com/benhunter/archive/2007/03/17/understandingbdd­rule­processing.aspx) for a great introduction.
## Validation
### 1. Validate
Verifies that the target computer meets the specified deployment requirement conditions. Such as ensuring minimum memory, minimum processor speed, specifying image size will fit, and ensuring current operating system to be refreshed.
### 2. Check BIOS
Checks the basic input/output system (BIOS) of the target computer to
ensure that it is compatible with the operating system you are deploying.
[ZTIBIOSCheck.wsf]
### 3. Next Phase
Updates the Phase property to the next phase in the deployment process. It
determines the sequence in which each task must be completed. [ZTINextPhase.wsf]
### State Capture
### 1. Generate Application Migration File
Generates the ZTIAppXmlGen.xml file, which
contains a list of file associations and applications that are installed on the target computer.
### 2. Capture User State
Captures the user state for user profiles that exist on the target
computer. [ZTIUserState.wsf]
### 3. Capture Groups
Captures group membership of local groups that exist on the target
computer.
### 4. Capture Network Settings
Gathers the network adapter settings from the target computer
[ZTIICConfig.wsf] It can capture e.g. static IP assignment in refresh scenarios. This step will be ignored if you run a Bare­Metal deployment.
## Refresh Only
### 1. Disable BDE Protectors
If BitLocker is installed on the target computer, this task sequence step disables the BitLocker protectors.
### 2. Apply Windows PE
Prepares the target computer to start in Windows Preinstallation
Environment (Windows PE).
## Preinstall
Meant to run only on new computers, where the Initialization phase did not run. Check the conditions. for this group and the initialization groups
### 1. Configure
Configures the Unattend.xml, Sysprep.inf, or Unattend.txt files with the
required property values that are applicable to the operating system you are deploying to the target computer. [ZTIConfigure.wsf]
### 2. Inject Drivers
Injects drivers that have been configured for deployment to the target computer.
### 3. Validate
((checks the min memory, processor speed,Ensure current operating system to be refreshed why do it again?)
### 4. Format and partition disk
(Why do this again?)
### 5. Use Toolkit Package
(Why do this again?)
## New Computer Only
### 1. Copy Scripts
Copies the deployment scripts used during the deployment processes to a
local hard disk on the target computer. [LTICopyScripts.wsf]
## Refresh Only
### 1. Backup
Backs up the target computer before starting the operating system deployment.
[ZTIBackup.wsf]
## Install
### 1. Restart to WinPE
### 2. Use toolkit package
(again?) ­> Restarted into WinPE so different environment. Need to
ensure some initialization happened.
### 3. Gather
(What does it gather and why do this again..?) ­> Restarted into WinPE so old environement including all variables is gone. Will not evaluate all rules again, just read the gathered values from a file and populate them into the environment.
### 4. Request state store
(what does this step do?) ­> It requests a state store on the SMP
### 5. Backup
(Backup of what?) ­> Will create a full backup of your computer (creating a wim of your harddrive) if necessary. Should be disabled on default. I would always put an additional condition on this to not run it on every deployments. Also the whole group this step is in runs only if the Deployment type is "Refresh"
### 6. Release state store
(what does this step do?)­> releasing the state store that has been
requested from the SMP
### 7. Apply Operating System Image
## Post Install
### 1. Apply Windows Settings
### 2. Apply Network Settings
### 3. Auto Apply Drivers
### 4. Configure
(what does this step do?) ­> Mainly updating the unattend.xml with your
dynamic values (Computername, OU, Domain, etc)
### 5. Setup Windows and config Manager

## State Restore

### 1. Restart computer
### 2. Use toolkit package
(again?) ­> Computer has been restarted. Initialize environment.
### 3. Install Software Updates
### 4. Gather
(Gather what?) ­> Computer has been restarted. publish all locally stored values back into the environment.
### 5. Tattoo
(what does this step do?) ­> Store some information about the deployment in the Registry to be able to collect them later using SCCM
### 6. Install Software
### 7. Restore Groups
(what does this step do?) ­> Restore any local groups if captured during
the "State Capture" phase
### 8. Move State Store
(what does this step do?) ­> moves the state store to a "secure" place at
the end of the task sequence so it does not get deleted.
### 9. Copy Logs
(what does this step do?) ­> it copies some logs (mainly smsts.log) to a
network location if configured (need to set the variable SLShare)

## Capture the reference machine
### 1. Prepare ConfiMgr Client
### 2. Prepare OS
(sysprep?) ­> Yes
### 3. Capture the reference machine
This TaskSequence template covers deploying a captured image but also creating this image. This step runs only if the variable `DoCapture` is set to YES so in most cases it will be ignored. You can safely remove this group if you never use this TS for capturing an image.

## Gather Logs and StateStore on Failure
See this as a Try...Catch block. It will ensure that the state store gets moved to a different location (mostly C:\Windows\Temp) and the logs will be copied to a network folder. It's mainly for error handling and will only execute if the last Action did not succeed. In regard to the "why"
it might have happened a reboot (that could even be the cause for the last error) so we need to initialize the environment first.
### 1. Use Toolkit Package
(again? why?)
### 2. Gather
(Gather what and why?)
### 3. Move state store
(what does this step do?)
### 4. Copy Logs
Copies log files from the local location at C:\... to the location specified in the `SLShare` location.