##Version 3.2.1

###1. Added

#####1. Inline Video Support

iOS now supports playing videos inline.  This should allow for a more desirable viewer experience.

#####2. Flag for Android Installed Application Tracking

On the Android platform, our analytics module accesses the installed apps on user's device and send it over to our server. This is really helpful to publishers to get better insights on the target audience. By default its turned on. You can turn it off by setting the flag enableAccessToInstalledApps to false in the following api.  By default, the flag is set to true.

R1ConnectPluginCommon.EnableAccessInstalledApps (boolean enableAccessToInstalledApps);
