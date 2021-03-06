scandiff
========

Scandiff is a PowerShell script to automate host discovery and scanning with nmap.  This script was written to perform nmap host discovery and port scanning from a remote network and send the results to a recipient through email.  After discovering and scanning hosts, scandiff performs an nmap ndiff on the output against previous results, 7zips all generated output, and optionally emails all output to a specified email address.

<b>IMPORTANT!</b> This performs a discovery scan looking for open ports first,before doing a scan using the default nmap portlist. This is done to speed up scanning across large IP spaces. If no open ports are discovered, this script will assume there are no live hosts and will exit.  This behavior can be suppressed by specifying -discovery 0.

There are several assumptions made about the installation path of nmap and 7Zip.  You'll need to modify these for your environment.

<b>Assumptions:</b>
* nmap: Assume this is installed at C:\Program Files (x86)\Nmap\nmap.exe
* ndiff: Assume this is installed at C:\Program Files (x86)\Nmap\ndiff.exe
* 7Zip: Assume this is installed at C:\Program Files\7-Zip\7z.exe
  
This script assumes you've copied out previous data if you want to keep it. Output files will be removed each time this script is run and replaced with new files with the exception of the diff file.  This file will be kept to compare against previous runs.
    
<b>Using scandiff:</b><br>
.\scandiff-0.9.ps1 -frequency [daily|weekly] -basename foo -outdir X:\path\to\output\directory -targets (nmap-style target specification or path to file containing targets) -email [0|1] -discover [0|1]

Example:<br>
./scandiff.ps1 -frequency daily -basename nmap-output -targets 192.168.1.10-25,scanme.nmap.org<br>
./scandiff.ps1 -frequency daily -basename nmap-output -targets c:\targets.txt

Scandiff takes a number of arguments.  The usage of each argument is described below:<p>
PARAMETER frequency<br>
Frequency is either daily or weekly.

Daily performs discovery using a limited set of ports and performs an nmap scan
using the default nmap TCP port list.

Weekly performs a discovery using a limited set of ports and performs an nmap 
scan using the full TCP port range and a limited set of UDP ports defined in
the script.

PARAMETER basename<br>
The basename parameter specifies the base name used for all output files.

PARAMETER targets<br>
This is the host or hosts to be scanned using the nmap-style host declaration,
e.g. 192.168.1.10-25, 192.168.1.0/24, hostname.domain.tld, or a comma-separated
combination of targets.

The path to an file containing hosts to be scanned can also be used. If the
target file is not in your current directory, you must pass the full path.

PARAMETER email<br>
This parameter determines whether the script should send email or not.
* 1 - Enable email output
* 0 - Suppress email output

PARAMETER outdir<br>
This parameter designates the output directory for all files generated by this script.
If not supplied, defaults to directory where this script resides.

PARAMETER discover<br>
This parameter controls whether discovery is performed.  Discovery is performed by default or if set to 1 or $True.  Set to 0 or $False to disable.

<b>Adapting scandiff to your environment:</b><br>
The frequency parameter is used to designate whether a limited-scope or larger-scope port scan should be performed.  Daily uses the default TCP and UDP port list.  Weekly specifies a list of specific UDP ports and TCP ports 1-65535.  These can be modified in the script as needed.

$scanOpts specifies the nmap scan type and other option flags.  Scandiff was written to use "-sS -Pn -T4 -v" for discovery scans and adds "-sV" to perform a version detection on discovered ports in the port scan section.

$zipPass specifies the password passed to 7Zip to encrypt output files to be attached to the email generated.  Currently, the script does not support suppressing 7Zip compression and encryption of output files.

The sendMail function can be modified to use any email server desired.  $EmailFrom, $EmailTo, $Subject, and $Body should be modified as you see fit.  The script was written to use gmail to send the email.  If you choose to use gmail, you will need to generate an app-specific password if you have 2-factor authenticaiton enabled on the account.

TO-DO:
* Ability to read config options from a config file
* Option to suppress zip output
* Add error handling
