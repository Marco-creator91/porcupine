# Porcupine Wake Word Engine

Made in Vancouver, Canada by [Picovoice](https://picovoice.ai)

Porcupine is a highly-accurate and lightweight wake word engine. It enables building always-listening voice-enabled
applications.

Porcupine is:

- private and offline
- [accurate](https://github.com/Picovoice/wake-word-benchmark)
- [resource efficient](https://www.youtube.com/watch?v=T0tAnh8tUQg) (runs even on microcontrollers)
- data efficient (wake words can be easily generated by simply typing them, without needing thousands of hours of bespoke audio training data and manual effort)
- scalable to many simultaneous wake-words / always-on voice commands
- cross-platform

To learn more about Porcupine, see the [product](https://picovoice.ai/products/porcupine/), [documentation](https://picovoice.ai/docs/), and [GitHub](https://github.com/Picovoice/porcupine/) pages.

## Requirements

- Java 11+

## Compatibility

- Linux (x86_64)
- macOS (x86_64, arm64)
- Windows (x86_64, arm64)
- Raspberry Pi 3 (32 and 64 bit), Raspberry Pi 4 (32 and 64 bit), Raspberry Pi 5 (32 and 64 bit)

## Installation

The latest Java bindings are available from the Maven Central Repository at:

```console
ai.picovoice:porcupine-java:${version}
```

If you're using Gradle for your Java project, include the following line in your `build.gradle` file to add Porcupine:
```console
implementation 'ai.picovoice:porcupine-java:${version}'
```

If you're using IntelliJ, open the Project Structure dialog (`File > Project Structure`) and go to the `Libraries` section.
Click the plus button at the top to add a new project library and select `From Maven...`. Search for `ai.picovoice:porcupine-java`
in the search box and add the latest version to your project.

## Build

To build from source, invoke the Gradle build task from the command-line:
```console
cd porcupine/binding/java
./gradlew build
```

Once the task is complete, the output JAR can be found in `porcupine/binding/java/build/libs`.

## AccessKey

Porcupine requires a valid Picovoice `AccessKey` at initialization. `AccessKey` acts as your credentials when using Porcupine SDKs.
You can get your `AccessKey` for free. Make sure to keep your `AccessKey` secret.
Signup or Login to [Picovoice Console](https://console.picovoice.ai/) to get your `AccessKey`.

## Usage

The easiest way to create an instance of the engine is with the Porcupine Builder:

```java
import ai.picovoice.porcupine.*;

final String accessKey = "${ACCESS_KEY}"; // AccessKey obtained from Picovoice Console (https://console.picovoice.ai/)
try{
    Porcupine handle = new Porcupine.Builder()
                        .setAccessKey(accessKey)
                        .setBuiltInKeyword(BuiltInKeyword.PORCUPINE)
                        .build();
} catch (PorcupineException e) { }
```

`handle` is an instance of Porcupine that detects utterances of "Picovoice". The `setBuiltInKeyword()` builder argument is a shorthand
for accessing built-in keyword model files shipped with the package.

The list of built-in keywords can be found in the `BuiltInKeyword` enum, and can be retrieved by:

```java
import ai.picovoice.porcupine.*;

for(BuiltInKeyword keyword : BuiltInKeyword.values()) {
    System.out.println(keyword.name());
}
```

Porcupine can detect multiple keywords concurrently:

```java
import ai.picovoice.porcupine.*;

final String accessKey = "${ACCESS_KEY}"; // AccessKey obtained from Picovoice Console (https://console.picovoice.ai/)
try{
    Porcupine handle = new Porcupine.Builder()
                        .setAccessKey(accessKey)
                        .setBuiltInKeywords(new BuiltInKeyword[]{BuiltInKeyword.BUMBLEBEE, BuiltInKeyword.PICOVOICE})
                        .build();
} catch (PorcupineException e) { }
```

To detect non-default keywords use the `setKeywordPaths()` builder argument instead:

```java
import ai.picovoice.porcupine.*;

final String accessKey = "${ACCESS_KEY}"; // AccessKey obtained from Picovoice Console (https://console.picovoice.ai/)
String[] keywordPaths = new String[]{ "/absolute/path/to/keyword/one", "/absolute/path/to/keyword/two", ...}
try{
    Porcupine handle = new Porcupine.Builder()
                        .setAccessKey(accessKey)
                        .setKeywordPaths(keywordPaths)
                        .build();
} catch (PorcupineException e) { }
```

The sensitivity of the engine can be tuned per-keyword using the `setSensitivities` builder argument

```java
import ai.picovoice.porcupine.*;

final String accessKey = "${ACCESS_KEY}"; // AccessKey obtained from Picovoice Console (https://console.picovoice.ai/)
try{
    Porcupine handle = new Porcupine.Builder()
                        .setAccessKey(accessKey)
                        .setBuiltInKeywords(new BuiltInKeyword[]{BuiltInKeyword.GRAPEFRUIT, BuiltInKeyword.PORCUPINE})
                        .setSensitivities(new float[]{ 0.6f, 0.35f })
                        .build();
} catch (PorcupineException e) { }
```

Sensitivity is the parameter that enables trading miss rate for the false alarm rate. It is a floating-point number within
`[0, 1]`. A higher sensitivity reduces the miss rate at the cost of increased false alarm rate.

When initialized, the valid sample rate is given by `handle.getSampleRate()`. Expected frame length (number of audio samples
in an input array) is `handle.getFrameLength()`. The engine accepts 16-bit linearly-encoded PCM and operates on
single-channel audio.

```java
short[] getNextAudioFrame(){
    // .. get audioFrame
    return audioFrame;
}

while(true){
    int keywordIndex = handle.process(getNextAudioFrame());
    if(keywordIndex >= 0){
	    // .. detection event logic/callback
    }
}
```

Once you're done with Porcupine, ensure you release its resources explicitly:

```java
handle.delete();
```

## Non-English Wake Words

In order to detect non-English wake words you need to use the corresponding model file. The model files for all supported languages are available [here](../../lib/common).

## Demos

The [Porcupine Java demo](../../demo/java) is a Java command-line application that allows for  processing real-time audio
(i.e. microphone) and files using Porcupine.
