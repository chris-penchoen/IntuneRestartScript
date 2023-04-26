SYNOPSIS
    Create nice toast notifications for the logged on user in Windows 10.

DESCRIPTION
    Everything is customizeable through config-toast.xml.
    Config-toast.xml can be locally, hosted online in blob storage or set to an UNC path with the -Config parameter.
    This way you can quickly modify the configuration without the need to push new files to the computer running the toast.
    Can be used for improving the numbers in Windows Servicing as well as kindly reminding users of pending reboots (and a bunch of other use cases as well).
    All actions are logged to a local log file in AppData\Roaming\ToastNotificationScript\New-ToastNotification.log.

PARAMETER Config
    Specify the path for the config.xml. If none is specified, the script uses the local config.xml

NOTES
    Filename: Lululemon-ToastNotification.ps1
    Version: 2.3.1
    Author: Chris Penchoen
    GitHub: https://github.com/chris-penchoen
    

    Version history:

    1.0   -   Script created
                - Original Author: Martin Bengtsson
                - Blog: www.imab.dk
                - Twitter: @mwbengtsson

    1.1   -   Separated checks for pending reboot in registry/WMI from OS uptime.
              More checks for conflicting options in config.xml.
              The content of the config.xml is now imported with UTF-8 encoding enabling other characters to be used in the text boxes.

    1.2   -   Added option for personal greeting using given name retrieved from Active Directory. If no AD available, the script will use a placeholder.
              Added ToastReboot protocol example, enabling the toast to carry out a potential reboot.

    1.3   -   All text elements in the toast notification is now customizeable through the config.xml
              Expanded the options for finding given name. Now also looking in WMI if no local AD is available. 
              Added Get-WindowsVersion function: Testing for supported Windows version
              Added Test-WindowsPushNotificationsEnabled function: Testing for OS toast blockers
              Added some more detailed logging
              Added contributions from @SuneThomsenDK @ https://www.osdsune.com
                - Date formatting in deadline group
                - Fixed a few script errors
                - More text options

    1.4   -   Added new feature for checking for local active directory password expiration. 
              If the password is about to expire (days configured in config.xml), the toast notification will display reminding the users to change their password

    1.4.1 -   Get-ADPasswordExpiration function modified to not requiring the AD Powershell module. Thank you @ Andrew Wells :-)
              Improved logging for when no toast notifications are displayed
              More commenting
              
    1.4.2 -   Bug fixes to the date formatting of ADPasswordExpiration now correctly supporting different cultures

    1.4.3 -   Some minor corrections to the get-givenname function when retreiving first name from WMI and registry
              Moved the default location for New-ToastNotification.log file to the user's profile
              Added contribution from @kevmjohnston @ https://ccmcache.wordpress.com
                - Added function for retrieving deadline date and time dynamically in WMI with ConfigMgr

    1.5   -   Added new option to run task sequences (PackageID) directly from the toast notification action button. Enable the option <RunPackageID> in the config.xml
              Fixed a few script errors when running the script on a device without ConfigMgr client

    1.6   -   Added new option to run applications (ApplicationID) directly from the toast notification action button. Enable the option <RunApplicationID> in the config.xml
              Created Display-ToastNotification function
                - Displaying the toast notification as been trimmed and merged into its own function
              Created Test-NTsystem function
                - Testing if the script is being run as SYSTEM. This is not supported  
              Converted all Get-WMIObject to Get-CimInstance
                - Get-WMIObject has been deprecated and is replaced with Get-CimInstance
    
    1.7   -   Added multilanguage support. Thank you Matt Benninge @matbe
                - Script and config files now support multiple languages
                - Note that old config xml files needs to be updated to support this
                - Moved text values from option to the text-section for consistency

    1.7.1 -   Added 2 new options (LogoImageName and HeroImageName) to the config file, allowing switching of images more easily and dynamically

    1.8.0 -   Added support for using Windows 10 Toast Notification Script with Endpoint Analytics Proactive Remediation
                - Added support for having config.xml file hosted online
                - Added support for having images used in the script hosted online


              ** Most of the work done in version 2.0.0 is done by Chad Brower // @Brower_Cha on Twitter **
              ** I have added the additional protocols/scripts and rewritten some minor things **
              ** As well as added support for dynamic deadline retrieval for software updates **
              ** Stuff has been rewritten to suit my understanding and thoughts of the script **

    2.0.0 -   Huge changes to how this script handles custom protocols
              Added Support for Custom Actions/Protocols within the script under user context removing the need for that to be run under SYSTEM/ADMIN
                - <Option Name="Action" Value="ToastRunUpdateID:" />
                - <Option Name="Action" Value="ToastRunPackageID:" />
                - <Option Name="Action" Value="ToastRunApplicationID:" />
                - <Option Name="Action" Value="ToastReboot:" />
              Added Support to dynamically create Custom Action Scripts to support Custom Protocols
              Added Support for Software (Feature) Updates : Searches for an update and will store in variable
              Added new XML Types for Software Updates:
                - <Option Name="RunUpdateID" Enabled="True" Value="3012973" />
                - <Option Name="RunUpdateTitle" Enabled="True" Value="Version 1909" />
              Added support for getting deadline date/time dynamically for software updates
                - Configure DynamicDeadline with the UpdateID

    2.0.1 -   Updated custom action scripts!
                - Moved all custom action scripts into the user's profile in $env:APPDATA\ToastNotificationScript
                    - $env:ALLUSERSPROFILE was used previously. This is a bad location if device is used by multiple users due to permission issues
                - Updated all custom action scripts to invoke their respective action via WMI
                    - Rewritten all custom action scripts
                - Added logic allowing new custom action scripts to be created if necessary
                    - Now checks script version in registry 
                    - If newer version is available from the script, new custom action scripts will be created
                        - This allows me to make sure the relevant scripts are in place in case I change something along the way
                - Modified script output of custom script for RunPackageID to pick up Program ID dynamically
              Added support for getting deadline date/time dynamically for applications
                - Configure DynamicDeadline with the Application ID

    2.0.2 -   Fixed an error in the custom protocols
                - The path to the custom scripts was incomplete

    2.1.0 -   Added a second action button: ActionButton2
                - This allows you to have 2 separate actions. Example: Action1 starts a task sequence, action2 sends the user to a web page for more info
                - This will require new config.xml files
              Reworked Get-GivenName function
                - Now looks for given name in 1) local Active Directory 2) with WMI and the ConfigMgr client 3) directly in registry
                    - Now checks 3 places for given name, and if no given name found at all, a placeholder will be used
              Fixed CustomAudioToSpeech option
                - This part haven't worked for a while it seems
                - Only works properly with en-US language
              Added Enable-WindowsPushNotifications function // Thank you @ Trevor Jones: https://smsagent.blog/2020/11/12/prevent-users-from-disabling-toast-notifications-can-it-be-done/
                - This will force enable Windows toast notification for the logged on user, if generally disabled
                    - A Windows service will be restarted in the process in the context of the user

    2.2.0 -   Added built-in prevention of having multiple toast notifications to be displayed in a row
                - This is something that can happen, if a device misses a schedule in configmgr. 
                - The nature of configmgr is to catch up on the missed schedule, and this can lead to multiple toast notifications being displayed
              Added the ability to run the script coming from SYSTEM context // Thank you @ Andrew: https://twitter.com/AndrewZtrhgf :-)
                - This has proven to only work with packages/programs/task sequences and when testing with psexec. 
                - Running the script in SYSTEM, with the script feature in configmgr and proactive remediations in Intune, still yields unexpected results

    2.3.0 -   Added the Register-CustomNotificationApp function
                - This function retrieves the value of the CustomNotificationApp option from the config.xml
                    - The function then uses this name, to create a custom app for doing the notification.
                        - This will reflect in the shown toast notification, instead of Software Center or PowerShell
                        - This also creates the custom notifcation app with a prevention from disabling the toast notifications via the UI

    2.3.1 -   Updated and modified script and config.xml for Lululemon specific use case 
                - Hosted UptimeConfig.xml on StaticSave temporarily; needs to be hosted on Azure Storage 
                - Hosted lululemon-logo-48x48.jpg and ToastHeroImageDefault-2.jpg to Imgur; also need to move to Azure Storage 
