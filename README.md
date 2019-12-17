# Setting up Visual Studio Code for ESP32 IDF (FreeRTOS)
Forked from Deous/VSC-Guide-for-esp32.
I'm doing the "For Dummies" edition. Removed the script esp32setenv.bat also

<p align="center">
    <img src="https://atmosphereiot.com/images/Third-Party-Logos/EspressifLogoFullGlow.png" alt="Logo" height=100 ><span width=50></span>
    <img src="https://code.visualstudio.com/assets/images/windows-logo.png" alt="Logo" width=100 >

  <h3 align="center">ESP32 IoT IDF</h3>

  <p align="center">
    Xtensa IoT framework.
  </p>
</p>

## Steps

- [Download and install VSC](#download-and-install-vsc)
- [Install VSC Extensions](#install-vsc-extensions)
- [Setup Toolchain](#setup-toolchain)
- [Setup and verify environment variables](#setup-and-verify-environment-variables)
- [Create project](#create-project)
- [Use Command Line](#Use-the-command-line)
- [Configure ESP32](#configure-esp32)
- [Cmake](#CMake)

## Other Notes
- [What happens on a Panic?](#What-happens-on-a-Panic?)
- [Before using Git](#Git-requirements)

## Download and install VSC

Below link will point to the latest version of the Microsoft code editor which is free and open source. <br/>
Go ahead and download the latest release.<br/>
<br/>
<a href="https://code.visualstudio.com/"><span>Visual Studio Code</span></a>

[To Top](#Steps)

## Install VSC extensions

After the installation open editor and go to 'Extensions' section to browse all available vsc extensions. <br/>
You need the following:<br/>

- C++
- Native Debug
- Code Outline or AL Outline (to display and navigate properties)
- GitLens (useful for reviewing the revision control information)
- Github Markdown Preview (hopefully saves re-working this file after uploading to Git)
- Open in Browser (Right click on html files...open - if you are doing a WebServer)
- VS Live Share - Obviously you are doing code reviews in teams. 

<img src="vsc-guide-1.jpg" >

[To Top](#Steps)

## Setup toolchain
[To Top](#Steps)
Next you need tools for compilation and linking your projects
Go to below link and follow instructions. I will link to the individual stages as well below:
I have not tested this process yet to see if there is more information 'for dummies' needed. The information
changes often enough and ESPRESSIF are getting decent documentation.
https://docs.espressif.com/projects/esp-idf/en/latest/get-started/

Step 1. Install prerequisites
https://docs.espressif.com/projects/esp-idf/en/latest/get-started/#step-1-install-prerequisites

Step 2. Get ESP-IDF
https://docs.espressif.com/projects/esp-idf/en/latest/get-started/#step-2-get-esp-idf

Step 3. Set up the tools
https://docs.espressif.com/projects/esp-idf/en/latest/get-started/#step-3-set-up-the-tools

Step 4. Set up the environment variables
https://docs.espressif.com/projects/esp-idf/en/latest/get-started/#step-4-set-up-the-environment-variables
## Setup and verify environment variables
Make sure all path and idf variables are present and pointing to the *right locations* as it had changed between for me compared to the original guide. 
IDF_PATH is cruicial. 
You can use run-command way to set them up with 
>setx IDF_PATH "...your path to esp-idf folder"

Example setup:
<br/>

This is the System Variables -> Path. I have the ESP-IDF and ESP8266 toolchain installed so need
paths to both of the build tools
<img src="vsc-guide-2.jpg" >
Here you can see the Environment Variables. See that I have two IDF_PATH variables referring to the 
esp-idf and esp8266. When I develop in 8266, the IDF_PATH needs to be to that SDK. I add a '32_' to the 
start of the ESP32 variable. I just change the Variable name, not the Value
For 8266 development, my variables are - 32_IDF_PATH, IDF_PATH
For ESP32 development, my variables are - IDF_PATH, 8266_IDF_PATH
respectively. This IDF_PATH is used in the SDK and the VSC. The first entry in Path (circled) is from 
the variable IDF_PATH
<img src="vsc-guide-3.jpg" >
This is what it looks like using the variable.
<img src="vsc-guide-3a.jpg" >

**Make sure you install Git for Windows**<br/>
<a href="https://git-scm.com/downloads">Git Client download page</a>

[To Top](#Steps)

## Create project

To create a project that will compile you need the following:

### .vscode folder must be present in the root project directory
Lets create the .vscode folder in the project folder if it is not there by magic.
Open vscode, FILE->OPEN FOLDER, select your project folder.
Next: Press F1 and type "c/Cpp: Edit Configurations"

### .vscode folder must have the following files:
 - <a href=".vscode/c_cpp_properties.json">c_cpp_properties.json</a>
 - <a href=".vscode/settings.json">settings.json</a>
 - <a href=".vscode/tasks.json">tasks.json</a>

Copy the files from this guide into the .vscode folder. Overwrite the c_cpp file.

### Time to fix c\_cpp\_properties.json
I am expecting that by the time you are using this, Espressif has changed the 
directory structure again. Open Powershell. Change to the *esp-idf\components* folder.
Enter this command 
>dir -Recurse -Directory | Select FullName | Export-Csv "folders.csv"

Open the folders.csv in notepad++. Do a regex Find and Replace on the following (replace with nothing). Repeat till end of document.
>^(?!.*include").*\r\n 

Perform a regular Find and Replace \ / (converting a backslash to a forward slash as '\' is a delimeter).
Perform a regular Find **C:/.../esp-idf** <- All of the bits in the middle please. Replace with 
>${env:IDF_PATH}

Seems that Export-Csv is not working so next do a find and replace on 
\r with ,\r (you will need to select "Extended")

Unfortunately the engineers at Espressif are not nice. So they include some of their .h files
in a different directory (twats) for example components/json/cJSON. Seriously WTF? 

Copy the c\_cpp\_properties.json file and rename to OLD\_c\_cpp\_properties.json as you will probably need to look back to find the non-include ending folders. In c\_cpp\_properties.json, delete all of the entries with {env:IDF_PATH} and replace these with your shiny new list. Check the OLD file for non-include ending folders and see if these are still valid. Copy these across as well.

This should also have (assumming the toolchain is installed as shown)
```json
  {"${workspaceRoot}",
  "${workspaceRoot}/main",
  "${workspaceRoot}/build/config",
  "${workspaceRoot}/build/include",
  "${workspaceRoot}/build",
  "${workspaceRoot}/components",
  "All the shiny new directories compliments of Powershell and Regex",
  "C:/Program Files/Espressif/ESP-IDF Tools/toolchain/xtensa-esp32-elf/include",
  "C:/Program Files/Espressif/ESP-IDF Tools/toolchain/xtensa-esp32-elf/include/c++/5.2.0",
  "C:/Program Files/Espressif/ESP-IDF Tools/toolchain/lib/gcc/xtensa-esp32-elf/5.2.0/include",
  "C:/Program Files/Espressif/ESP-IDF Tools/toolchain/lib/gcc/xtensa-esp32-elf/5.2.0/include-fixed"}
```

Now copy this entire array and replace the "Replace this section please." Press SAVE.
```json
  {
    "intelliSenseMode": "clang-x64",
            "browse": {
                "path": [
                  "Replace this section please",
                ],
            }
  }
```

### Lets grab an example project 
This will put the basic file and folder requirements into your project folder.

**I recommend this step to be followed otherwise the later "menuconfig" step will fail.**

Using FileExplorer open 
>[IDF\_PATH]\esp-idf\examples\wifi\getting_started\station

Copy the files in the station folder to your project folder.

[To Top](#Steps)

## Use the command line
If you use the built in Terminal and the compile fails, you will not be able to trace the fault as it closes.
- `idf.py -p COM3 flash` -- will compile and flash the board on port COM3
- `idf.py -p COM3 monitor` -- will display live serial monitor in console

if you want to flash and immediately monitor use 
- `idf.py -p COM3 flash monitor`
 
`build` will build project with changes <br/>
`fullclean` will erase old files and refresh the whole build for new compilation <br/>

Create batch files for these locally because we are lazy and just want to press f followed by enter.
I suggest this includes 'cls' so that you get a clean screen before compiling.
f.bat
cls
idf.py -p COM4 flash

Typical project file structure: <br/>

<img src="vsc-guide-5.jpg">

### Important note:
**To open your project you must choose 'Open Folder' option. Do not open a Workspace** <br/>

<img src="open-folder-menu.jpg">

[To Top](#Steps)

## Configure ESP32
Before compiling your project run the command to set up all necessary settings that IDF will use for your ESP32 chip.<br/>
Use Command window (Don't use the Integrated Terminal) at your project folder. Run ` idf.py menuconfig`<br/>
If you are not in Powershell but in cmd.exe - you can run `powershell idf.py menuconfig`<br/>
<br/>
Example:<br/>
<img src="vsc-guide-4.jpg" ><br/>
<img src="esp-configmenu.jpg" ><br/>

Hit ESC until menuconfig asks about saving the configuration.<br/>
Confirm save.<br/>
<img src="menu-done.jpg" ><br/>

[To Top](#Steps)

## CMake 
To use CMake environment your project must have CMakeLists.txt (just like Make files) in project directory
It usually contains the below lines. 

```
set(MAIN_SRCS
    main/spi_master_example_main.c
    main/pretty_effect.c
    main/decode_image.c)

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(spi_master)
```

This is your Make Project main file. You can change them to the project files you have. If you create more .c files you will need to add these to this top level CMakeLists.txt file. Change the parameter in project() if you want. 
Follow the examples provided in examples folder together with idf framework that you downloaded
Copy and Paste the example into your project directory so that you do not mess with the original code.
Overwrite the existing files. 

[To Top](#Steps)

## What happens on a Panic?
Check out this page. https://docs.espressif.com/projects/esp-idf/en/latest/get-started/idf-monitor.html
If you are debugging using the Monitor system, then the Guru Meditation Error will display the "Crash Location"
However if not and you get the raw Error Register dump, you can find out the location. Ignore the Register dump, look at the Backtrace.

>Backtrace: 0x400f360d:0x3ffb7e00 0x400dbf56:0x3ffb7e20 0x400dbf5e:0x3ffb7e40 0x400dbf82:0x3ffb7e60 0x400d071d:0x3ffb7e90

You want the following command to evaluate it. (PROJECT is in CMakeLists.txt project(PROJECT)).
The ADDRESS is directly from the backtrace. It is the 0x400...:0x.... value (0x[8digits]:0x[8digits]). This will
give you the function name of the crash. You may have to check more than one to find your code as the crash could
have occurred in the API due to your code.
>xtensa-esp32-elf-addr2line -pfiaC -e build/PROJECT.elf ADDRESS

For example:
>xtensa-esp32-elf-addr2line -pfiaC -e build/spi_maste.elf 0x400f360d:0x3ffb7e00

[To Top](#Other-Notes)

## Git requirements
Before you click on the SourceControl button and add files to a Git Repo, create a .gitignore file to the Project directory. Or copy this one on this Repo. This will save some hassle as you do not need the vscode database files in your repo. 

>.vscode/*.VC.*
>build/

Now go ahead and click on that SourceControl. Be a GIT.

[To Top](#Other-Notes)


Hopefully this works for you, if not please let me know and I will get an update going.
