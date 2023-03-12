# VSRemoteDebugger
Remote SSH Debugging tool for Visual Studio 2022 (ARM/ARM64/Raspberry Pi compatible)

[![Build Status](https://dev.azure.com/radutomy0781/radutomy/_apis/build/status/radutomy.VSRemoteDebugger?branchName=refs%2Ftags%2Fv1.5)](https://dev.azure.com/radutomy0781/radutomy/_build/latest?definitionId=5&branchName=refs%2Ftags%2Fv1.5)

> **⚠️ Warning ⚠️** Do not trust this repository!
>
> This version uses a local static version of the SSH library because current ssh library does not support newer types of certificates.

## Usage guide

- SSH based authentification needs to be set up between local and remote. Follow the following guide:
https://www.onmsft.com/how-to/how-to-generate-an-ssh-key-in-windows-10
	
	Set the private key on the local machine as `~\.ssh\id_rsa`. 
	Set the public key on the remote machine as `~/.ssh/authorized_keys`.

- Make sure that the remote machine has a user able to run non-shell commands without a password. This is already configured on a RPi.
- Install .NET on the remote machine. Follow the following guide:
	https://learn.microsoft.com/en-us/dotnet/iot/deployment
	
	Run the following command to install .NET:
	```console
	curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --channel STS
	```
	To simplify path resolution, add a DOTNET_ROOT environment variable and add the .dotnet directory to $PATH with the following commands:
	```console
	echo 'export DOTNET_ROOT=$HOME/.dotnet' >> ~/.bashrc
	echo 'export PATH=$PATH:$HOME/.dotnet' >> ~/.bashrc
	source ~/.bashrc
	```
	Verify the .NET installation with the following command:
	```console
	dotnet --version
	```
- Install Visual Studio Debugger https://github.com/Microsoft/MIEngine/wiki/Offroad-Debugging-of-.NET-Core-on-Linux---OSX-from-Visual-Studio
	```console
	curl -sSL https://aka.ms/getvsdbgsh | /bin/sh /dev/stdin -v latest -l ~/vsdbg
	```

Make sure the keys are working before proceeding!!
- Install this extension
- In Visual Studio go to `Tools -> Options -> VsRemoteDebugger -> Remote Machine Settings` and modify the access settings
- In Visual Studio go to `Tools -> (click on) Start Remote Debugger` or add the `right click on Toolbar -> Customize ... -> VSRemoteDebugger Toolbar`

## Example configuration

ASP.NET 7 on Raspberry Pi 3 Model B (win-arm64)

### Local Machine Settings

| Parameter                                  | Value                   |
| -------------------------------------------| ------------------------|
| No Debug (just build and deploy to remote) | False                   |
| Publish                                    | True                    |
| Use Command Line Arguments                 | True                    |

### Remote Machine Settings

| Parameter                                  | Value                   |
| -------------------------------------------| ------------------------|
| .Net Path                                  | /home/pi/.dotnet/dotnet |
| Group Name                                 | pi                      |
| IP Address                                 | raspberrypi             |
| Project folder                             | /home/user/project_name |
| User Name                                  | pi                      |
| Visual Studio Debugger Path                | /home/user/vsdbg/vsdbg  |

### Command line arguments (optional)

Project -> Properties -> Debug -> General -> Open debug launch profiles UI

`--urls=http://raspberrypi:5055/` because remote debugging not using the "App URL" parameter.

## The extension performs the following steps:

1. Builds the solution in Visual Studio 
2. Creates the necessary file structure on the remote machine
3. (optional) if running for the first time, installs Visual Studio's vsdbg debugging server on the remote machine
4. Copies the files from the output folder into the remote machine
5. Connects to the VsDbg server and starts debugging the current project via SSH

## Limitations

- ~~Remote debugging only available for ARM-based machines (for now)~~ Now ARM64 compatible
- Visual feedback is poor when running for the first time
- Not tested (and most likely not working) for 32-bit versions of Windows
- Command line arguments can be used as long as the project doesn't have more than one debugging profile (check by going to Visual Studio -> Project Settings -> Debugging -> Profile dropdown). If the dropdown has more than one entry, then command line arguments cannot be used.
- Currently not compatible with Open-SSH 8.8 due to underlying library bug.

## Publish vs Build

In a nutshell, if you have a web project (Blazor, ASP .NET) that requires a static assets folder (`wwwroot`) then enable publishing (`Tools -> Settings -> VsRemoteDebugger -> Local Machine Settings -> set publish to true`). 

Publishing is performed by invoking an external console and executing `dotnet publish` in the context of the current solution. This had to be done this way due to some inherit problems in Visual Studio's API.

By default the publish flag is set to false, in which case we're programatically triggering a Build using Visual Studio's API. This method is more robust, but does not copy the static assets required for web projects. 

## Demo

![](VSRemoteDebuggerDemo.gif)
