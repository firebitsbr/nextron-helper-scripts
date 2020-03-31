# THOR Seed

A THOR and THOR Lite PowerShell Launcher

## What is THOR Seed

THOR Seed is a lightweight PowerShell script that facilitates the deployment of THOR in cases in which you can't or don't want to use an agent for a continous compromise assessment.

It retrieves a THOR package from a remote source, extracts it, runs it with certain settings and removes the temporary files afterwards.

You can decide if you want to send the logs to a remote log server, write them to a network share or write them to a local directory for later dispatching and collection.

The script itself writes an extensive log named `thor-seed.log`. You can deactivate the log with the `-NoLog` parameter or a preset in the respective script section.

## Requirements

- PowerShell version 3
- 50 MB of disk space (temporarily)
- Network connection to a THOR source (ASGARD, ASGARD Cloud, Nextron Customer Portal, THOR/THOR Lite as ZIP on a web server)

## Use Cases

From an on-premise ASGARD server

```console
thor-seed.ps1 -AsgardServer asgard1.internal
```

From ASGARD cloud instance

```console
thor-seed.ps1 -AsgardServer asgard23.cloud.nextron -ApiKey 12345678
```

From Nextron customer portal

```console
thor-seed.ps1 -UseCustomerPortal -ApiKey 12345678
```

With you custom THOR or THOR Lite package

```console
thor-seed.ps1 -CustomUrl https://web1.internal/thor/mythor-pack.zip
```

## Parameters

### -AsgardServer

Enter the server name or IP address of your ASGARD instance.

### -UseCustomerPortal

Use the official Nextron customer portal instead of an ASGARD instance. 

### -ApiKey

API key used when connecting to Nextron's customer portal instead of an ASGARD instance.

### -CustomUrl

Allows you to define a custom URL from which the THOR package is retrieved. Make sure that the package contains the full program folder, provide it as ZIP archive and add valid licenses (Incident Response license, THOR Lite license). THOR Seed will automaticall find the THOR binaries in the extracted archive.

### -SyslogServer

Enter the server or IP address of your remote SYSLOG server to send the results to.

### -OutputPath

A switching parameter that does not accept any value, but rather is used to tell the function to open the path location where the file was downloaded to.  

### -QuickScan

Perform a quick scan only. This reduces scan time by 80%, skipping "Eventlog" scan and checking the most relevant locations in "Filesystem" scan only.

### -NoLog

Do not write a log file in the current working directory of the PowerShell script named thor-seed.log.

## Preconfigured Variables

Sometimes you may not be able to pass the command line parameters or want to ship a preconfigured version of the script that just has to be run. In that case you can use the respective section in the PowerShell script and the so-called "config templates" that can be passed to a THOR scan using the `-t` parameter.

THOR Seed contains such a template as an inline string that can be modified by you before the scan. THOR Seed will eventually write the config as file into the temporary directory and pass it as parameter to the extracted THOR binary.

The config files have a YAML format. It is easy to read and write. All command line options that you can see when you run `thor64.exe --help` can be used in these config templates, but you have to use their long form, e.g. instead of using `-p C:\Windows` you have to use `path: C:\Windows` or instead of `-c 50` you'll have to use `cpulimit: 50`.

```powershell
# Predefined YAML Config
$UsePresetConfig = $True
# Lines with '#' are commented and inactive. We decided to give you 
# some examples for your convenience. You can see all possible command 
# line parameters running `thor64.exe --help`. Only the long forms of the
# parameters are accepted in the YAML config.
$PresetConfig = @"
module:
# - Autoruns
  - Rootkit
  - ShimCache
  - DNSCache
# - RegistryChecks
# - ScheduledTasks
  - FileScan
# - Eventlog
nofast: true       # Don't trottle the scan, even on single core systems
# nocolor: true    # Don't colorize the output
lookback: 3        # Log and Eventlog look back time in days
cpulimit: 50       # Limit the CPU usage of the scan
sigma: true        # Activate Sigma scanning on Eventlogs
# dumpscan: true   # Scan memory dump files found during Filescan
# printshim: true  # Output all SHIMCache entries
# nolog: true      # Don't write any file outputs (only makes sense when using SYSLOG)
# rebase-dir: \\server\share  # redirect all file outputs to this location
path:
    - C:\Temp
    - C:\Users\Public
"@
```

## Issues

### Custom THOR / THOR Lite Packages

Provide the package with license. This typically means that you have to repack the THOR package that you've downloaded. Extract it, add a license file (`.lic`) and then create a ZIP archive from that program folder. You can remove some folders to save disk space and reduce network load when running the script on thousands of systems.

The required files and directories are the following. You can safely remove all other files and directories.

```console
.
├── config
│   ├── directory-excludes.cfg
│   ├── false_positive_filters.cfg
│   ├── thor.yml
├── custom-signatures
│   ...
├── signatures
│   ├── iocs
│   │   ├── custom-evil-hashes.dat
│   │   ├── filename-characteristics.dat
│   │   ├── keywords.dat
│   │   ├── malicious-events.dat
│   │   ├── malicious-mutexes.dat
│   │   ├── malware-domains.dat
│   │   └── trusted-md5s.dat
│   ├── misc
│   │   └── file-type-signatures.cfg
│   ├── sigma
│   │   ...
│   ├── sigrev
│   └── yara
│       ├── thor-all.yas
│       ├── thor-keywords.yas
│       ├── thor-log-sigs.yas
│       ├── thor-process-memory-sigs.yas
│       └── thor-registry.yas
├── thor.exe
├── thor.exe.sig
├── thor64.exe
├── thor64.exe.sig
└── tools
    ├── UnRAR.exe
    ├── UnRAR.exe.sig
    ├── upx.exe
    └── upx.exe.sig
```

Importan: The listing above does not include the license file, which is obviously also required.

## Execution

You may run the script as follows:

```console
powershell.exe -ExecutionPolicy Bypass .\thor-seed.ps1 -CustomUrl https://my-webserver.internal/thor/thor-packed.zip
```