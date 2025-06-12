==============================================================================
                README - Client/Workstation Setup
==============================================================================

The PaperCut Client program is optional client software that displays
a user's current account balance in a popup window. It is also responsible
for displaying popup messages and the shared account selection popup if the
user has been granted access to this feature.

The client software is made available to other network users via a read-only
share called "PCClient" setup on this computer as part of the install process.

This client is usually configured to launch automatically when the user logs
onto the network.  This document details various installation and deployment
options.

The client software is available for:
    Microsoft Windows 64bit (see "Windows 32bit Systems" below for 32bit support)
    Apple Mac OS X (10.7+)
    Linux/Unix - Requires Java 11

WINDOWS 32bit SYSTEMS

In V19 to support the latest version of Java, PaperCut became 64bit only software.
To ensure that 32bit workstations can continue to operate with PaperCut we have
provided a 32bit client download and installation instructions via the following
knowledgebase article:

https://www.papercut.com/kb/Main/EndOfLife32BitOperatingSystems


WINDOWS 64bit SYSTEMS

The client can be deployed on Windows workstations using one of two methods:
    - Zero-Install Method (64bit workstation only)
    - Local Install

The zero-install method involves running the executable program directly off
the server without any need for local install.  This is the recommended method
as it ensures the client software is up-to-date.

The local install method copies the client program files to the workstation's
local hard disk.  This approach is recommended for Laptop/Notebook systems
that are not centrally managed and permanently connected to the network.


ZERO-INSTALL METHOD (64bit workstation only)

The client can simply be run directly from the PCClient share setup on the
server.  Two executables provide this launch functionality:
    pc-client.exe
    pc-client-local-cache.exe


pc-client.exe will launch the client directly off the network share.  The
"local-cache" version, pc-client-local-cache.exe, is a smarter version that
first copies itself and associated files to the local drive and launches
itself from there.  The local-cache version has the advantage that any future
startups will use the local copy and hence minimize network traffic.  The
cache is self-managing and kept up-to-date ensuring that any new versions of
the client are automatically and transparently copied down to the client.

Using pc-client-local-cache.exe is recommended on large networks.  It does
however require a globally writable cache directory.  By default the cache
is created in a directory on the system drive (normally C:\Cache).  An
alternate cache can be specified with the "--cache" directive (see below).
Administrators should ensure that standard users have write access to the
system drive, or manually create the cache directory if required.


Typical methods used to start the client executable include:

*Logon Script*

  Start the program by adding a line in the users' logon script:

      cmd /c "start \\servername\PCClient\win\pc-client-local-cache.exe --silent"


*Group Policy*

  Use the group policy template to define the pc-client-local-cache.exe as a
  program to be run on Logon. This is done via a policy located under:

    Computer Config->Administrative Templates->System->Logon->Run these
    programs at user logon

  Alternatively a line can be added to the user's logon script under:

    User Configuration->Windows Settings->Scripts (Logon/Logoff)->Logon

  to run the program as part of the logon process.

   Batch File:
     cmd /c "start \\servername\PCClient\win\pc-client-local-cache.exe --silent"

   VBScript:
     Set WshShell = WScript.CreateObject("WScript.Shell")
     WshShell.Run "\\server_name\PCClient\win\pc-client-local-cache.exe --silent"


  Note: The group policy option at:

      User Configuration -> Administrative Templates -> System -> Scripts ->
       Run logon scripts synchronously

    should be set to 'Disabled' (the default).  Enabling this option may
    result in the client tool being terminated before or shortly after login.
==============================================================================

*All Users StartUp Folder*

  On each workstation create a shortcut to
    \\servername\PCClient\win\pc-client-local-cache.exe
  in the "All Users" Startup folder.  This folder is usually located at
  C:\Documents and Settings\All Users\Start Menu\Programs\Startup.  The program
  will then run whenever a user logs into the workstation.

*Starting via the Windows Registry (Advanced)*

  Use a logon script or otherwise to add a registry key to each workstation.
  Create a key called "UIT" with a value of
      "\\servername\PCClient\win\pc-client-local-cache.exe"
  in the root
      HKLM/Software/Microsoft/Windows/CurrentVersion/Run

