## Visual Studio ##
Currently, only Microsoft Visual Studio 2015 Update 1 or older can be used to compile BitShares-core. Visual Studio 2015 Update 2 or newer will not work. This is being actively worked on, and should be resolved soon. Until then, you may install Microsoft Visual Studio 2015 Update 1 by following the directions here.

It is difficult to find an .iso of Visual Studio Update 1. However, it is possible to install that specific version by downloading an .iso of a newer version of Visual Studio 2015, and start the installation with a special command-line parameter. Here is how to do it:

Download the .iso "Visual Studio 2015 with Update 3" for x64 (community edition is fine). The filename Microsoft gives is "mu_visual_studio_2015_update_3_x86_x64_dvd_8923065.iso". Mount the .iso and run:

``ServicedSetup\1033\vs_community.exe /OverrideFeedURI http://download.microsoft.com/download/3/2/A/32A1974F-D236-43C1-8981-97DDCBAEF14A/20160225.3/enu/feed.xml``

More information about installing a specific version of Visual Studio 2015 is available [here](https://docs.microsoft.com/en-us/visualstudio/install/how-to-install-a-specific-release-of-visual-studio?view=vs-2015).

As you step through the installer wizard, make sure to install C++ as a programming language (The default install does not include it).

On completion, you may get errors about being unable to install some runtime libraries. That is okay.

For more information on compiling BitShares Core, [click here](BUILD_WIN32.md).