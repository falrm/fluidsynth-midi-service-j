[![CircleCI](https://circleci.com/gh/falrm/fluidsynth-midi-service-j/tree/master.svg?style=svg)](https://circleci.com/gh/falrm/fluidsynth-midi-service-j/tree/master)

# What is this?

It is an Android MIDI device service implementation based on [Fluidsynth software synthesizer](https://github.com/Fluidsynth/fluidsynth/). It comes with some sample dogfooding app UI.

Since Android 6.0 (Marshmallow), it started to provide the standard way to
receive and send MIDI messages via USB, BLE or any other means through
[`android.media.midi` API](https://developer.android.com/reference/android/media/midi/package-summary). (Details are described at [Android Open Source Project](https://source.android.com/devices/audio/midi).)

By implementing [`android.media.midi.MidiDeviceService` class](https://developer.android.com/reference/android/media/midi/MidiDeviceService), it is possible
to create a new virtual MIDI device, completely based on software.
With Android audio API, it is also possible to reuse existing software
MIDI synthesizers that runs on Linux (if its audio dependency is isolated).

Fluidsynth is one of such a software synthesizer and is (considerably) easy
to port to Android. It is written in C with a handful of C library dependencies.

Apart from Android MIDI API, it is also possible to use libfluidsynth as a mere library that is used only for an app, without offering MIDI ports as a service.


## Compared to other Android ports

There are some people who build Fluidsynth for Android, but what makes
this port special is that it also provides audible driver sources, not just
dummy output. It makes use of [OpenSL ES API](https://developer.android.com/ndk/guides/audio/opensl/) as well as [Oboe Audio API](https://github.com/google/Oboe). Both are now based on Fluidsynth master, as my patches are now merged there.

More background can be found at https://dev.to/atsushieno/fluidsynth-20x-for-android-4j6b

(I have no idea whether other Fluidsynth Android ports not supporting has some special contract with Fluidsynth developers and they don't have to provide sources or not.)


## Demo movie

[demo movie](docs/demo.mp4) (How can I embed a video that can be played on github?)


## Prebuilt binaries

Beta packages are available at [DeployGate](https://dply.me/l0etkk).

## CI builds

Pretty complete build instructions, as well as where to get those binary artifacts
to just throw into your `fluidsynthjna/src/main/jniLibs` are on CircleCI's latestest build 
[![CircleCI](https://circleci.com/gh/falrm/fluidsynth-midi-service-j/tree/master.svg?style=svg)](https://circleci.com/gh/falrm/fluidsynth-midi-service-j/tree/master)


[demo movie](docs/demo.mp4)



## Building

Building the entire app is complicated because it needs to build fluidsynth for Android as well as generating automated Java API binding from libfluidsynth C headers.

This is the basic build steps:

- go to `fluidsynthjna` directory and run `make prepare-fluidsynth` and `make`.
  - At `make prepare-fluidsynth` step, it may ask you to enter your admin password. It is what `cerbero` build system (explained later) does.
- then run `./gradlew assembleRelease` (etc.) to build Java (Kotlin) app.

You will need make, wget, and Maven (mvn) installed too.


### Dependencies

It depends on official [fluidsynth github repo](https://github.com/Fluidsynth/fluidsynth) which now contains some
Android build scripts, which in turn depends on [Cerbero build system](https://cgit.freedesktop.org/gstreamer/cerbero/) to build glib-2.0 and co, one of fluidsynth's dependencies. (See [docs/design/BUILDING.md](docs/design/BUILDING.md) on how so.)

To avoid this most difficult part, building libfluidsynth.so with OpenSLES support, we have a set of prebuilt shared libraries so that anyone who just wants to build own synthesizer apps can reuse it: see
https://github.com/atsushieno/fluidsynth-midi-service-j/issues/12

It bundles `FluidR3Mono_GM.sf3` which is bundled into apk as Android assets. It is released under MIT license (see the source directory).

For comsumption in Java-based Android application, we use [JNA](https://github.com/java-native-access/jna) and [JNAerator](https://github.com/nativelibs4java/JNAerator) to provide Java binding for libfluidsynth API. There is a known issue on JNAerator build; it requries Java8 javac (openjdk works). JNAerator has not been maintained upstream to accept newer versions. We'd need Java8 for Android app builds anyways.

The rest of the Java application is written in Kotlin.


## Scope of the project

This application is built as a proof-of-concept and dogfooding for fluidsynth Android audio drivers. It wouldn't bring best user experiences and features. For example, the MIDI player is based on Java (Kotlin) which is not optimal.
