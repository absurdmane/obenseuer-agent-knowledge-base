# Audio, radio, camera, graphics settings, post-processing
Scope: soundscape and mixer, volume settings, radio, player camera, graphics quality and post-processing, light dimming and flicker. Not covered: camera shake and event cameras, Azure sky lighting (periodic-rendering.md, time-and-weather.md), per-room sound assets, TV channel content, the music source.
Status: code-only
Related: [power.md](power.md), [ui-options-input.md](ui-options-input.md), [periodic-rendering.md](periodic-rendering.md), [time-and-weather.md](time-and-weather.md)

## Where it lives [code]

- Soundscape: `Soundscape` (ScriptableObject: `loopSounds`, `randomSounds`), `SoundscapeController` (`instance`, owns the mixer groups), `SoundscapeArea`, `SoundscapeTrigger` (room or door trigger), `SoundscapeGlobal` (day/night), `SoundscapeSound*` (per-sound settings), `SoundscapeSoundTypes`. Generic sound: `SoundInfo` (clip, settings, owns an `AudioSource`), `AudioSourceMaker`, `MiscAudioPlayer` (static helpers, loads `Resources/Sounds/MainMixer`), `RandomAudio`, `PlayAudio`.
- Volume: `VolumeSave` (sliders to `AudioMixer` parameters, saved as `Volume.json`), `OptionsMenu`. Radio: `Radio : Interactable`. TV: `TelevisionNoise` (power in power.md).
- Camera: `PlayerCamera`, `ThirdPersonCameraController` (orbit), `ThirdPersonCameraCollision` (first/third person, zoom), `SetCameraFOV`, `DisplaySave` (FOV). Post-processing: `EffectsController` (singleton), `EffectPostProcessVolume`, `PostProcessingEffects`. Quality: `QualitySave`, `OptionsMenu`. Lights: `LightCullZone`, `LightFlicker`.

## How it works [code]

- Soundscape: the controller keeps a foreground `currentSoundscape` and a background one set by a trigger. `PlaySoundscape` stops the previous foreground area and `UpdateSoundscape` starts its loop and random sounds. On each hour change the foreground area is updated again, so `minPlayTimeHour`/`maxPlayTimeHour`/`probabilityToPlay` apply. A sound attached to an area that is not playing starts at volume 0. `SoundscapeTrigger` fires on the player's head trigger; a door on it changes the background volume (`maxBackgroundVolume` vs `maxSoundBehindDoor`). `SoundscapeGlobal` listens to `MinutePassed` and uses the day area from 08:00 to 21:00, else the night area (up to a minute late). Sounds flagged `DontPlayWhenWastelandIsEnabled` go to a separate mixer group.
- Mixer parameters (exposed names): `Master`, `Sound`, `OtherSounds`, `Music`, `Nasty` (menu sliders), `Soundscape`, `BackgroundSoundscape` (area volumes). The `Sound` slider also sets `OtherSounds`. `SoundscapeController.UpdateMainMixer` moves `Soundscape` toward 0 by 0.2 per frame while a trigger is set, so a mod fade on it is undone. No music manager class was found (the music source is probably scene-driven).
- Radio: `states = clips + 1`; state 0 is off, 1..n are channels; `Use()` steps the state and wraps to off. `SetState` plays the on/switch/off sounds and fires the `OnTurnOn`/`OnTurnOff` outputs. `RadioEnabled` only mutes: `Use()` still advances the channel. Saved: `soundscapeAreaId`, `CurrentState`, `RadioEnabled`.
- Camera: `ThirdPersonCameraController` reads the `Mouse X/Y` and `RightStick*` axes times `mouseSensitivitySetting` in `LateUpdate`; gates: `controlsDisabled`, `followPlayerDisabled`, `menuEnabled`, chair clamp. `ThirdPersonCameraCollision.FirstPerson()/ThirdPerson()` set `firstperson` and fire the static `OnCameraModeChange`; zoom is blocked by the `DisableZoom` list or `FirstPersonOnly(true)`. `PlayerCamera.Awake` forces depth texture mode on every camera. `SetCameraFOV.Update` reassigns `Camera.fieldOfView` from `DisplaySave.displaySave.fov` every frame.
- Quality: the `OptionsMenu` apply code saves each value through `QualitySave.saveQuality` (`Quality.json`), calls `QualitySettings.SetQualityLevel`, then overrides shadows by preset (distance 0/10/30/150, cascades 0/2/4/4, pixel lights 1/2/3/4) and anisotropic filtering (lodBias 1/2/3), plus texture mip limit, run in background, resolution and VSync. Antialiasing and Post FX levels are applied through `EffectsController` (`ChangeAntialiasing`, `ChangePostProcessBlur` toggles depth of field, `ChangePostProcessProfiles` swaps volume profiles); it also wraps Beautify and volumetric clouds (`HxVolumetricCamera`). `PostProcessingEffects` reads the lens distortion setting in `Start`.
- Lights: `LightCullZone.ChangeIntensityMultiplier` (static) is global dimming. `LightFlicker` flips lights on and off in a coroutine and restores the intensities it stored at start each cycle, so external intensity changes are overwritten.

## Mod guidance

A mod FOV must write `DisplaySave.displaySave.fov`, not the camera. Set a quality override after the options apply (it overwrites shadows after `SetQualityLevel`) or patch the preset tables. To gate the radio block `Radio.Use`/`SetState`, not just `RadioEnabled`. Pair every `OnCameraModeChange` subscription with a removal in `OnDestroy`. Do not write the `Soundscape`/`BackgroundSoundscape` mixer parameters directly.

## Unverified

What enables the "wasteland" sound flag; which class plays music; the exact name of the `OptionsMenu` apply method.