Special Note for Windows XP SP2:  Windows XP may prevent the running of remote
executables hosted on a server's share.  If users receive a security warning,
add the server to the workstation's list of "Trusted sites" via Internet Options
in the control panel and reboot.


LOCAL INSTALL METHOD 1

The client may be installed locally on the workstation by running the install program:
    Windows:  win/client-local-install.exe
    Mac:      mac/client-local-install
Administrator privileges are required for the install. This method is recommended
on single-user Laptop/Notebook systems not permanently connected to the network.


LOCAL INSTALL METHOD 2 (Advanced)

Network administrators may also choose to install the client locally via a
script or batch file.  The client can be installed via an "XCOPY" method
copying all files from the [app-dir]\client directory, however the zero-install
method is recommended.


GROUP POLICY INSTALL (Advanced - For Windows System Administrators)

An MSI package is also included for installation over an Active Directory Group Policy.
This installer can be in the Windows client directory:
    [app-dir]\client\win\pc-client-admin-deploy.msi

For detailed instructions see the user manual in the section:
   Services for Users->User Client->User Client Deployment.

Please note that using the local-cache version of the User Client is still our recommended
mode of distribution, as all updates are automatically pushed to client machines, no
further administration is required.



OTHER OPERATING SYSTEMS (Mac, Linux and Unix)

Further information regarding installing the client software on Mac, Linux and
Unix systems is covered in detailed in the user manual.  The user manual is
available in PDF form or via the Help link inside the application.


COMMAND LINE OPTIONS

The client can be started with command line options to modify the start-up
behaviour.

    --silent

        The silent option tells the client not to report errors if it has
        problems connecting to the server.  If the server is unavailable at
        time of startup (e.g. the client is not connected to the network),
        or if the user does not currently exist in the database, the client
        will simply sleep waiting for the condition to change.


    --minimized

        The minimized option tells the client to start minimized.  On windows
        the client will be minimized to the task tray.


    --debug

        When enabled, debugging information is written to user-client.log,
        located in the user's home directory.


    --user <username>

        Use the given username to query the user balance, etc.  This can be
        useful if the user is logged into a machine as a different user than
        they are authenticated to the server/printers.  For example, if a user
        is using a laptop that is not a part of the domain.


    --cache <cache directory>

        This argument is actioned by pc-client-local-cache.exe.  It defines the
        location of the globally writable cache directory on the system's local
        hard drive.  The cache is used to minimize network traffic on future
        launches.  The default location is C:\Cache.  Standard users will
        require WRITE and READ access to this directory.


    --neverrequestidentity

        The client will use the username of the logged in user to identify itself
        with the server.  In a domain environment, users always login using their
        network identity and the names will always match.  However on non-domain
        systems where local accounts are used (e.g. Laptops), these names may
        not match.  The client will display a popup requesting the user to
        confirm their identity.  This option will suppress this dialog.


    --windowposition <position>

        Specify where the client window should appear. The valid options include
        top-left, top-right, bottom-left or bottom-right. In addition to the above
        set of fixed positions, co-ordinates of the window can also be specified
        by setting the <position> parameter to XY<x>,<y>. The <x> value sets the
        x co-ordinate of the window (if negative the value indicates the distance
        from the right of screen). The <y> value sets the y co-ordinate of the
        window (if negative the value indicates the distance from the bottom
        of screen).


    --windowtitle <title>

        Allows the window title to be customized. If the <title> includes {0}
        then this will be replaces by the user's username.


    --default-selection <option>

        Specifies the default selected option on the account selection popup.
        This option is useful when one particular charging option is the most
        common, but other options are required on occasion.
        For example, applying a default selection of charge-account-list ensures
        that the option Charge to shared account is selected, and the Account
        list is highlighted. In this case, the keyboard can be used to quickly
        navigate the account list, saving a few clicks of the mouse for every
        print.
        Valid options include: charge-personal, charge-account-list,
                               charge-account-pin and print-as-user.


    Please see the manual for more options.


CONFIGURATION

The URL that appears in the window may be modified under
Options->Config Editor under the admin login on the server.  The appropriate
keys are:
    client.config.link-text
    client.config.link-url
