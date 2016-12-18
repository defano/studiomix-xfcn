# StudioMix XFCN
A HyperCard XFCN (external function) providing advanced audio controls and sound mixing.

### Wait, what?!

I developed this as high school student (circa 1995-96) as an extension to [HyperCard](http://hypercard.org) (a long-defunct program / application development environment that shipped with every Macintosh sold in the late eighties and nineties). I have archived its source code and documentation here for posterity. And nostalgia.

![Screenshot](/images/screenshot.png)

## Introduction

Since the inception of HyperCard, there has always been the play command. Unfortunately, HyperCard has never supported a command to play multiple sounds 	simultaneously. Now, with the functionality of StudioMix, HyperCard can now 	not  only mix multiple sounds but also allow controls for pitch, volume, background 	looping, pausing, and stopping all while HyperCard continues to process user actions.

## Requirements

* A Macintosh computer with a 68020 microprocessor or better.
* Apple Sound Chip (ASC) or the Enhanced Apple Sound Chip (EASC)
* Macintosh OS System software 6.0.5 or better.
* At least 2.5 megabytes memory. 5 megabytes required for most functionality.
* HyperCard version 2.1.0 or better.

## Error Codes

Error Code | Description
-----------|------------
`(-700) "Bad Parameter List."` | StudioMix was passed the wrong number of parameters for the given subcommand
`(-701) "Not Enough Memory."` | Not enough application memory is available to carry out the function of the specified subcommand. Increasing the amount of memory given to 			HyperCard may help.
`(-702) "Invalid or Bad Sound "snd" Resource."` | The sound resource you wish to play does not exist in any currently open 			resource forks. If the sound exists in another stack use HyperCard's "Start using" command.
`(-703) "Bad Parameter in Command."` | A parameter passed to a subcommand is invalid.
`(-704) "Digital Sound Studio General Failure."` | Multiple errors occurred when executing the subcommand. This error is 			generally proceeded by other internal errors such as `(-701)`. Sending `FLUSH` may help.
`(-705) "Error Sending Sound Manager Command."` | The sound manager returned an error when it was passed a command, 			this is an internal system software problem. Try using `FLUSH`.
`(-706) "Cant unload a resource in use"` | The memory manager was unable to safely dispose of a sound resource because it is still in use.
`(-707) "Couldn’t find a resource with that name"` | StudioMix could not find the specified resource. Try checking the spelling of the name, the resource may also lie in another file not currently open.
`(-709) "There is currently no sound resource cued in that channel"` | StudioMix could not play the cue in the channel because there is no sound cued in it. This may be caused from an UNLOAD command or the CUE was unable to load the sound.

## Special Considerations

During the external functions first call from HyperCard StudioMix creates three global variables to store memory data. The variables are labeled:

```
DSSInternalData0
DSSInternalData1
DSSInternalData2
```

These three variables hold a `LongInt` unsigned integer that holds the location of a sound 	channel. Since the StudioMix XFCN cannot hold global data between invocations, it stores data in the HyperCard environment. The `LongInt` stored is determined by:

```
DSSInternalData : LongInt;
DSSInternalData := LONGINT(POINTER(SndChannelPtr));
```

__WARNING:__

Changing any of the internal data variables will render StudioMix inoperable until a `FLUSH` subcommand is issued. Memory allocated by the old sound channels will be 	lost.

## Syntax

`StudioMix(COMMAND,[Param1],[Param2],[Param3],[Param4])`

StudioMix will return "empty" when it has successfully executed, otherwise when 	sound processing fails an error code will be passed to HypeCard  as its result.

`COMMAND` Specifies a StudioMix subcommand. The command is case-insensitive and may contain a variable in its place. `COMMAND` should contain one of the following subcommands:

----

### PLAY

Begins sound play in a given channel. `PLAY` requires parameters 1 - 3. Syntax for the play subcommand is:

`StudioMix(PLAY,ChanID,ResName,[InitVol,InitPitch])`

Parameter | Description
----------|------------
`ChanID` | Contains either string constant `Chan1` or `Chan2`. 						Only one sound can be played on a single channel at one time. A `PLAY` command passed on a busy sound channel is ignored by the XFCN. To play multiple sounds simultaneously call StudioMix twice; the first time using `Chan1` and the second time using `Chan2`.
`ResName` | Contains the full name of the resource to be played (this  will be the same name that is passed to the 						built in HyperCard `Play` command).
`[InitVol]` | (Optional) Contains the initial volume that the 						sound will  begin play at. `[InitVol]` should contain 						an integer between 0 and 255. 0 specifies the lowest 						(and muted) sound volume, whereas 255 specifies 						the loudest sound volume. Negative integers will be 						played at a 0 volume as well as integers greater 						than 255 will be given volumes of 255.
`[InitPitch]` | (Optional) Contains an integer between 1 and 10 specifying the rate at which the sound will begin play 						at. 1 signifies a slower play, 10 signifies a faster 						play. Passing a 5 as `[InitPitch]` will play the 						sound as its default rate contained in its sound 						header.

__NOTE:__ The InitVol parameter is required when using InitPitch. Do not include the InitPitch without also specifying an InitVol, otherwise the InitPitch will be misconstrued by the XFCN as InitVol.

__EXAMPLE:__

Starting sound play from StudioMix’s second sound channel with an initial volume of 200 at a default rate of 22khz:

```
Get StudioMix("PLAY","Chan2","MySndRes",200,5)
```

Starting sound play from StudioMix’s second sound channel ignoring optional parameters.

```
Get StudioMix("PLAY","Chan2","MySndRes")
```

----

### STOP 	

Stops the current sound being played on a given sound channel. The `STOP` subcommand syntax is as follows:

```
StudioMix("STOP",ChanID)
```

Parameter | Description
----------|------------
`ChanID` | Contains either `Chan1` or `Chan2`. If no sound is 						currently being played on the channel `STOP` is ignored.

__EXAMPLE:__

Stopping the sound playing on `Chan2`.

```
Get StudioMix("STOP","Chan2")
```

----

### PAUSE

Pauses the sound playing in a given channel. Syntax for the `PAUSE` subcommand is as follows:

```
StudioMix(PAUSE,ChanID)
```

Parameter | Description
----------|------------
`ChanID` | Contains either `Chan1` or `Chan2`. If no sound is 						currently paused on the channel given `PAUSE` is ignored.

__EXAMPLE:__

Pausing the sound playing on Chan1.

```
Get StudioMix("PAUSE","Chan1")
```

----

### RESUME

Resumes a sound that has been paused, or rate modified in a given channel. `RESUME` resets the rate of the currently playing sound to that specified in its sound header. Syntax for the `RESUME` subcommand is as follows:

```
StudioMix(RESUME,ChanID)
```

Parameter | Description
----------|------------
`ChanID` | Contains either `Chan1` or `Chan2`.

__NOTE:__ 	

The `RESUME` subcommand is also available as `UNPAUSE`.

__SPECIAL CONSIDERATIONS:__

`RESUME` will continue sound play at the sounds default rate. Therefore, a sound that is paused and then "resumed", will not necessarily play at the rate it was playing before a pause command, if the `RATE` command was sent to StudioMix prior to `PAUSE`. In order to counteract this problem, store the sound's rate in a variable (`theOldRate`), pause the sound, and instead of sending `Get 			studioMix(RESUME,Chan1)` use `Get studioMix(SETRATE,Chan1,theOldRate)`.

__EXAMPLE:__

Resuming play on the previously paused `Chan1`.

```
Get StudioMix("RESUME","Chan1")
```

----

### SETRATE

Set the rate of play (pitch) of the currently playing sound in a specified sound channel, hence altering the sounds pitch, octave, and duration. The `SETRATE` syntax is as follows:

```
StudioMix(SETRATE,ChanID,RateVar)
```

Parameter | Description
----------|------------
`ChanID` | Contains either `Chan1` or `Chan2`. If no sound is currently playing on the channel given `SETRATE` is ignored.
`RateVar` | An integer ranging from 1 to 10 where 1 is a slower rate and 10 is a faster rate. A `RateVar` less than 1 will be set to 1 as well a `RateVar` greater than 10 will be set to 10. Passing 5 to `RateVar` specifies that the sound should be played at the default rate.

__NOTE:__

Calling the `SETRATE` command on a channel that is paused will cause the channel to resume play at the new rate. See the `RESUME` command for more information about "unpausing" channels using `SETRATE`.

__EXAMPLE:__

Setting the rate of the sound playing on channel 2 (`Chan2`) to a pitch slightly higher than its recorded speed of 22khz.

```
Get StudioMix("SETRATE","Chan2",7)
```

----

### SETVOL

Sets the output volume of the currently playing sound on a specified channel. The `SETRATE` syntax is as follows:

```
StudioMix(SETVOL,ChanID,AmpVar)
```

Parameter | Description
----------|------------
`ChanID` | Contains either `Chan1` or `Chan2`. If no sound is currently playing on the channel given `SETVOL` is ignored.
`AmpVar` | An integer ranging from 0 to 255 where 0 is the lowest (muted) sound volume and 255 is the loudest volume. A 					`AmpVar` less than 0 will be set to 0 whereas a `AmpVar` greater than 255 will be set to 255.

__EXAMPLE:__

Setting the volume of the sound playing on channel 1 (`Chan1`) to its loudest volume.

```
Get StudioMix("SETVOL","Chan1",255)
```

----

### BEAT

Installs a sound resource as a background loop. The sound specified in `BeatParam` is looped indefinitely without a gap between sounds. The syntax for `BEAT` is as follows:

```
StudioMix(BEAT,BeatParam)
```

Parameter | Description
----------|------------
`BeatParam` | Contains either a valid command for the beat channel or a valid sound resource name. `BeatParam` can contain one of the following: `STOP` - Stops the currently playing loop sound. `DEVIATE` - Causes a deviation (cut) the in playing sound.

If the beat command is not passed one of above string constants it assumes that the BeatParam parameter contains a variable of `ResName` - The name of a valid sound resource to be looped.

__EXAMPLE:__

Starting a sound named "MyBeat" in an indefinite loop.

```
Get StudioMix("BEAT","MyBeat")
```

Deviating the current beat.

```
Get StudioMix("BEAT","Deviate")
```

Stopping the current beat.

```
Get StudioMix("BEAT","Stop")
```

----

### VARIATE

Inserts a "variation" in the currently playing beat. When `VARIATE` is called the beat loop is interrupted and the passed sound resource is played. When the variation resource is finished the beat continues playing.

```
StudioMix(VARIATE,SndRes)
```

Parameter | Description
----------|------------
`SndRes` | Contains the name of an available sound resource.

__EXAMPLE:__

Adding a variation in the currently playing beat.

```
Get StudioMix("VARIATE","MyVariationResource")
```

----

### ISDONE 	

Returns a boolean (logical) expression for the status of a given sound channel. The `ISDONE` subcommand returns TRUE if the if the sound channel has finished processing commands and `FALSE` if it hasn’t. `ISDONE` syntax is as follows

```
StudioMix(ISDONE,Chan)
```

Parameter | Description
----------|------------
`Chan` | Contains a channel identifier equal to `Chan1`, `Chan2` or `Beat`"

__EXAMPLE:__

Determine if the sound playing on channel 1 has finished

```
Get StudioMix("ISDONE","Chan1")
```

----

### MASTERVOL

Allows control of the Macintosh’s master volume. Changing the value of `MASTERVOL` changes the volume of all StudioMix channels, the alert sound and any other sounds currently playing. This has the same effect as changing the volume in the "Sound" control panel.

```
StudioMix(MASTERVOL,VolLevel)
```

Parameter | Description
----------|------------
`VolumeLevel` |	Contains an integer between 0 and 7; 0 being softest and 7 being loudest. The actual value used for volume is calculated by dividing `VolLevel` by seven. Therefore a `VolLevel` of 14 is the  same as a VolLevel of 7.

__EXAMPLE:__

Changing the volume of all sound channels on the Macintosh to a loud setting:

```
Get StudioMix("MASTERVOL","7")
```

----

### GETMASTERVOL

Returns the value set by `SETMASTERVOL`, the Macintosh master volume. The syntax for determining `GETMASTERVOL` is as follows:

```
StudioMix(GETMASTERVOL)
```

__EXAMPLE:__

Putting the master volume level into a container called `gMasterVol`

```
put StudioMix(GETMASTERVOL) into gMasterVol
```

----

### CUE

Loads a specified sound in a given sound channel so that there is no loading delay when the sound is played. Use `CUE` when sound play timing is imperative.

```
StudioMix(CUE,Chan,SndRes)
```

Parameter | Description
----------|------------
`Chan` | Contains a channel identifier equal to `Chan1`, `Chan2`.
`SndRes` | Contains the name of an available sound resource.

__EXAMPLE:__

Using the `CUE` command to preload a resource for quick timing control.

```
On PlayStudioMixCue

    If StudioMix(QUE,"Chan1","MySound") is not empty then
		    DoLoadError
	  end if

			-- Do your other routine

		Get StudioMix(PLAY,"Chan1","QUE","200,"5")

End PlayStudioMixCue
```

----

### UNLOAD 	

Unloads the memory obtained by a StudioMix `PLAY` or `CUE` command. When no sound has been cued in a channel and a `PLAY` command is executed, StudioMix allocates a bock of memory to hold the given sound resource. The next time StudioMix is called with `PLAY`, `STOP`, or `CUE` the old block of memory is purged. To purge a block of memory without calling `PLAY` or `STOP` use `UNLOAD`. The `UNLOAD` syntax is as follows:

```
StudioMix(UNLOAD,Chan)
```

Parameter | Description
----------|------------
`Chan` | Contains a channel identifier equal to `Chan1`, `Chan2` or `Beat`

__EXAMPLE:__

Unloading the sound resource that is no longer in use on channel 1

```
Get StudioMix("UNLOAD","Chan1")
```

----

### FLUSH

Flushes all memory from StudioMix sound channels. The `FLUSH` subcommand begins by quieting all StudioMix sounds currently playing, 			removes all used resources, and deletes the used sound channels.

The `FLUSH` subcommand is useful in fixing internal errors returned by StudioMix. Often after HyperCard receives an `errorCode` from StudioMix the XFCN will continue to fail in processing commands generally due to an internal memory error. This may be fixed without restarting HyperCard by sending a `FLUSH` command. The `FLUSH` subcommand uses no other parameters. Its syntax is as follows:

```
StudioMix("Flush")
```

__EXAMPLE:__

```
Get StudioMix("Flush")
```

----

### GESTALTSTUDIO

Determines if the current operating environment has the necessary hardware and software to run the StudioMix XFCN. If the current 			operating environment is suitable for StudioMix then `GESTALTSTUDIO` returns `true`, otherwise it returns `false`

StudioMix calls this routine each time a sound based subcommand is passed to it. If StudioMix determines that the operating environment is unable to run StudioMix it will emit the system sound and ignore any call to it.

`GESTALTSTUDIO` can be used as a preliminary measure to alert the user of hardware incompatibilities. Syntax is as follows:

```
StudioMix("GestaltStudio")
```

__EXAMPLE:__

```
Get StudioMix("GestaltStudio")
```

----

### OFFSET

Returns the number of bytes that a given sound resource’s data is offset from its resource header. Offset returns the same value as the toolbox trap `GetSndHeaderOffset`.  Syntax is as follows:

```
StudioMix("Offset",MySndResourceName)
```

`MySndResourceName` is a valid sound resource identifier.

__EXAMPLE:__

```
Get StudioMix("Offset",CoolSound)
```

----

### ENABLEBEEP & DISABLEBEEP

Respectively enables or disables the system beep alert sound on the Macintosh. A system beep that has been disabled will remain disabled until an `ENABLEBEEP` call is made or the Macintosh is restarted. Syntax is as follows:

```
StudioMix("EnableBeep")
StudioMix("DisableBeep")
```

__NOTE:__

When the system alert has been disabled no sound will be heard nor will the menubar be flashed when the `Beep` command is sent from HyperCard. In addition, no other applications will be able to use the alert sound.

__EXAMPLE:__

```
Get StudioMix("DisableBeep")
```

----

### CPUData

Returns information from the sound manager regarding sound process time given to StudioMix.

CPU Data returns 3 items separated by commas. Item 1 is the percentage of time currently be used by the CPU for sound manager processing. Item 2 is the maximum percentage that item may reach. If this is less that 100% it signifies that the CPU is also 			busy processing other (often time-based) data such as QuickTime. Item 3 returns the number of sound channels allocated. StudioMix always has three channels available, other applications may also have an allocated channel. Syntax is as follows:

```
StudioMix("CPUData")
```

__EXAMPLE:__

Answering information regarding CPU status:

```		
On DoTellCPU
    Put item 1 of StudioMix(CPUData) into CurTime
    Put item 2 of StudioMix(CPUData) into MaxTime
    Put item 3 of StudioMix(CPUData) into NumChannels

    Answer "CPU DATA:" & Return & "Processing time: " & CurTime & Return & "Max CPU Time: " && MaxTime & Return & "Allocated Channels: " && NumChannels

End DoTellCPU
```

----

### DISKPLAY

Determines whether built in play-from-disk routines should be used in StudioMix. Play from disk allows StudioMix to play large sounds with fairly little memory. However, as a tradeoff to memory, `DISKPLAY` may cause HyperCard to run slower than usual.

```
StudioMix("DISKPLAY",DiskBoolean)
```

Parameter | Description
----------|------------
DiskBoolean | Contains either `TRUE` or `FALSE`. True indicating that play from disk should be enabled.

__EXAMPLE:__

```
Get StudioMix("DiskPlay",True)
```

----

### GETDISKPLAY

Returns the value set by `DISKPLAY` indicating whether StudioMix is executing play from disk routines.

```
StudioMix("GETDISKPLAY")
```

__EXAMPLE:__

```
Get StudioMix("DiskPlay")
```
