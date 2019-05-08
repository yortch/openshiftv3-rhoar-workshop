## Lab 1: Setup Visual Studio Code with the Java Extension Pack

In this lab we will install Visual Studio Code and the Java Extension including Language Support for Java by Red Hat.

### Pre-requisites

Please read the pre-requisites for this workshop. If you cannot meet these pre-requisites you may not be successful in this workshop. So let your instructor know.

* You will need a computer where you can install [Visual Studio Code](https://code.visualstudio.com/). Please make sure you have rights to install software on your device!

* Visual Studio Code is supported on the following platforms. So your device must be one of these:
	* Windows
	* macOS
	* Linux
	
	
###  Install Visual Studio 	
	
Copy the installers from flash drive
	


** If MaCOS: *** 

Copy VisualStudio binary  from usb to the location of your choice and follow the below instructions

https://docs.microsoft.com/en-us/visualstudio/mac/installation?view=vsmac-2019








*If Windows:** Copy VisualStudio installer from usb to the location of your choice and follow the below instructions

https://docs.microsoft.com/en-us/visualstudio/install/install-visual-studio?view=vs-2019



### Installing the Command-line Tools ###

After completing this section, you should be able to:

* Locate the binaries for the OpenShift Container Platform command-line
interface (OCP CLI)
* Install the OCP CLI tools.
* CLI basic configuration

*Locating the binaries*

The OCP CLI exposes commands for managing applications, as well as
lower-level tools to interact with each component of a system. The
binaries for Mac, Windows, and Linux are available to copy from flash drive 

Copy the OC libraries from flash drive 


You can also download the libraries from here .

* Supported Binary from the Red Hat Portal: https://access.redhat.com/downloads/content/290
* Upstream Origin Binary: https://github.com/openshift/origin/releases
* Past releases: https://mirror.openshift.com/pub/openshift-v3/clients

*Installing the CLI tools*

The CLI is provided as compressed files that can be decompressed to any
directory. In order to make it simple for any user to access the OCP
CLI, it is recommended that it is made available in a directory mapped
to the environment variable called `PATH` from the OS. More information
can be found about installation process here:
https://docs.openshift.com/container-platform/latest/cli_reference/get_started_cli.html

1.  *OSX and Linux:*
+
1.1. Copy the binary to the `/usr/local/bin` directory, or one of the
paths listed in the `PATH` environment variable.
2.  *Windows:*
+
2.1. Use oc.exe to open an OpenShift shell. If you getting error from
running oc, go to git-scm.com to download git bash for Windows (during
installation you need to specify in the selection to integrate with the
command prompt)
+
2.2. Download and install Notepad++ and install the JSON plugin or use
http://jsonlint.com/ to edit and validate JSON.
+
http://ammonsonline.com/formatted-json-in-notepad/
+
2.3. Configure your default editor to be Notepad++

*CLI basic configuration*

The easiest way to initially setup the OpenShift CLI is to use the
`oc login` command. It’ll interactively ask you a server URL, username
and password. The information is automatically saved in a CLI
configuration file that is then used for subsequent commands.










