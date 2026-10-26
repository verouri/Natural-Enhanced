[README.md](https://github.com/user-attachments/files/33983266/README.md)
# Natural Enhanced

Natural Enhanced is a C# weather and atmosphere enhancement mod for **Flyout** by Stonext Games. Built for MelonLoader, it works with the simulator's existing Expanse volumetric clouds and aerial perspective, HDRP height fog, sunlight, ocean shader, and 3D precipitation to create a coherent weather layer with smooth transitions.

The mod changes the environment only. It does not modify the camera, aircraft geometry, controls, or flight physics.

The 1.2 visual pass takes inspiration from **GTA V's NaturalVision Evolved**: natural contrast, teal and orange color separation, cinematic bloom, wet surfaces, reflective water, and atmospheric depth. These effects are implemented from scratch using Flyout's HDRP post-processing, Expanse sky, cloud and fog systems, and ocean material APIs. NVE itself, its assets, and its timecycle are not redistributed or copied.

![Natural Enhanced storm](docs/shot-storm-final.png)

## Features

- **Volumetric clouds:** Controls Expanse's `ProceduralCloudVolume`, including coverage, density, self-shadowing, silver lining, multiple scattering, ambient light, shadow quality, raymarch steps, temporal denoising, and LOD distances. These are Flyout's raymarched volumetric clouds, not a separate billboard layer.
- **Weather presets and smooth transitions:** Choose Clear, Partly Cloudy, Scattered Cumulus, Broken Clouds, Overcast, Light Rain, Heavy Rain, Thunderstorm, Fog, or Haze. Presets ease values over a configurable transition.
- **3D rain:** World-space GPU particle layers use a clone of the game's rain material, follow real wind, and include near and far layers with optional surface splashes. Rain is not rendered as a full-screen overlay.
- **Thunderstorms:** Darker convective cloud settings, heavy rain, gusty wind, and restrained randomized lightning. Lightning briefly raises cloud ambient and silver illumination and adds a distant flash.
- **Atmospheric depth:** Expanse aerial perspective works with a dedicated HDRP height-fog volume. Fog density and height can be adjusted in Dynamic and preset modes, and the height-fog backend can be toggled in the panel. Fog is applied after the simulator's weather update.
- **Ocean and reflections:** Finds the game's `OceanHDRP` / `Shader Graphs/OceanShader` material, enables HDRP screen-space reflections on High and Medium, and adjusts water color, roughness, wave speed, distortion, and clarity for clear, overcast, and storm weather. It does not use a replacement water shader.
- **Cinematic color grade:** A dedicated HDRP global Volume controls contrast, saturation, exposure, white balance, color filter, bloom, screen-space ambient occlusion, vignette, and film grain. The grade responds to weather and sun elevation, with warm highlights and cool shadows in clear light, cold desaturated storms, and a restrained night grade. The panel includes a **Look** tab and grade-strength slider.
- **Snowfall:** Wakes Flyout's real `Weather.snow` emitter, follows the aircraft with configurable intensity, and restores its original state when disabled.
- **Explosion effects:** Scales the real `ExplosionFX` particles, shockwave, and HDRP light at the actual `Explode` call. A fallback flash is used only if the game has no light.
- **Cloud structure and detail:** Exposes Expanse structure, fine-detail, and warp controls for more defined clouds.
- **Rain-wet surfaces:** In rain, HDRP Lit/TerrainLit materials for terrain, runways, and aircraft receive increased smoothness, a small metallic response, and darker base color. Captured values are restored when dry. No shine overlay is used.
- **Sunlight:** Sun color temperature and intensity follow the actual sun elevation, with golden-hour warmth, storm dimming, and cloud illumination. The mod drives HDRP `HDAdditionalLightData` and Expanse `LightControl`.
- **F6 weather panel:** A draggable in-flight panel with preset buttons, cloud coverage and density, silver lining, shadows, ambient light, fog, haze, rain, lightning, color-grade controls, quality settings, and diagnostics.
- **Quality settings:** Low, Medium, High, and Ultra presets, plus adaptive performance that ignores minimized or background frames. Adaptive quality stops at Medium by default to preserve cloud self-shadowing and water reflections during frame drops; Low remains available manually.
- **Configuration and diagnostics:** JSON configuration in `UserData/NaturalEnhanced`, a rotating log, screenshots (F10), and a diagnostics dump (F11).

## Requirements

- Flyout `0.2504` (Unity `2022.3.62f3`, IL2CPP x64)
- MelonLoader `0.7.1` (net6)
- Windows x64

See [docs/TECHNICAL.md](docs/TECHNICAL.md) for the verified API discovery.

## Installation

1. Install MelonLoader for Flyout.
2. Copy `NaturalEnhanced.dll` into the Flyout `Mods` folder. The default Steam path is:

   ```text
   C:\Program Files (x86)\Steam\steamapps\common\Flyout\Mods\
   ```

3. Start Flyout and press **F6** in flight to open the weather panel.

On first launch, the mod creates `Flyout\UserData\NaturalEnhanced\config.json`.

See [INSTALL.md](INSTALL.md) for the full instructions.

## Default controls

| Key | Action |
| --- | --- |
| **F6** | Open or close the draggable weather panel |
| F4 | Apply the next weather preset instantly |
| F7 | Apply the next weather preset with a 90-second transition |
| F8 | Follow simulator weather or lock a preset |
| F9 | Cycle quality |
| F10 | Save a screenshot |
| F11 | Save a diagnostics dump |

## Configuration

Settings are stored in `UserData/NaturalEnhanced/config.json`. The panel is the primary interface; editing the JSON file is optional. See [docs/CONFIGURATION.md](docs/CONFIGURATION.md) for the available fields.

## Build

From the repository root, run:

```powershell
dotnet build -c Release
```

Or use the build script:

```powershell
.\build.ps1
```

The project targets `net6.0` and references the assemblies generated by MelonLoader under `MelonLoader\Il2CppAssemblies`. If Flyout is installed elsewhere, adjust `<GameDir>` in `NaturalEnhanced.csproj` or pass `-p:GameDir=...`.

## Verification status

The following functionality was verified during a live Flyout flight (RTX 3060, 1920×1080, HDRP):

- The DLL loads under MelonLoader 0.7.1 and discovers the world scene.
- Expanse `ProceduralCloudVolume`, `TextureCloudPlane`, `AtmosphereLayer`, `GlobalSettings`, `LightControl`, and `Il2Cpp.Weather` are found.
- Harmony prefixes on `ProceduralCloudVolume.Update` and `LightControl.Update`, and a postfix on `WorldScene.Update`, install and execute.
- Weather presets visibly change cloud coverage and density, sky state, and rain. Clear and Thunderstorm produce clearly distinct environments.
- Fog is visible and controllable. A dedicated HDRP height-fog volume renders layered ground and distance fog; the Fog preset and density/height sliders change visibility in Dynamic and preset modes. An earlier double-fog/black artifact was fixed by leaving the shipping Expanse fog layer unchanged.
- The ocean material is found (`water=1`). Water smoothness, roughness, color, scale, and wave speed change with weather, and two HDRP `ScreenSpaceReflection` volume components are enabled on Medium/High. Live water pixels were not captured from the inland test airfield, so ocean appearance still needs a coastal or high-altitude check.
- 3D rain particle layers are created and render in world space.
- The NVE-inspired grade volume is created and applied at runtime. Toggling `grade 0/1` visibly changes contrast, sky and terrain saturation, exposure, and highlight rolloff. Test captures showed a clear difference with no shader errors.
- Rain-wet material discovery finds the HDRP Lit/TerrainLit surface set (164 materials in the test scene), applies wet smoothness and color, and restores the captured values.
- Cloud terrain shadows are enabled at High (reduced strength) and Ultra (full strength); High/Ultra diagnostics report `castShadows=True`.
- Lightning events fire on the Thunderstorm preset.
- The F6 panel renders, including the Look tab. Measurements showed roughly 38–50 fps at High and 50–59 fps at Medium in the tested scene.

**Version notes:**

- **1.4.0 — NVE reference pass:** The supplied NVE package was inspected. Its default ReShade color preset is neutral; the visible look comes from GTA timecycle, weather, and water data. Natural Enhanced now uses original timecycle-derived curves for exposure, low-sun bloom, water specular and color, and sky/fog tint, without copying NVE assets. See [docs/NVE_REFERENCE.md](docs/NVE_REFERENCE.md).
- **1.3.1 fixes:** Water-detail normals are generated from the game's real `Texture` wave map, the grade can no longer emit `NaN`, and debug toggles are case-insensitive. A test camera was used only for ocean inspection and was removed; the shipped DLL contains no camera writes.

**Not yet verified:** The 1.3 ocean/coastline pass; clean Medium/High/Ultra FPS after the heavier SSR settings; golden-hour and night captures from the final build; long-duration resource-leak testing; every preset in every lighting condition; all aircraft/cockpit combinations; and VR. During the final ocean run, the test game stalled in the hangar/editor and the in-game camera repeatedly clipped inside the aircraft, so those captures are not claimed. See [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) and the limitations below.

## Limitations

- The volumetric clouds use Expanse's shipped raymarcher, driven by this mod. No new cloud or compute shader is included, so cloud rendering quality is bounded by Expanse and the game's settings.
- Custom shaders and bundles are not included. Rain uses the game's existing rain material; wet surfaces adjust HDRP Lit/TerrainLit PBR values rather than using a replacement shader. Cockpit-droplet shaders are not implemented. Fog uses HDRP's built-in height-fog component, not a volumetric light-shaft simulation.
- The NVE-inspired look is an original HDRP volume grade, not a copy of NVE's LUT, timecycle, or ReShade stack. It cannot reproduce GTA V's art assets, terrain density, vehicle shaders, or city lighting.
- The 1.3 wave-detail normal layer is generated from the game's texture and is subtle. It does not add mesh displacement, foam, caustics, or planar reflections.
- Water enhancement drives the existing `OceanHDRP` material and HDRP SSR. It does not replace the ocean shader, add foam, or alter wave geometry.
- Wet surfaces add sheen and a darker PBR response, but do not add physical puddles, ripples, or planar road reflections.
- Lightning uses cloud ambient and silver illumination, sun pulses, and a distant flash light. It is not a full volumetric shadowed lightning simulation. Thunder audio is not included.
- Live real-world weather is not integrated. In Dynamic mode, the mod follows the simulator's own weather; presets are local.
- Cloud terrain shadows are enabled at High (65% strength) and Ultra (full strength). Medium keeps cloud self-shadowing but disables terrain casting for performance.
- Rain splashes use a ground projection, not a full collision raycast, so they are approximate on steep terrain.
- Harmony patches are required. A future Flyout update that renames the patched methods can disable cloud or lighting application until the mod is updated.
- F6 is fixed in this build and may overlap a game binding. The JSON config stores the intended key, but the panel shortcut is currently F6.

## Source layout

```text
NaturalEnhanced.csproj
Source/
  NaturalEnhancedMod.cs       entry point, lifecycle, hotkeys, Harmony install
  Config/                     JSON config
  Core/                       game-reference discovery, quality presets, baselines
  Weather/                    weather profiles and transitions
  Rendering/                  clouds, atmosphere, HDRP fog, water, NVE grade, wet surfaces, lighting, rain, lightning
  UI/ControlPanel.cs          F6 draggable weather panel
  Patches/                    Harmony prefixes/postfixes
docs/                         technical notes, configuration, troubleshooting
tools/                        development probes (not part of the mod)
```

## Credits and legal

- Flyout and its Expanse/HDRP content are the property of Stonext Games and their respective licensors. This mod ships no game assets; it interacts with the installed game at runtime.
- Expanse is the volumetric cloud and atmosphere middleware already present in Flyout.
- Natural Enhanced source is released under the MIT License (see [LICENSE](LICENSE)).
- No proprietary assets or copied code are redistributed.
