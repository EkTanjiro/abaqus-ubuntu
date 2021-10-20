# Abaqus 2019 on Ubuntu 20.04

## Intro

Ubuntu seems to not be officially supported by the Abaqus installation procedure. This guide shows how to install the necessary libraries and how to tweak the installation files in order to install Abaqus on Ubuntu 20.04. To successfully follow this guide you need writing privileges ('sudo').

*Note:*</br>
This guide should also work for Abaqus6.14 (changing accordigly file names and paths) and Ubuntu 18.xx adn 19.xx, although I haven't tested it.

## Install prerequisites

The standart Ubuntu release might not have one or more of the following libraries needed by Abaqus:

* libjpeg
* libstdc++ 4.7
* openmotif 2.3
* libgomp
* libXm4
* ksh
* redhat-lsb-core-4.x
* lsb-release-2.0
* libpng12-0

To install them open a terminal and execute the following commmand:

```bash
sudo apt update
sudo apt install csh tcsh ksh gcc g++ gfortran libstdc++5 build-essential make libjpeg62 libmotif-dev
```

Check the output of the installations and if there are errors try to install the ones that failed using the synaptic package manager. To install it, run:

```bash
sudo apt-get update
sudo apt-get install synaptic
```

Once installed, open it and look for the aforementioned libraries and install them.

## Modify the Linux.sh file

Since Ubuntu is not officially supported, trying to install Abaqus will result in an error. In order to fix it, prior launching
the installation, locate all the **Linux.sh** files in the Abaqus installation folders, delete their content and past the following
in each of them:

```sh
DSY_LIBPATH_VARNAME=LD_LIBRARY_PATH

which lsb_release
if [[ $? -ne 0 ]] ; then
  echo "lsb_release is not found: check in the PDIR the list of installed packages for servers validation."
  exit 12
fi

DSY_OS_Release="CentOS" #Override release setting, old: DSY_OS_Release=`lsb_release --short --id |sed 's/ //g'`
echo "DSY_OS_Release=\""${DSY_OS_Release}"\""
export DSY_OS_Release=${DSY_OS_Release}
export DSY_Skip_CheckPrereq=1 #Added to avoid prerequisite check

if [[ -n ${DSY_Force_OS} ]]; then
  DSY_OS=${DSY_Force_OS}
  echo "DSY_Force_OS=\""${DSY_Force_OS}"\", use it for DSY_OS"
  return
fi

case ${DSY_OS_Release} in
    "RedHatEnterpriseServer"|"RedHatEnterpriseClient"|"RedHatEnterpriseWorkstation"|"CentOS")
        DSY_OS=linux_a64;;
    "SUSELINUX"|"SUSE")
        DSY_OS=linux_a64;;
    *)
        echo "Unknown linux release \""${DSY_OS_Release}"\""
        echo "exit 8"
        exit 8;;
esac
```

Note the changes: 1) the release version was forced to be `"CentOS"` , and 2) checking of prerequisites was disabled.

## Run setup

Once all the prerequisites are installed and the installation files modified, it is possible to proceed with the installation:

```bash
cd path_to_abaqus_installation_folder\1
sudo ./StartGUI.sh
```

this will start the installation for all the Abaqus-related products. Skip the installation of the FlexNET server (this will cause an error) and proceed with the installation using the default locations for the software:

```bash
/usr/DassaultSystemes/SimulationServices/V6R2019x
/usr/SIMULIA/CAE/2019
```

Once the installation is complete you have to setup the FlexNET server. To do so, execute:
 `sudo gedit /usr/DassaultSystemes/SimulationServices/V6R2019x/linux_a64/SMA/site/custom_v6.env`

