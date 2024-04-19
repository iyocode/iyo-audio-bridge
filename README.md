# iyo-audio-bridge
Audio bridge for iyo vad pro plugin. This is a fork of the [BlackHole open source project](https://github.com/ExistentialAudio/BlackHole).
We exclused that project's commit history and changes can be seen in diff form with `git diff 036c0581 HEAD`.

## Build & Install
After building, to install BlackHole:

1. Copy or move the built `BlackHoleXch.driver` bundle to `/Library/Audio/Plug-Ins/HAL`
2. Restart CoreAudio using `sudo launchctl kickstart -kp system/com.apple.audio.coreaudiod`

### Customizing BlackHole

The following pre-compiler constants may be used to easily customize a build of BlackHole.

```
kDriver_Name
kPlugIn_BundleID
kPlugIn_Icon

kDevice_Name
kDevice_IsHidden
kDevice_HasInput
kDevice_HasOutput

kDevice2_Name
kDevice2_IsHidden
kDevice2_HasInput
kDevice2_HasOutput

kLatency_Frame_Size
kNumber_Of_Channels
kSampleRates
```

They can be specified at build time with `xcodebuild` using `GCC_PREPROCESSOR_DEFINITIONS`. 

Example:

```bash
xcodebuild \
  -project BlackHole.xcodeproj \
  GCC_PREPROCESSOR_DEFINITIONS='$GCC_PREPROCESSOR_DEFINITIONS kSomeConstant=value'
```

Be sure to escape any quotation marks when using strings. 

### Renaming BlackHole

To customize BlackHole it is required to change the following constants. 
- `kDriver_Name`
- `kPlugIn_BundleID` (note that this must match the target bundleID)
- `kPlugIn_Icon`

These can specified as pre-compiler constants using ```xcodebuild```.

```bash
driverName="BlackHole"
bundleID="audio.existential.BlackHole"
icon="BlackHole.icns"

xcodebuild \
  -project BlackHole.xcodeproj \
  -configuration Release \
  PRODUCT_BUNDLE_IDENTIFIER=$bundleID \
  GCC_PREPROCESSOR_DEFINITIONS='$GCC_PREPROCESSOR_DEFINITIONS
  kDriver_Name=\"'$driverName'\"
  kPlugIn_BundleID=\"'$bundleID'\"
  kPlugIn_Icon=\"'$icon'\"'
```

### Customizing Channels, Latency, and Sample Rates

`kNumber_Of_Channels` is used to set the number of channels. Be careful when specifying high channel counts. Although BlackHole is designed to be extremely efficient at higher channel counts it's possible that your computer might not be able to keep up. Sample rates play a roll as well. Don't use high sample rates with a high number of channels. Some applications don't know how to handle high channel counts. Proceed with caution.

`kLatency_Frame_Size` is how much time in frames that the driver has to process incoming and outgoing audio. It can be used to delay the audio inside of BlackHole up to a maximum of 65536 frames. This may be helpful if using BlackHole with a high channel count. 

`kSampleRates` set the sample rate or sample rates of the audio device. If using multiple sample rates separate each with a comma (`,`). For example: `kSampleRates='44100,48000'`.

### Mirror Device

By default BlackHole has a hidden mirrored audio device. The devices may be customized using the following constants. 

```
// Original Device
kDevice_IsHidden
kDevice_HasInput
kDevice_HasOutput

// Mirrored Device
kDevice2_IsHidden
kDevice2_HasInput
kDevice2_HasOutput
```

When all are set to true a 2nd BlackHole will show up that works exactly the same. The inputs and outputs are mirrored so the outputs from both devices go to the inputs of both devices.

This is useful if you need a separate device for input and output.

Example

```
// Original Device
kDevice_IsHidden=false
kDevice_HasInput=true
kDevice_HasOutput=false

// Mirrored Device
kDevice2_IsHidden=false
kDevice2_HasInput=false
kDevice2_HasOutput=true
```

In this situation we have two BlackHole devices. One will have inputs only and the other will have outputs only.

One way to use this in projects is to hide the mirrored device and use it behind the scenes. That way the user will see an input only device while routing audio through to the output behind them scenes. 

Hidden audio devices can be accessed using `kAudioHardwarePropertyTranslateUIDToDevice`.
