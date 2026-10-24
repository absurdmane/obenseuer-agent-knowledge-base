# Periodic rendering work (reflection probes)

Scope: game code that re-renders reflection probes on a timer; suspects for regular frame hitches. Not covered: the render
pipeline itself, probe/asset values (intervals set in the scene are runtime data).
Status: partly verified
Related: [INDEX.md](INDEX.md)

## Reflection probe timers

- `ReflectionProbeUpdater` (scene component, list `reflectionProbes`): subscribes to `TimeOfDayAzure.SecondsPassed`, adds the
  game seconds and calls `RenderProbe()` on every probe in the list once more than 300 game seconds passed (const
  `updateInterval` = 300, the code compares with a literal 300). Also once in `Awake`. So the real-time period depends on the
  game time speed (`TimeOfDayAzure.actualSpeed` = `dayLenghtInMinutes` / `timeSpeedOverrideFactor`; the field default is 24,
  the scene value is unknown).
- `UnityEngine.AzureSky.AzureEnvironmentController`: when `state` is On and `refreshMode` is `ViaScripting`, calls
  `reflectionProbe.RenderProbe()` every `refreshInterval` real seconds (`Time.deltaTime`; field default 2, the scene value
  unknown); `EveryFrame` renders every frame. It also writes `RenderSettings` ambient values every frame.
- `RefreshCubemapInTrigger`: `RenderProbe()` on its probe when something enters its trigger.

`RenderProbe()` only queues the render: the cost lands in Unity's render loop, not in the calling script, so a script-level
timer (a spike profiler) shows a slow frame with little script time.

## Suspect: a periodic outdoor hitch

A player sees a hitch every ~20 s, outdoors only, by day only (not at night), with no mods but a dev menu; GC runs every ~7 s
and doesn't line up [verified]. Day-only fits a probe render: six cubemap faces with the sun's shadows. Unconfirmed: force a
probe render by day and see whether it hitches the same way (a debug script that renders the probes now; it also prints
each timer's real-time period: `ReflectionProbeUpdater` = 300 × `actualSpeed` × 60 / 86400 s, so ~20 s means a ~96-minute day).

Spike profiler capture (outdoors, Open Sewer Tenement) [verified]: two slow frames of ~36 ms, 0.14 s apart (one hitch);
timed scripts only 1.3–1.4 ms of each, top method `Cable_Procedural_Simple.Update` 0.2 ms; Unity counters in those frames:
45–49k draw calls, 44–48k batches, 28–31M triangles. So the cost is in rendering, not in scripts. The draw-call count of a
normal frame is not known yet, so the probe theory is still unconfirmed.

Forced render by day (the debug script): the hitch has the same shape as the periodic one in the
FPS window [verified, by eye]. So the periodic outdoor hitch is most likely a timed reflection probe render. Which of the two
timers it is, and its real-time period, not recorded yet.

Fix [verified]: setting `ReflectionProbe.timeSlicingMode = IndividualFaces` on the timed probes (those in
`ReflectionProbeUpdater.reflectionProbes` and `AzureEnvironmentController.reflectionProbe`) at runtime removes the hitch;
the reflection update looked "more smoothly". Unity spreads a time-sliced render over several frames.

## Mod guidance

- Set `timeSlicingMode` on the probes after the scene loads (they live in scene objects); keep the game's value to put back on
  unload. It's a component setting, not saved, so removing the mod leaves nothing behind.
- `RenderProbe()` is still called by the game's timers: no need to patch them.

Runtime data (Open Sewer Tenement, runtime log) [verified]: the hitch is `ReflectionProbeUpdater` — period ~20 s real
(300 game s; so `actualSpeed` ~96). It holds 16 probes, each 128 px, HDR, shadow distance 100 m, far clip 1000 m, game's
`timeSlicingMode` = `AllFacesAtOnce`. 16 probes × 6 faces = 96 scene renders in one frame (fits the 45–49k draw calls).
The Azure environment controller's probe is off (`state` not On).
With `IndividualFaces`, small dips remain; `shadowDistance = 0` on the probes removes most of them, half resolution (64 px)
or half far clip (500 m) change little [verified, by eye in the FPS window]. Shadows in the probe render are the main cost,
which also fits "no hitch at night".

## Changes in 0.5.13 `[code]`

`ReflectionProbeUpdater.Awake` now calls `ApplySettings` (mode Realtime, refresh mode ViaScripting, `timeSlicingMode` from a serialized field with default `IndividualFaces`, `shadowDistance` from a serialized field with default 0) and then renders all probes, so the time-slicing and shadow-distance workaround described above is the updater's own default for its probes; the mod guidance is only needed for the `AzureEnvironmentController` probe. The periodic update goes through `UpdateReflectionProbes`; in play mode it starts `UpdateProbesOneByOne`, which renders each probe and waits up to 30 frames for `IsFinishedRendering` before the next (a call during a run sets `updatePending`); outside play mode it renders all at once. `SunDirection.instance` is a new static singleton.