and then change the license server type to FLEXNET and the abaquslm_license_file to the one of your license (something like ###@your_company_domain), as follows:

```bash
    doc_root="http://help.3ds.com"
    license_server_type=FLEXNET
    abaquslm_license_file="27800@your_company_domain"
```

## Make abaqus command available from any directory

In order to run Abaqus from any location, execute the following comand: `sudo ln /var/DassaultSystemes/SIMULIA/Commands/abq2019 /usr/bin/abaqus`

## OpenGL Errors

If there is any warning regarding OpenGL in the terminal during start, simply run Abaqus with -mesa parameter:

```bash
    abaqus cae -mesa
    abaqus view -mesa
```

## Shortcuts

Create shortcut for CAE:

```sh
gedit ~/.local/share/applications/abaquscae.desktop
```

```
    desktop
    [Desktop Entry]
    Type=Application
    Version=1.0
    Name[en_US]=abaquscae
    Icon=/opt/SIMULIA/CAE/2019/linux_a64/CAEresources/graphic/icons/icoR_application.png
    Exec=sh -c "export FILE=%u && cd $(dirname $FILE) && abaqus cae database=$FILE -mesa"
    # Exec=abaqus cae database=%u -mesa
    Terminal=true
    Categories=Science;
```

Create shortcut for Viewer:

```sh
gedit ~/.local/share/applications/abaqusviewer.desktop
```

```sh
    desktop
    [Desktop Entry]
    Type=Application
    Version=1.0
    Name[en_US]=abaqusviewer
    Icon=/opt/SIMULIA/CAE/2019/linux_a64/CAEresources/graphic/icons/icoR_application.png
    Exec=sh -c "export FILE=%u && cd $(dirname $FILE) && abaqus viewer database=$FILE -mesa"
    # Exec=abaqus view database=%u -mesa
    Terminal=true
    Categories=Science;
```

Then make *.desktop executable:  
*Properties-->Permissions-->Allow executing file as program*.

## MIMETYPES & ICONS

Delete LibreOffice mime type for ODB:

```sh
sudo gedit /usr/share/mime/packages/libreoffice.xml
```

and comment tag:

```xml
    <mime-type type="application/vnd.sun.xml.base">
```

Add Abaqus mime types:

```sh
sudo gedit ~/.mime.types
```

Add lines:

```sh
    application/abaquscae				cae
    application/abaqusviewer			odb
```

Create file:

```sh
    sudo gedit /usr/share/mime/packages/abaquscae.xml
```

```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <mime-info xmlns="http://www.freedesktop.org/standards/shared-mime-info">
        <mime-type type="application/abaquscae">
            <comment>Abaqus CAE</comment>
            <glob pattern="*.cae"/>
        </mime-type>
    </mime-info>
```

Create file:

```sh
    sudo gedit /usr/share/mime/packages/abaqusviewer.xml
```

```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <mime-info xmlns="http://www.freedesktop.org/standards/shared-mime-info">
        <mime-type type="application/abaqusviewer">
            <comment>Abaqus Viewer</comment>
            <glob pattern="*.odb"/>
        </mime-type>
    </mime-info>
```

Then update MIME database:

```sh
    sudo update-mime-database /usr/share/mime
```

Additionally install "MIME Type Editor" via software and edit file associations.

Create icons for Abaqus file types:

```sh
    sudo cp 3DS.svg /usr/share/icons/Humanity/mimes/256/application-abaquscae.svg
    sudo cp 3DS.svg /usr/share/icons/Humanity/mimes/256/application-abaqusviewer.svg
    sudo gtk-update-icon-cache /usr/share/icons/Humanity -f
```

## FONTS

Open Abaqus CAE and check if it uses *Courier-New* and *Verdana* fonts:

```sh
    lsof -c ABQcaeK | grep fonts
```

Increase font size: <https://kb.dsxclient.3ds.com/mashup-ui/page/resultqa?id=QA00000008675e>

Use *xfontsel* to check if chosen font is available on system.

Install MS core fonts and RESTART:

```sh
    sudo apt install ttf-mscorefonts-installer
```

<!--

    sudo apt install xfonts-utils xfonts-100dpi xfonts-encodings

-->

Edit dictionaries:

```sh
    /opt/SIMULIA/CAE/2019/linux_a64/SMA/Configuration/Xresources/en_US/en_US_Dict.py
    /opt/DassaultSystemes/SimulationServices/V6R2019x/linux_a64/SMA/Configuration/Xresources/en_US/en_US_Dict.py
```

and increase font size:

```python
    masterDict[r'-*-courier-<weight>-<slant>-normal--10-*'] = r'-*-courier-<weight>-<slant>-normal--20-*'
    masterDict[r'-*-courier-<weight>-<slant>-normal--12-*'] = r'-*-courier-<weight>-<slant>-normal--25-*'
    masterDict[r'-*-courier-<weight>-<slant>-normal--14-*'] = r'-*-courier-<weight>-<slant>-normal--34-*'
    masterDict[r'-*-helvetica-<weight>-<slant>-normal--10-*'] = r'-*-helvetica-<weight>-<slant>-normal--20-*'
    masterDict[r'-*-helvetica-<weight>-<slant>-normal--12-*'] = r'-*-helvetica-<weight>-<slant>-normal--25-*'
    masterDict[r'-*-helvetica-<weight>-<slant>-normal--14-*'] = r'-*-helvetica-<weight>-<slant>-normal--34-*'
```

For big displays font HELVETICA and sizes 17-20-25 are more appropriate:

```python
    masterDict[r'-*-courier-<weight>-<slant>-normal--10-*'] = r'-*-helvetica-<weight>-<slant>-normal--17-*'
    masterDict[r'-*-courier-<weight>-<slant>-normal--12-*'] = r'-*-helvetica-<weight>-<slant>-normal--20-*'
    masterDict[r'-*-courier-<weight>-<slant>-normal--14-*'] = r'-*-helvetica-<weight>-<slant>-normal--25-*'
    masterDict[r'-*-helvetica-<weight>-<slant>-normal--10-*'] = r'-*-helvetica-<weight>-<slant>-normal--17-*'
    masterDict[r'-*-helvetica-<weight>-<slant>-normal--12-*'] = r'-*-helvetica-<weight>-<slant>-normal--20-*'
    masterDict[r'-*-helvetica-<weight>-<slant>-normal--14-*'] = r'-*-helvetica-<weight>-<slant>-normal--25-*'
```

<!--
#Enable fixed fonts in Ubuntu:
#http://marklodato.github.io/2014/02/23/fixed-fonts.html
-->

## FORTRAN

To use *gfortran* edit: `/opt/DassaultSystemes/SimulationServices/V6R2019x/linux_a64/SMA/site/lnx86_64.env`

as follows:

* Change ifort to gfortran on the fortCmd line and make sure that g++ is on the cppCmd line
* Remove gnu-incompatible compiler flags from the compile_fortran line (-V, -auto, -mP2OPT_hpo_vec_divbyzero=F, -extend_source, -fpp, -WB)
* link_sl: remove command line options ‘-V’, ‘-cxxlib’, ‘-threads’, ‘-parallel’, ‘-shared-intel’
* add '-lgfortran' to the link_sl and link_exe lines

<!--
How to avoid strange Fox-windows? After each start delete file $HOME/abaqus_2019.gpr?
-->

## KWON ISSUES AND POSSIBLE SOLUTIONS

### Run bash and not dash

If you get an error starting like:

```bash
`./StartGUI.sh: 4: typeset: not found CurrentMediaDir initial="" ./StartGUI.sh: 6: typeset: not found ./StartGUI.sh: 8: [[: not found ./StartGUI.sh: 10: [[: not found
...
```

Check that you run bash and not dash (dash is default in Ubuntu, and the Abaqus Installation script uses bash-specific commands).

One way is to search as root (or via sudo) for `/usr/bin/sh` and make it a symlink to `/usr/bin/bash` .

You should not forget to re-link `/usr/bin/sh` to dash after the Abaqus installation.

### License Validation

If you cannnot validate the license, probably your Ubuntu is not in English. You ran the program from the terminator with this code:

```bash
LANG=en_US.utf8 abaqus cae -mesa
```

## Credits

This guide is based on:

* <https://github.com/Kevin-Mattheus-Moerman/Abaqus-Installation-Instructions-for-Ubuntu>
* <https://github.com/imirzov/Install-Abaqus-2019-on-Ubuntu-18.04-LTS>
