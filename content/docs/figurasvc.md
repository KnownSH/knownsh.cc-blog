+++
+++

<head>
    <meta content="Intro to FiguraSVC 1.1.0" property="og:description" />
</head>

# Intro to FiguraSVC 1.1.0
A simple unofficial addon for [Figura](https://modrinth.com/mod/figura) that allows Simple Voice Chat to be detected by Figura Lua scripts. \
**This works without any serverside code or mods.**

<br>

## Overview
This mod is built to be **as simple as possible**, the mod introduces only one new event to Figura, which you can declare in your Lua script like so:
```lua
events["svc.microphone"] = function(pcm)
  -- code (unsafe on servers) (client only)
end
```
This `svc.microphone` event is triggered each gametick that the client has their microphone active.

<br>

## Multiplayer Compatibility

To ensure compatibility and avoid potential issues when using FiguraSVC in a multiplayer setting, it is recommended to wrap the event in a mod check. This way, you can verify if the mod is loaded before executing the event:
```lua
if client:isModLoaded("figurasvc") then
    events["svc.microphone"] = function (pcm)
        -- code (safe on servers) (client only)
    end
end
```
<br>

While the above works on the clientside, other players on a server **won't see anything when the event is called**. To make anything *(i.e. talking animations)* visible to all other players, you can utilize the following code snippet:
```lua
-- Example talking animation
local microphoneOffTime = 0
local isMicrophoneOn = false

function pings.talking(state)
    -- (safe on servers) (visible to everyone)
    animations.model.talk:setPlaying(state)
end

function events.tick()
    local previousMicState = isMicrophoneOn
    
    microphoneOffTime = microphoneOffTime + 1
    isMicrophoneOn = microphoneOffTime <= 2 -- has svc.microphone been called in the past 2 ticks?
  
    if previousMicState ~= isMicrophoneOn then
        pings.talking(isMicrophoneOn)
    end
end

if client:isModLoaded("figurasvc") and host:isHost() then
    events["svc.microphone"] = function (pcm)
        microphoneOffTime = 0
    end
end
```
In this example, the pings.talking function is used to trigger an animation when the microphone state changes, ensuring that it is visible to all players.

<br>

---

## "What does the `pcm` parameter do?"

PCM, or Pulse-Code Modulation, is a process used to digitally represent analog audio signals. In FiguraSVC, `pcm` is a value that represents a 16-bit PCM audio sample that lasts **0.05 seconds**, aka one Minecraft game tick.

In Luau, the `pcm` parameter would look like this:
```lua
type PCMTable = {[number]: number}
```

Each entry in the table represents the amplitude of the audio signal at a specific moment in time. By accessing and manipulating this table, you can perform various operations on the audio data.

<br>

Heres a quick and dirty example script I made that demonstrates using the `pcm` value to get the average audio amplitude for that game tick.
```lua
local function processPCM(pcm)
    -- Here, you have access to the PCM audio data in the 'pcm' variable
    -- You can perform various operations on the PCM data, such as:
    local averageAmplitude = 0
    for i = 1, #pcm do
        averageAmplitude = averageAmplitude + math.abs(pcm[i])
    end
    averageAmplitude = averageAmplitude / #pcm

    print(averageAmplitude)
end

-- Check if FiguraSVC mod is loaded and set up the svc.microphone event
if client:isModLoaded("figurasvc") and host:isHost() then
    events["svc.microphone"] = function (pcm)
        -- Handle the microphone input and PCM data
        processPCM(pcm)
    end
end
```