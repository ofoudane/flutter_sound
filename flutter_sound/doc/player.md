[Back to the README](../README.md#flutter-sound-api)

-----------------------------------------------------------------------------------------------------------------------

# Flutter Sound Player API

The verbs offered by the Flutter Sound Player module are :

- [Default constructor](#creating-the-player-instance)
- [openAudioSession](#openaudiosession-and-closeaudiosession) and [closeAudioSession()](#openAudioSession-and-closeAudioSession) to open or close an audio session
- [setAudioFocus()](#setaudiofocus) to manage the session Audio Focus
- [startPlayer()](#startplayer) to play an audio file or  a buffer.
- [startPlayerFromTrack](#startplayerfromtrack) to play data from a track specification and display controls on the lock screen or an Apple Watch
- [startPlayerFromStream](#startplayerfromstream) to play live data. Please look to the [following notice](codec.md#playing-pcm-16-from-a-dart-stream).
- [feedFromStream](#feedfromstream) to play live PCM data synchronously.  Please look to the [following notice](codec.md#playing-pcm-16-from-a-dart-stream).
- [foodSink](#foodsink) is the output stream when you want to play asynchronously live data
- [FoodData and FoodEvent](#food) are the two kinds of food that you can provide to the ```foodSink``` Stream.
- [stopPlayer()](#stopplayer) to stop a current playback
- [stopPlayer()](#stopplayer) to stop a current playback
- [pausePlayer()](#pauseplayer) to pause the current playback
- [resumePlayer()](#resumeplayer) to resume a paused playback
- [seekPlayer()](#seekplayer) to position directely inside the current playback
- [setVolume()](#setvolume) to adjust the ouput volume
- [playerState, isPlaying, isPaused, isStopped, getPlayerState()](#playerstate-isplaying-ispaused-isstopped-getplayerstate) to know the current player status
- [isDecoderSupported()](#isdecodersupported) to know if a specific codec is supported on the current platform.
- [onProgress()](#onprogress) to subscribe to a Stream of the Progress events
- [getProgress()](#getprogress) to query the current progress of a playback.
- [setUIProgressBar](#setuiprogressbar) to set the position of the progress bar on the Lock Screen
- [nowPlaying()](#nowplaying) to specify the containt of the lock screen beetween two playbacks
- [setSubscriptionDuration()](#setsubscriptionduration) to specify the frequence of your subscription

-------------------------------------------------------------------------------------------------------------------

## Creating the `Player` instance.

*Dart definition (prototype) :*
```
/* ctor */ FlutterSoundPlayer()
```

This is the first thing to do, if you want to deal with playbacks. The instanciation of a new player does not do many thing. You are safe if you put this instanciation inside a global or instance variable initialization.

*Example:*
```dart
FlutterSoundPlayer myPlayer = FlutterSoundPlayer();
```

--------------------------------------------------------------------------------------------------------------------

## `openAudioSession()` and `closeAudioSession()`

*Dart definition (prototype) :*
```
Future<FlutterSoundPlayer> openAudioSession
({
        AudioFocus focus = AudioFocus.requestFocusTransient,
        SessionCategory category = SessionCategory.playAndRecord,
        SessionMode mode = SessionMode.modeDefault,
        int audioFlags = outputToSpeaker,
        AudioDevice device = AudioDevice.speaker,
        bool withUI = false,
})

Future<void> closeAudioSession()
```

A player must be opened before used. A player correspond to an Audio Session. With other words, you must *open* the Audio Session before using it.
When you have finished with a Player, you must close it. With other words, you must close your Audio Session.
Opening a player takes resources inside the OS. Those resources are freed with the verb `closeAudioSession()`.

### `focus:` parameter

`focus` is an optional parameter can be specified during the opening : the Audio Focus.
This parameter can have the following values :
- AudioFocus.requestFocusAndStopOthers (your app will have **exclusive use** of the output audio)
- AudioFocus.requestFocusAndDuckOthers (if another App like Spotify use the output audio, its volume will be **lowered**)
- AudioFocus.requestFocusAndKeepOthers (your App will play sound **above** others App)
- AudioFocus.requestFocusAndInterruptSpokenAudioAndMixWithOthers (for Android)
- AudioFocus.requestFocusTransient (for Android)
- AudioFocus.requestFocusTransientExclusive (for Android)
- AudioFocus.doNotRequestFocus (useful if you want to mangage yourself the Audio Focus with the verb ```setAudioFocus()```)

The Audio Focus is abandoned when you close your player. If your App must play several sounds, you will probably open  your player just once, and close it when you have finished with the last sound. If you close and reopen an Audio Session for each sound, you will probably get unpleasant things for the ears with the Audio Focus.

### `category`

`category` is an optional parameter used only on iOS.
This parameter can have the following values :
- ambient
- multiRoute
- playAndRecord
- playback
- record
- soloAmbient
- audioProcessing

See [iOS documentation](https://developer.apple.com/documentation/avfoundation/avaudiosessioncategory?language=objc) to understand the meaning of this parameter.

### `mode`

`mode` is an optional parameter used only on iOS.
This parameter can have the following values :
- modeDefault
- modeGameChat
- modeMeasurement
- modeMoviePlayback
- modeSpokenAudio
- modeVideoChat
- modeVideoRecording
- modeVoiceChat
- modeVoicePrompt

See [iOS documentation](https://developer.apple.com/documentation/avfoundation/avaudiosessionmode?language=objc) to understand the meaning of this parameter.

### `audioFlags` are a set of optional flags (used on iOS):

- outputToSpeaker
- allowHeadset
- allowEarPiece
- allowBlueTooth
- allowAirPlay
- allowBlueToothA2DP

### `device` is the output device (used on Android)

- speaker
- headset,
- earPiece,
- blueTooth,
- blueToothA2DP,
- airPlay

## `withUI` is a boolean that you set to `true` if you want to control your App from the lock-screen (using [startPlayerFromTrack()](player.md#startplayerfromtrack) during your Audio Session).

You MUST ensure that the player has been closed when your widget is detached from the UI.
Overload your widget's `dispose()` method to closeAudioSession the player when your widget is disposed.
In this way you will reset the player and clean up the device resources, but the player will be no longer usable.

```dart
@override
void dispose()
{
        if (myPlayer != null)
        {
            myPlayer.closeAudioSession();
            myPlayer = null;
        }
        super.dispose();
}
```

You may not open many Audio Sessions without closing them.
You will be very bad if you try something like :
```dart
    while (aCondition)  // *DO'NT DO THAT*
    {
            flutterSound = FlutterSoundPlayer().openAudioSession(); // A **new** Flutter Sound instance is created and opened
            flutterSound.startPlayer(bipSound);
    }
```

`openAudioSession()` and `closeAudioSession()` return Futures. You may not use your Player before the end of the initialization. So probably you will `await` the result of `openAudioSession()`. This result is the Player itself, so that you can collapse instanciation and initialization together with `myPlayer = await FlutterSoundPlayer().openAudioSession();`

*Example:*
```dart
    myPlayer = await FlutterSoundPlayer().openAudioSession(focus: Focus.requestFocusAndDuckOthers, outputToSpeaker | allowBlueTooth);

    ...
    (do something with myPlayer)
    ...

    await myPlayer.closeAudioSession();
    myPlayer = null;
```

------------------------------------------------------------------------------------------------------------------

## `setAudioFocus()`

*Dart definition (prototype) :*
```
Future<void> setAudioFocus
({
        AudioFocus focus = AudioFocus.requestFocusTransient,
        SessionCategory category = SessionCategory.playAndRecord,
        SessionMode mode = SessionMode.modeDefault,
        int audioFlags = outputToSpeaker,
        AudioDevice device = AudioDevice.speaker,
})
```

### `focus:` parameter possible values are
- AudioFocus.requestFocus (request focus, but do not do anything special with others App)
- AudioFocus.requestFocusAndStopOthers (your app will have **exclusive use** of the output audio)
- AudioFocus.requestFocusAndDuckOthers (if another App like Spotify use the output audio, its volume will be **lowered**)
- AudioFocus.requestFocusAndKeepOthers (your App will play sound **above** others App)
- AudioFocus.requestFocusAndInterruptSpokenAudioAndMixWithOthers
- AudioFocus.requestFocusTransient (for Android)
- AudioFocus.requestFocusTransientExclusive (for Android)
- AudioFocus.abandonFocus (Your App will not have anymore the audio focus)

### Other parameters :

Please look to [openAudioSession()](player.md#openaudiosession-and-closeaudiosession) to understand the meaning of the other parameters


*Example:*
```dart
        myPlayer.setAudioFocus(focus: AudioFocus.requestFocusAndDuckOthers);
```

-----------------------------------------------------------------------------------------------------------------

## `startPlayer()`

*Dart definition (prototype) :*
```
Future<Duration> startPlayer
({
        String fromUri,
        Uint8List fromDataBuffer,
        StreamSink fromStream,
        Codec codec,
        int sampleRate, // Used only when Codec == Codec.pcm16
        TWhenFinished whenFinished
})
```

You can use `startPlayer` to play a sound.

- `startPlayer()` has three optional parameters, depending on your sound source :
   - `fromUri:`  (if you want to play a file or a remote URI)
   - `fromDataBuffer:` (if you want to play from a data buffer)
   - `sampleRate` is mandatory if `codec` == `Codec.pcm16`. Not used for other codecs.

You must specify one or the three parameters : `fromUri`, `fromDataBuffer`, `fromStream`.

- You use the optional parameter`codec:` for specifying the audio and file format of the file. Please refer to the [Codec compatibility Table](codec.md#actually-the-following-codecs-are-supported-by-flutter_sound) to know which codecs are currently supported.

- `whenFinished:()` : A lambda function for specifying what to do when the playback will be finished.

Very often, the `codec:` parameter is not useful. Flutter Sound will adapt itself depending on the real format of the file provided.
But this parameter is necessary when Flutter Sound must do format conversion (for example to play opusOGG on iOS).

`startPlayer()` returns a Duration Future, which is the record duration.

Hint: [path_provider](https://pub.dev/packages/path_provider) can be useful if you want to get access to some directories on your device.


*Example:*
```dart
        Directory tempDir = await getTemporaryDirectory();
        File fin = await File ('${tempDir.path}/flutter_sound-tmp.aac');
        Duration d = await myPlayer.startPlayer(fin.path, codec: Codec.aacADTS);

        _playerSubscription = myPlayer.onProgress.listen((e)
        {
                // ...
        });
}
```

*Example:*
```dart
    final fileUri = "https://file-examples.com/wp-content/uploads/2017/11/file_example_MP3_700KB.mp3";

    Duration d = await myPlayer.startPlayer
    (
                fileUri,
                codec: Codec.mp3,
                whenFinished: ()
                {
                         print( 'I hope you enjoyed listening to this song' );
                },
    );
```

--------------------------------------------------------------------------------------------------------------------------------

## `startPlayerFromTrack()`

*Dart definition (prototype) :*
```
Future<Duration> startPlayerFromTrack(
    Track track,
    {
    TWhenFinished whenFinished = null,
    TonPaused onPaused = null,
    TonSkip onSkipForward = null,
    TonSkip onSkipBackward = null,
    bool removeUIWhenStopped = true,
    bool defaultPauseResume = null,
    })
```

Use this verb to play data from a track specification and display controls on the lock screen or an Apple Watch. The Audio Session must have been open with the parameter `withUI`.

- `track` parameter is a simple structure which describe the sound to play. Please see [here the Track structure specification](track.md)

- `whenFinished:()` : A function for specifying what to do when the playback will be finished.

- `onPaused:()` : this parameter can be :
   - a call back function to call when the user hit the Skip Pause button on the lock screen
   - <null> : The pause button will be handled by Flutter Sound internal

- `onSkipForward:()` : this parameter can be :
   - a call back function to call when the user hit the Skip Forward button on the lock screen
   - <null> : The Skip Forward button will be disabled

- `onSkipBackward:()` : this parameter can be :
   - a call back function to call when the user hit the Skip Backward button on the lock screen
   - <null> : The Skip Backwqrd button will be disabled

- `removeUIWhenStopped` : is a boolean to specify if the UI on the lock screen must be removed when the sound is finished or when the App does a `stopPlayer()`. Most of the time this parameter must be true. It is used only for the rare cases where the App wants to control the lock screen between two playbacks. Be aware that if the UI is not removed, the button Pause/Resume, Skip Backward and Skip Forward remain active between two playbacks. If you want to disable those button, use the API verb ```nowPlaying()```.
Remark: actually this parameter is implemented only on iOS.

- `defaultPauseResume` : is a boolean value to specify if Flutter Sound must pause/resume the playback by itself when the user hit the pause/resume button. Set this parameter to *FALSE* if the App wants to manage itself the pause/resume button. If you do not specify this parameter and the `onPaused` parameter is specified then Flutter Sound will assume `FALSE`. If you do not specify this parameter and the `onPaused` parameter is not specified then Flutter Sound will assume `TRUE`.
Remark: actually this parameter is implemented only on iOS.


`startPlayerFromTrack()` returns a Duration Future, which is the record duration.


*Example:*
```dart
    final fileUri = "https://file-examples.com/wp-content/uploads/2017/11/file_example_MP3_700KB.mp3";
    Track track = Track( codec: Codec.opusOGG, trackPath: fileUri, trackAuthor: '3 Inches of Blood', trackTitle: 'Axes of Evil', albumArtAsset: albumArt )
    Duration d = await myPlayer.startPlayerFromTrack
    (
                track,
                whenFinished: ()
                {
                         print( 'I hope you enjoyed listening to this song' );
                },
    );
```

--------------------------------------------------------------------------------------------------------------------------------

## `startPlayerFromStream()`

*Dart definition (prototype) :*
```
Future<void> startPlayerFromStream
(
    Codec codec = Codec.pcm16,
    int numChannels = 1,
    int sampleRate = 16000,
)
```

**This new functionnality needs, at least, and Android SDK >= 21**

- The only codec supported is actually `Codec.pcm16`.
- The only value possible for `numChannels` is actually 1.
- SampleRate is the sample rate of the data you want to play.

Please look to [the following notice](codec.md#playing-pcm-16-from-a-dart-stream)

*Example*
You can look to the three provided examples :

- [This example](../example/README.md#liveplaybackwithbackpressure) shows how to play Live data, with Back Pressure from Flutter Sound
- [This example](../example/README.md#liveplaybackwithoutbackpressure) shows how to play Live data, without Back Pressure from Flutter Sound
- [This example](../example/README.md#soundeffect) shows how to play some real time sound effects.

*Example 1:*
```dart
await myPlayer.startPlayerFromStream(codec: Codec.pcm16, numChannels: 1, sampleRate: 48000);

await myPlayer.feedFromStream(aBuffer);
await myPlayer.feedFromStream(anotherBuffer);
await myPlayer.feedFromStream(myOtherBuffer);

await myPlayer.stopPlayer();
```
*Example 2:*
```dart
await myPlayer.startPlayerFromStream(codec: Codec.pcm16, numChannels: 1, sampleRate: 48000);

myPlayer.foodSink.add(FoodData(aBuffer));
myPlayer.foodSink.add(FoodData(anotherBuffer));
myPlayer.foodSink.add(FoodData(myOtherBuffer));

myPlayer.foodSink.add(FoodEvent((){_mPlayer.stopPlayer();}));
```

---------------------------------------------------------------------------------------------------------------------------------------------

## `feedFromStream`

*Dart definition (prototype) :*
```
Future<void> feedFromStream(Uint8List buffer) async
```

This is the verb that you use when you want to play live PCM data synchronously.
This procedure returns a Future. It is very important that you wait that this Future is completed before trying to play another buffer.

*Example:*

- [This example](../example/README.md#liveplaybackwithbackpressure) shows how to play Live data, with Back Pressure from Flutter Sound
- [This example](../example/README.md#soundeffect) shows how to play some real time sound effects synchronously.

```dart
await myPlayer.startPlayerFromStream(codec: Codec.pcm16, numChannels: 1, sampleRate: 48000);

await myPlayer.feedFromStream(aBuffer);
await myPlayer.feedFromStream(anotherBuffer);
await myPlayer.feedFromStream(myOtherBuffer);

await myPlayer.stopPlayer();
```

---------------------------------------------------------------------------------------------------------------------------------

## `foodSink`

*Dart definition (prototype) :*
```
StreamSink<Food> get foodSink
```

This the output stream that you use when you want to play asynchronously live data.
This StreamSink accept two kinds of objects :
- FoodData (the buffers that you want to play)
- FoodEvent (a call back to be called after a resynchronisation)

*Example:*

[This example](../example/README.md#liveplaybackwithoutbackpressure) shows how to play Live data, without Back Pressure from Flutter Sound
```dart
await myPlayer.startPlayerFromStream(codec: Codec.pcm16, numChannels: 1, sampleRate: 48000);

myPlayer.foodSink.add(FoodData(aBuffer));
myPlayer.foodSink.add(FoodData(anotherBuffer));
myPlayer.foodSink.add(FoodData(myOtherBuffer));
myPlayer.foodSink.add(FoodEvent((){_mPlayer.stopPlayer();}));
```

----------------------------------------------------------------------------------------------------------------------------------

##`Food`

*Dart definition (prototype) :*
```
/* ctor */ FoodData(Uint8List buffer)
/* ctor */ FoodEvent(Function callback)
```

This are the objects that you can `add` to `foodSink`
The Food class has two others inherited classes :

- FoodData (the buffers that you want to play)
- FoodEvent (a call back to be called after a resynchronisation)

*Example:*

[This example](../example/README.md#liveplaybackwithoutbackpressure) shows how to play Live data, without Back Pressure from Flutter Sound
```dart
await myPlayer.startPlayerFromStream(codec: Codec.pcm16, numChannels: 1, sampleRate: 48000);

myPlayer.foodSink.add(FoodData(aBuffer));
myPlayer.foodSink.add(FoodData(anotherBuffer));
myPlayer.foodSink.add(FoodData(myOtherBuffer));
myPlayer.foodSink.add(FoodEvent(()async {await _mPlayer.stopPlayer(); setState((){});}));
```

---------------------------------------------------------------------------------------------------------------------------------

## `stopPlayer()`

*Dart definition (prototype) :*
```
Future<void> stopPlayer( )
```

Use this verb to stop a playback. This verb never throw any exception. It is safe to call it everywhere,
for example when the App is not sure of the current Audio State and want to recover a clean reset state.

*Example:*
```dart
        await myPlayer.stopPlayer();
        if (_playerSubscription != null)
        {
                _playerSubscription.cancel();
                _playerSubscription = null;
        }
```



---------------------------------------------------------------------------------------------------------------------------------

## `pausePlayer()`

*Dart definition (prototype) :*
```
Future<void> pausePlayer( )
```

Use this verbe to pause the current playback. An exception is thrown if the player is not in the "playing" state.

*Example:*
```dart
await myPlayer.pausePlayer();
```

--------------------------------------------------------------------------------------------------------------------------------

## `resumePlayer()`

*Dart definition (prototype) :*
```
Future<void> resumePlayer( )
```

Use this verbe to resume the current playback. An exception is thrown if the player is not in the "paused" state.

*Example:*
```dart
await myPlayer.resumePlayer();
```

-------------------------------------------------------------------------------------------------------------------------------
## `seekPlayer()`

*Dart definition (prototype) :*
```
Future<void> seekPlayer( Duration duration )
```

To seek to a new location. The player must already be playing or paused. If not, an exception is thrown.

*Example:*
```dart
await myPlayer.seekToPlayer(Duration(milliseconds: milliSecs));
```

----------------------------------------------------------------------------------------------------------------------------------

## `setVolume()`

*Dart definition (prototype) :*
```
Future<void> setVolume( double volume )
```

The parameter is a floating point number between 0 and 1.
Volume can be changed when player is running. Manage this after player starts.

*Example:*
```dart
await myPlayer.setVolume(0.1);
```

---------------------------------------------------------------------------------------------------------------------------------

## `playerState`, `isPlaying`, `isPaused`, `isStopped`. `getPlayerState()`

*Dart definition (prototype) :*
```
    PlayerState playerState;
    bool get isPlaying => playerState == PlayerState.isPlaying;
    bool get isPaused => playerState == PlayerState.isPaused;
    bool get isStopped => playerState == PlayerState.isStopped;
    Future<PlayerState> getPlayerState() async
```

This four verbs is used when the app wants to get the current Audio State of the player.

`playerState` is an attribut which can have the following values :

  - isStopped   /// Player is stopped
  - isPlaying   /// Player is playing
  - isPaused    /// Player is paused

- isPlaying is a boolean attribut which is `true` when the player is in the "Playing" mode.
- isPaused is a boolean atrribut which  is `true` when the player is in the "Paused" mode.
- isStopped is a boolean atrribut which  is `true` when the player is in the "Stopped" mode.

Flutter Sound shows in the `playerState` attribut the last known state. When the Audio State of the background OS engine changes, the `playerState` parameter is not updated exactly at the same time.
If you want the exact background OS engine state you must use ```PlayerState theState = await myPlayer.getPlayerState()```.
Acutually `getPlayerState()` is only implemented on iOS.

*Example:*
```dart
        swtich(myPlayer.playerState)
        {
                case PlayerState.isPlaying: doSomething; break;
                case PlayerState.isStopped: doSomething; break;
                case PlayerState.isPaused: doSomething; break;
        }
        ...
        if (myPlayer.isStopped) doSomething;
        if (myPlayer.isPlaying) doSomething;
        if (myPlayer.isPaused) doSomething;
        ...
        PlayerState theState = await myPlayer.getPlayerState();
        ...

```

---------------------------------------------------------------------------------------------------------------------------------

## `isDecoderSupported()`

*Dart definition (prototype) :*
```
 Future<bool> isDecoderSupported(Codec codec)
```

This verb is useful to know if a particular codec is supported on the current platform.
Returns a Future<bool>.

*Example:*
```dart
        if ( await myPlayer.isDecoderSupported(Codec.opusOGG) ) doSomething;
```

---------------------------------------------------------------------------------------------------------------------------------

## `onProgress`

*Dart definition (prototype) :*
```
Stream<PlaybackDisposition> get onProgress => playerController != null ? playerController.stream : null;
```

The attribut `onProgress` is a stream on which FlutterSound will post the player progression.
You may listen to this Stream to have feedback on the current playback.

PlaybackDisposition has two fields :
- Duration duration  (the total playback duration)
- Duration position  (the current playback position)

*Example:*
```dart
        _playerSubscription = myPlayer.onProgress.listen((e)
        {
                Duration maxDuration = e.duration;
                Duration position = e.position;
                ...
        }
```

---------------------------------------------------------------------------------------------------------------------------------

## `getProgress()`

*Dart definition (prototype) :*
```
Future<Map> getProgress() async
```

This verb is used to get the current progress of a playback.
It returns a `Map` with two Duration entries : `'progress'` and `'duration'`.
Remark : actually only implemented on iOS.

*Example:*
```dart
        Duration progress = (await getProgress())['progress'];
        Duration duration = (await getProgress())['duration'];
```

---------------------------------------------------------------------------------------------------------------------------------

## `setUIProgressBar()`

*Dart definition (prototype) :*
```
Future<void> setUIProgressBar ( {
                                    Duration duration,
                                    Duration progress,
                               }) async

```

This verb is used if the App wants to control itself the Progress Bar on the lock screen. By default, this progress bar is handled automaticaly by Flutter Sound.
Remark `setUIProgressBar()` is implemented only on iOS.

*Example:*
```dart

        Duration progress = (await getProgress())['progress'];
        Duration duration = (await getProgress())['duration'];
        setUIProgressBar(progress: Duration(milliseconds: progress.milliseconds - 500), duration: duration)
````

---------------------------------------------------------------------------------------------------------------------------------

## `nowPlaying()`

*Dart definition (prototype) :*
```
Future<void> nowPlaying( Track track,
              {
                Duration duration,
                Duration progress,
                TonSkip onSkipForward,
                TonSkip onSkipBackward,
                TonPaused onPaused,
              }) async
```

This verb is used to set the Lock screen fields without starting a new playback.
The fields 'dataBuffer' and 'trackPath' of the Track parameter are not used.
Please refer to 'startPlayerFromTrack' for the meaning of the others parameters.
Remark `setUIProgressBar()` is implemented only on iOS.

*Example:*
```dart
    Track track = Track( codec: Codec.opusOGG, trackPath: fileUri, trackAuthor: '3 Inches of Blood', trackTitle: 'Axes of Evil', albumArtAsset: albumArt );
    await nowPlaying(Track);
```

---------------------------------------------------------------------------------------------------------------------------------

## `setSubscriptionDuration()`

*Dart definition (prototype) :*
```
Future<void> setSubscriptionDuration(Duration duration)
```

This verb is used to change the default interval between two post on the "Update Progress" stream. (The default interval is 10ms)

*Example:*
```dart
// 0.010s. is default
myPlayer.setSubscriptionDuration(Duration(milliseconds: 20));
```

---------------------------------------------------------------------------------------------------------------------------------

[Back to the README](../README.md#flutter-sound-api)
