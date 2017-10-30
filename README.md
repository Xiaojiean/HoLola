# HoLola - Holographic Data Visualization for Lola

NOTE: Please use `--recursive` when cloning this repo to pull in submodules.

## Dependencies

* [Visual Studio 2017](https://developer.microsoft.com/en-us/windows/mixed-reality/install_the_tools), including:
  * .NET Framework 3.5 development tools
  * .NET Portable Library targeting pack
  * Visual C++ runtime for UWP
  * Visual Studio Tools for Unity (will be installed by Unity)
  * Windows 10 SDK for UWP: C# and C++
* [Unity 5.6.2f1](https://store.unity.com/) or greater (earlier versions may build the project, but contain bugs in some APIs!)
* [am2b-iface]() (included as a submodule-- either clone this repository with `--recursive` or run `git submodule update --init` after cloning to obtain the latest compatible version)

## Build Instructions

### Building & Deploying the Unity Project

#### Normal build/deploy

If the following steps don't work, please follow the [complete instructions here](https://developer.microsoft.com/en-us/windows/mixed-reality/exporting_and_building_a_unity_visual_studio_solution).

1. Open the Unity project in Unity (folder HoLola)
2. `File->Build Settings...`
3. Double-check that the **PLatform** is set to **Windows Store** and that at least one scene is listed under **Scenes In Build**
4. Click the **Build** button. You will be asked to choose a directory to put the binaries in (e.g `.\Build\`).
5. After the build completes, navigate to the build directory you selected and open `HoLola.sln` in Visual Studio (*NOTE: This is **not** the same `HoLola.sln` as you will find in the Unity project root directory!)
6. Set your CPU architecture to `x86`, then set the target (next to the green arrow) to either `Device` (if the Hololens is connected via USB) or `Hololens Emulator`.
7. Press `F5` to deploy and launch the application.

#### Manual Deployment

If you do not have sufficient permissions on your build machine, or if you want to package the application for later redistribution or deployment, you can create an APPX package then side load it onto the device through the web interface.

**Starting from step 5 in the Normal build/deploy steps above:**
1. After building Holola.sln, right-click the Holola project in the solution explorer and select `Store...`
2. When asked if you want to submit the application to the Windows Store select **No**
3. Follow the steps in the package creation wizard. If you're testing a lot of builds, it may be useful to check the auto-increment version box to easily distinguish between new and old builds.
4. When asked about target platforms, the only one you need is `x86`
5. After the package is created, open a web browser and navigate to the Hololens' IP address (or to `http://127.0.0.1:10000` if the device is connected via USB)
6. In the browser select `Apps` on the left
7. In the App Manager, under *Install app* use the `Choose File` button to select the `.appx` file created after step 5
8. Click `Add dependency`, and add all of the `.appx` files under `<APPX Dir>/Dependencies/x86'
9. Click `Go`
10. Your app should now be installed! Bloom to find it in the app list, or use the *Insalled apps* menu in the App Manager to start it.
 
When distributing the application, simply zip up the directory containing the .appx package. In addition to the application this will also include debugging symbols (*.pdb), the application's dependencies (under the `Dependencies` directory--note that you can safely delete all but the `x86` dependencies if you want to save space), and some scripts which can make deploying the app from the commandline a little easier. The only files which are strictly necessary for the app to work on another device are the .appx and the files under `Dependencies\x86`.

#### Using the Hololens Emulator

https://developer.microsoft.com/en-us/windows/mixed-reality/using_the_hololens_emulator

### Building & Deploying the External DLLs

*NOTE:* Recent builds are checked in under the HoLola Unity project, so this should only be necessary if you are making changes to either DLL.

*NOTE:* Do not mix & match `Debug` and `Release` builds! If you want to deploy a `Release` version of the final Unity project, these DLLs should *also* be built with the `Release` target! For the time being, the checked-in versions are `Debug`.

Both the native and managed DLLs can be built from `LolaComms.sln` under [/net/LolaComms](./net/LolaComms).

**For deployment to Hololens or the emulator** you *must* build `x86`, *not* `x64`.

Resulting DLLs should be automatically deployed to the Unity project:

* Managed DLLs need to be copied to `<Unity Project Root>/Assets/Plugins/`
* Unmanaged (C++) DLLs need to be copied to `<Unity Project Root>/Assets/`

Once the DLLs have been updated in the Unity project, build from Unity and follow the instructions above to deploy to the device or emulator.

#### Do I have to do all of this every time? D:

##### If you have only changed existing C# Source code
You only need to rebuild from the final .sln. You do not have to re-export anything from Unity.

##### If you have modified Unity scene components, added or removed C# Source files
You need to rebuild **from Unity**, then rebuild the final .sln.

##### If you have modified anything in either LolaComms or LolaCommsNative
You need to rebuild `LolaComms.sln`, then rebuild from Unity, then rebuild the final .sln

## Debugging Tips

#### Log Files
Unity produces a log file containing all the output from `Debug.Log*` calls, callstacks if a Monobehavior hits an unhandled exception, etc. The log file is over-written every time the app launches, so make sure to download the logs after every test!

The log is located in: `User Files \ LocalAppData \ HoLola \ TempState`

#### ETW Events
ETW Trace data is available for some native components, which may help track down issues in the LolaComms DLLs. To enable them and get trace data, open the `Logging` section of the device portal, then under *Custom Providers* enter the GUID `8EB119A9-2FE5-46F5-998B-A396CA3F74B7` and click *Enable*. You should now see live event data at the bottom of the page when the application is active.

See [net/LolaCommsNative/LolaCommsNative/LolaCommsNative.man](net/LolaCommsNative/LolaCommsNative/LolaCommsNative.man) for event definitions.

*Note:* For some reason, provider info and event payloads are not appearing in the device portal. This *may* be related to [this issue](https://wpdev.uservoice.com/forums/110705-universal-windows-platform/suggestions/18591439-loggingchannel-not-showing-string-message-content), meaning a future update on the Hololens (and possibly a matching SDK update) could fix it, but it might also be that the configuration is not quite right on our end...

#### Debugging the external DLLs

1. Enable mixed-mode debugging in the project properties to examine behavior in LolaCommsNative
2. To catch exceptions and breakpoints in both DLLs, ensure `Enable Just My Code` is **NOT** checked in your debugging Options.

*Unfortunate side-effects:* Doing both of the above will also cause you catch a lot of exceptions in Unity and other components you likely don't want to deal with. This can make debugging tedious since Unity often throws a bunch of exceptions during initialization, so the recommendation is to keep your debugging setup for managed-only and Just My Code unless you *need* to dig into the communications DLLs.

## Known Issues

* Sometimes when deploying to the Hololens Emulator the app crashes on startup with a CLR error. Under a debugger this appears to be a fatal error in Windows.Mirage.dll, but there are also often bad references in Unty's D3D code (null textures, etc) which can lead to an early crash. Re-deploying without making any changes often fixes this.
