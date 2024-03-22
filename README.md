# VL.Audio.LTC
A set of nodes to encode and decode [Linear (or Longitudinal) Timecode (LTC)](https://en.wikipedia.org/wiki/Linear_timecode) in [VL.Audio](https://github.com/vvvv/VL.Audio).

Code borrowed from [VVVV.Audio](https://github.com/tebjan/VVVV.Audio) by [Tebjan Halm](https://github.com/tebjan).
Depends on [libtc](https://github.com/x42/libltc) by [Robin Gareus](https://github.com/x42) and [LTCSharp](https://github.com/elliotwoods/LTCSharp) (enclosed) by [Elliot Woods](https://github.com/elliotwoods).


## Using the library
In order to use this library with VL you have to install the nuget that is available via nuget.org. For information on how to use nugets with VL, see [Managing Nugets](https://thegraybook.vvvv.org/reference/hde/managing-nugets.html) in the VL documentation. As described there you go to the commandline and then type:

    nuget install VL.Audio.LTC


Try it with vvvv, the visual live-programming environment for .NET  
Download: http://visualprogramming.net

## Some Remarks

LTC only supports framerates of 24, 25 and 30 that are tradionally encoutered in the context of film. 
As long as software runs at those framerates all should be fine and dandy, seconds will be split into [0 ... 23], [0 ... 24], [0 ... 29] frames.

60 (or 50) fps like you'd more likely use in real time media are not supported though. You need to pick a LTC standard that is a divisor of your desired framerate.
The count of frames returned will then be multiplied by the result of that division. So for a desired framerate of 60 you'd pick an LTC standard that uses 30 fps,
the decoder will then ideally return the following frames [0,0,1,1,2,2,3,3 ... 29,29] and you need to account for that.
But here things can get finicky. During testing I encountered for example the following [0,0,1,1,1,2,3,3 ... 29,29] and [wirmachenbunt](https://wirmachenbunt.de/) even reported that some frames 
totally got lost like [0,0,1,1,1,1,3,3 ... 29,29]. From what I can tell this was happening when using WASAPI or ASIO with a audio buffer size >256. 
Might be hardware and/or driver dependend. So make sure to check the continuity of the incoming frames (by queuing them for example).


### License
#### [LGPL-3.0](https://github.com/bj-rn/VL.Audio.LTC/blob/master/LICENSE)
VL.Audio, VVVV.Audio and libtc are released under LGPL-3.0 license. LTCSharp is released under MIT license.


