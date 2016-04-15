#Instructions for setting up the .NET Core debugger using daily builds
This page gives you detailed instructions on how to debug code running under .NET Core in VS Code using daily builds of the C# extension and .NET CLI. You should expect most things to work, but these are daily builds. **Sometimes there will be bumps in the road.**

####Your Feedback​
File bugs and feature requests [here](https://github.com/OmniSharp/omnisharp-vscode/issues) and [join our insiders group](http://landinghub.visualstudio.com/dotnetcoreinsiders) to help us build great tooling for .NET Core.

####Requirements
* Requires .NET Core RC2 (will not work with earlier versions)
* X64 only
* Supports OSX, Ubuntu 14.04, Red Hat Enterprise Linux 7.2, Debian 8.2, Centos 7.1, and Windows 7+

###First Time setup
##### 1: Get Visual Studio Code
Install Visual Studio Code (VSC). Pick the latest VSC version from here: https://code.visualstudio.com Make sure it is at least 0.10.10. 

If you are not sure what version you have, you can see your version of VS Code:

* **OSX:** Code->Abort Visual Studio Code
* **Windows / Linux:** Help->Abort

##### 2: Install .NET command line tools

---

**OSX**

If you have previously installed, remove old versions --

    sudo rm -rf /usr/local/share/dotnet

Download and install: https://dotnetcli.blob.core.windows.net/dotnet/beta/Installers/Latest/dotnet-dev-osx-x64.latest.pkg

Install OpenSSL:

    brew install openssl

---

**Windows**

Uninstall: Go to Control Panel->Add or remove programs, search for '.NET Core CLI' and uninstall.

Download and install: https://dotnetcli.blob.core.windows.net/dotnet/beta/Installers/Latest/dotnet-dev-win-x64.latest.exe

---

**Ubuntu**

One time only - add the .NET Core feed to apt-get

    sudo sh -c 'echo "deb [arch=amd64] http://apt-mo.trafficmanager.net/repos/dotnet/ trusty main" > /etc/apt/sources.list.d/dotnetdev.list' 
    sudo apt-key adv --keyserver apt-mo.trafficmanager.net --recv-keys 417A0893

Uninstall old versions:

    sudo apt-get remove dotnet-sharedframework-microsoft.netcore.app dotnet-dev-1.0.0 dotnet-host

Find the current version:

    sudo apt-get update
    apt-cache policy dotnet-dev-1.0.0-rc2 | grep dotnet-dev-1.0.0-rc2-00 | sort | tail -1

Install the newest:

    sudo apt-get install dotnet-dev-1.0.0-rc2-<ver-number>

**Other Linux distros with no installers for .NET command line tools (CentOS / RHEL)**

Go to https://github.com/dotnet/cli and find the ".NET Core SDK Binaries" download link for your distribution

Extract the downloaded archive into a folder on your machine

Add 'dotnet' executable to your PATH as the debugger in C# extension needs to locate it. You can do this via:
    
    echo "export PATH=<the downloaded folder path>:\$PATH" | sudo tee /etc/profile.d/setdotnetpath.sh
    sudo chmod +x /etc/profile.d/setdotnetpath.sh

Log out and log back in for the changes to take effect

---

##### 3: Install C# Extension for VS Code

* Go to https://github.com/OmniSharp/omnisharp-vscode/releases/download/v1.0.3-rc2/csharp-1.0.3-rc2.vsix and download the extension.
* Start VS Code
* File->Open and open the downloaded vsix file

##### 4: Wait for download of platform-specific files 
The first time that C# code is opened in VS Code, the extension will download the platform-specific files needed for debugging and editing. Debugging and editor features will not work until these steps finish.


###Once for each project
The following steps have to executed for every project. 

##### 1: Get a project
You can start from scratch by creating an empty project with `dotnet new`:

    cd ~
    mkdir MyApplication
    cd MyApplication
    dotnet new
    
Open the project.json and change the 'Microsoft.NETCore.App' version to 'rc2-24008'
Verify that worked by doing:

    dotnet restore

You can also find some example projects on https://github.com/aspnet/cli-samples

**If dotnet restore fails**: Depending on what build you are using, dotnet restore may fail because dotnet new referenced a package which is on myget.org, but didn't create a NuGet.Config file. If so, create a NuGet.Config file in the project directory with the following content:

    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
      <packageSources>
        <!--To inherit the global NuGet package sources remove the <clear/> line below -->
        <clear />
        <add key="dotnet-core" value="https://www.myget.org/F/dotnet-core/api/v3/index.json" />
        <add key="api.nuget.org" value="https://api.nuget.org/v3/index.json" />
      </packageSources>
    </configuration>


##### 2: Open the directory in VS Code
Go to File->Open and open the directory in Visual Studio Code. If this is the first time that the C# extension has been activated, it will now download additional platform-specific dependencies.

**Troubleshooting 'Error while installing .NET Core Debugger':** If the debugger is failing to download its platform-specific dependencies, first verify that you have the RC2 build of the .NET CLI installed, and it is functioning. You can check this by starting a bash/command prompt and running 'dotnet --info'. 

If the CLI is installed, here are a few additional suggestions:

* If clicking on 'View Log' doesn't show a log this means that running the 'dotnet --info' command failed. If it succeeds in bash/command prompt, but fails from VS Code, this likely means that your computer once had an older build of .NET CLI installed, and there are still remnants of it which cause VS Code and other processes besides bash to use the older version instead of the current version. You can try to clean your computer using the uninstall suggestions from http://dotnet.github.io/getting-started/.
* If 'dotnet restore' is failing, make sure you have an internet connection to nuget.org, and make sure that if additional NuGet.Config files are being used, they have valid content. The log will indicate what NuGet.Config files were used. Try removing the files other than the one coming from the extension itself.

##### 3: Add VS Code configuration files to the workspace
VS Code needs to be configured so it understands how to build your project and debug it. For this there are two files which need to be added -- .vscode/tasks.json and .vscode/launch.json. 

* Tasks.json is used to configure what command line command is executed to build your project, and launch.json configures the type of debugger you want to use, and what program should be run under that debugger. 
* Launch.json configures VS Code to run the build task from tasks.json so that your program is automatically up-to-date each time you go to debug it.

For most projects, the C# extension can automatically generate these files for you. When you open a project and the C# extension is installed, you should see the following prompt in VS Code:

![Info: Required assets to build and debug are missing from your project. Add them? Yes | Close](https://raw.githubusercontent.com/wiki/OmniSharp/omnisharp-vscode/images/info-bar-add-required-assets.png)

Clicking 'Yes' on this prompt should add these resources.

**Creating configuration files manually**

In case you would rather generate .vscode/tasks.json by hand, you can start with [this example](https://raw.githubusercontent.com/wiki/OmniSharp/omnisharp-vscode/ExampleCode/tasks.json) which configures VS Code to launch 'dotnet build'. If you don't want to build from VS Code at all, you can skip this file. If you do this, you will need to comment out the 'preLaunchTask' from .vscode/launch.json when you create it.

In case you would rather generate .vscode/launch.json by hand, when you want to start debugging, press the debugger play button (or hit F5) as you would normally do. VS Code will provide a list of templates to select from. Pick ".NET Core" from this list and the edit the 'program' property to indicate the path to the application dll or .NET Core host executable to launch.

##### 4: Windows Only: Enable Portable PDBs
In the future, this step will go away, but for now you need to [change the project.json to use portable PDBs](https://github.com/OmniSharp/omnisharp-vscode/wiki/Portable-PDBs#net-cli-projects-projectjson).

##### 5: Pick your debug configuration

The default launch.json offers several different launch configurations depending on what kind of app you are building -- one for command line, one for web, and one for attaching to a running process. 

To configure which configuration you want, bring up the Debug view by clicking on the Debugging icon in the View Bar on the side of VS Code.

![Debug view icon](https://raw.githubusercontent.com/wiki/OmniSharp/omnisharp-vscode/images/debugging_debugicon.png)

Now open the configuration drop down from the top and select the one you want.

![Debug launch configuration drop down](https://raw.githubusercontent.com/wiki/OmniSharp/omnisharp-vscode/images/debug-launch-configurations.png)

###Debugging Code compiled on another computer
* If the target binary is built on Linux / OSX, dotnet CLI will produce portable pdbs by default so no action is necessary.
* On Windows, you will need to take additional steps to build [portable PDBs](https://github.com/OmniSharp/omnisharp-vscode/wiki/Portable-PDBs#how-to-generate-portable-pdbs).

####More things to configure In launch.json
#####Just My Code
You can optionally disable justMyCode by setting it to "false". You should disable Just My Code when you are trying to debug into a library that you pulled down which doesn't have symbols or is optimized.

    "justMyCode":false*

Just My Code is a set of features that makes it easier to focus on debugging your code by hiding some of the details of optimized libraries that you might be using, like the .NET Framework itself. The most important sub parts of this feature are --

* User-unhandled exceptions: automatically stop the debugger just before exceptions are about to be caught by the framework
* Just My Code stepping: when stepping, if framework code calls back to user code, automaticially stop.

#####Source File Map
You can optionally configure a file by file mapping by providing map following this schema:

    "sourceFileMap": {
        "C:\foo":"/home/me/foo"
        }

#####Symbol Path
You can optionally provide paths to symbols following this schema:

    "symbolPath":"[ \"/Volumes/symbols\"]"
