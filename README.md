# VL.Audio.LTC
[![Nuget (with prereleases)](https://img.shields.io/nuget/vpre/VL.Addons?style=flat-square)](https://www.nuget.org/packages/VL.Audio.LTC)

A set of nodes to encode and decode [Linear (or Longitudinal) Timecode (LTC)](https://en.wikipedia.org/wiki/Linear_timecode) in [VL.Audio](https://github.com/vvvv/VL.Audio).

Code borrowed from [VVVV.Audio](https://github.com/tebjan/VVVV.Audio) by [Tebjan Halm](https://github.com/tebjan).
Depends on [libtc](https://github.com/x42/libltc) by [Robin Gareus](https://github.com/x42) and [LTCSharp](https://github.com/elliotwoods/LTCSharp) (enclosed) by [Elliot Woods](https://github.com/elliotwoods).


## Using the library
In order to use this library with VL you have to install the nuget that is available via nuget.org. For information on how to use nugets with VL, see [Managing Nugets](https://thegraybook.vvvv.org/reference/hde/managing-nugets.html) in the VL documentation. As described there you go to the commandline and then type:

    nuget install VL.Audio.LTC


Try it with vvvv, the visual live-programming environment for .NET  
Download: http://visualprogramming.net

## Some Remarks
LTC only supports framerates of 24, 25 and 30 FPS that are tradionally encoutered in the context of film/video. As long as software runs at those framerates all should be fine and dandy, seconds will be split into [0 ... 23], [0 ... 24], [0 ... 29] frames.

60 (or 50) fps like you'd more likely use in real time media are not supported though. You need to pick a LTC standard that is a divisor of your desired framerate. The count of frames returned will then be multiplied by the ratio of that division. So for a desired Mainloop framerate of 60 FPS you'd pick an LTC standard that uses 30 FPS, the decoder will then ideally return the following frames [0,0,1,1,2,2,3,3 ... 29,29] and you need to account for that (see Interpolate on the ToSeconds node).

But things can get finicky.
During testing I encountered for example the following [0,0,1,1,1,2,3,3 ... 29,29] and [wirmachenbunt](https://wirmachenbunt.de/) even reported that some frames totally got lost like [0,0,1,1,1,1,3,3 ... 29,29]. 
From what I can tell this is more likely to happen when using WASAPI or ASIO with a audio buffer size >256. Might be hardware and/or driver dependend. So make sure to check the continuity of the incoming frames (by queuing them for example).

TBH if you are working in a professional context and can spare the money using a dedicated timecode reader card might be the better option.

### Issues with C++/CLI
C++/CLI libraries in .NET Core need a shim called *Ijwhost.dll* for finding and loading the runtime. Currently there seems to be no "official" method on how to deal with this shim when having a C++/CLI library in a NuGet package. Some say to include it in the package other say  that this can easily lead to dll hell (what happens when to packages come with different versions of the shim?). Here are some related github issues with further info:

* [No Ijwhost.dll when referencing C++/CLR NuGet package from .NET 6 C# app](https://github.com/dotnet/sdk/issues/24310)
* [How to avoid double writes for ijwhost.dll in NuGet packages?](https://github.com/dotnet/sdk/issues/34213)
* [C++/CLI libraries may fail to load due to ijwhost.dll not being on the search path](https://github.com/dotnet/runtime/issues/38231)
* [Ijwhost.dll loading not always working for C++/CLI assembly](https://github.com/dotnet/runtime/issues/37972)
* [FileNotFoundException in .net 6](https://github.com/AmpScm/SharpProj/issues/25)

Tried the "workaround" of including a manifest file for *Ijwhost.dll*. But:
* vvvv only finds the shim when located alongside *LTCSharp.dll*, e.g. both files are in `VL.Audio.LTC\lib\net6.0-windows`
* when the shim is located in `VL.Audio.LTC\runtimes\win-x64\native` as suggested by some vvvv doesn't pick it up
* in neither case the shim gets copied to the output when exporting a document referencing *VL.Audio.LTC* when using normal [vvvv library workflow](https://thegraybook.vvvv.org/reference/extending/creating.html)

Thanks to [Elias](https://github.com/azeno) for a [PR](https://github.com/bj-rn/VL.Audio.LTC/pull/1) and helping me to figure out that the copying of the shim works when VL.Audio.LTC is referenced as "real" package.

Unfortunately this makes debugging of exporting executables during development a bit of a pita.

So for now one has to:

* Make changes
* right-click the VL.Audio.LTC project inside VS and select the `Pack` command
* this will create a nuget package inside the pkg folder, for example:
`X:\path\to\_vl-libs\VL.Audio.LTC\pkg`

To test the package you can run the following nuget command to install the package: 
`nuget install VL.Audio.LTC -source X:\path\to\_vl-libs\VL.Audio.LTC\pkg;nuget.org`
For testing the export you'll have to add to add your local path (`X:\path..`) to your *nuget.config* file which should be in
`C:\Users\yourusername\AppData\Roaming\NuGet`.

Alternatively move the created package from the `pkg` folder
into the root folder of `*VL.Audio.LTC* and start vvvv with the follwing arguments:
`start X:\vvvv\vvvv_gamma_6.2\vvvv.exe --package-repositories X:\path\to\_vl-libs\ --export-package-sources D:\path\to\_vl-libs\`


---


### License
#### [LGPL-3.0](https://github.com/bj-rn/VL.Audio.LTC/blob/master/LICENSE)
VL.Audio, VVVV.Audio and libtc are released under LGPL-3.0 license. LTCSharp is released under MIT license.


#### Credits
Initial development and release sponsored by [wirmachenbunt](https://wirmachenbunt.de/).
