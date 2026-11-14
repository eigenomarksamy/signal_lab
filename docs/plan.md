# signal lab plan

an interactive, synthetic radar-like sensor and tracking sandbox. users watch moving targets, see noisy or missing measurements, and compare raw detections with estimated trajectories. the simulation and the tracking run in C++20, compiled to WebAssembly for the browser and natively for tests and benchmarks, with no server. it is planned for `signal.eigenomar.com`.

nothing is built yet. this plan holds the bootstrap (B1), the MVP (M1–M5) and phase 2 (L1–L5), and the shared decisions they rely on. the signal lab is the second of three C++ labs, after `pathfinding_playground` and before `gravity_engine`, each in its own repository. it reuses the pathfinding playground's bootstrap approach, not its code. work across the three, such as their release order or a library they might later share, is coordinated in the ecosystem, not planned here.

## how this plan works

- **tickets.** every ticket has an id, a lane prefix and a number, never reused or renumbered. a split ticket keeps its number and adds a letter (M2a, M2b). each ticket has a stable anchor, its id in lowercase (`#m1`), above its heading, which doesn't change when the heading does.
- **lanes.**

  | prefix | lane |
  | --- | --- |
  | B | the bootstrap: native, WebAssembly and web builds, CI and the deploy |
  | M | the MVP, released at `signal.eigenomar.com` |
  | L | phase 2: later work, after the MVP's release |

- **each ticket** says what to do, what it needs, and when it is done. its "done when" is its acceptance criteria. a need is a hard need: the ticket starts once it is done. a ticket in another repository is named with that repository, as `pathfinding_playground` B1.
- **issues.** every open ticket gets one GitHub issue in this repository. the issue's title starts with the ticket's id, and its body links the ticket's anchor here without copying it. the ticket holds the scope and history, and the issue holds status, priority and blockers.
- **shared decisions** bind this lab and other repositories. this plan states each outcome it relies on in one line and cites the decision by its id. a change to one is made in the decision first.
- **adding work.** new work goes here as a ticket in a lane. this repository is public, so confidential work never goes in this file. the ecosystem's shared rules, which `CLAUDE.md` imports, say where it goes.
- **when work ships,** update the ticket's status in the index, add a result with the commit to the ticket, and add a line to the [log](#log).

## shared decisions

the three C++ labs share their constraints, stack, architecture and definition of done as shared decision SD3, which makes no shared package mandatory. the outcomes this lab relies on:

- **its own app:** it works on its own at its own URL, with no account, login, payment or database, and needs no backend, external API, uploaded data or real-time third-party service.
- **usable at once:** the default experience starts with sample scenarios and sensible parameters.
- **controls:** play, pause, reset, and step wherever stepping is meaningful.
- **seeds:** anything random takes a reproducible seed, and a preset gives the same outcome for the same configuration.
- **honest models:** educational approximations are clearly told apart from validated models; this lab is a teaching simulation.
- **access:** controls work by keyboard and the layout is responsive; the visualization may favor desktop, with a working mobile fallback.
- **motion:** reduced-motion preferences are honored for nonessential animation, and nothing flashes excessively or animates without a way to pause it.
- **privacy:** no analytics or tracking by default; only optional preferences are kept, locally.
- **identity:** the three labs look consistent, without a shared frontend package before one is needed.
- **footer:** the ecosystem's button to `https://eigenomar.com`, labelled `eigenomar.com`, a plain link in the app's own button style, opening in a new tab here, since leaving would lose a configured scene.
- **C++20:** the engine is C++20; C++23 features wait until the pinned compiler and Emscripten are shown to support them.
- **separation:** the engine is separate from rendering, browser events and framework code; simulation and filter state stay in C++, and TypeScript does the controls, state coordination and drawing.
- **the boundary:** a small, documented interface to JavaScript, through Embind at first, transferring batched state rather than one call per target or detection.
- **two builds:** one engine source compiles to a native build and a WebAssembly build.
- **C++ habits:** RAII, clear ownership, `const` correctness, explicit types, `std::span` and standard containers, without premature templates or allocations in frame-critical paths.
- **steps:** the simulation advances in fixed steps, decoupled from the visual frame rate.
- **limits:** inputs are bounded and invalid parameters handled, so no scenario can freeze the tab.
- **threads:** single-threaded at first, with a standard Web Worker for long computation if needed; C++ pthreads aren't a baseline.
- **metrics:** performance is measurable, here as execution time, simulation steps and estimation error against the synthetic ground truth.
- **portability:** it builds, runs and tests on Windows and Linux.
- **the stack:** CMake, Emscripten and GoogleTest with CTest for the engine; React, TypeScript and Vite with Canvas 2D for the app, WebGL2 only if profiling justifies it, tested with Vitest and Playwright; clang-format, clang-tidy, compiler warnings and sanitizers in native CI; GitHub Actions for CI and deploys; Cloudflare Workers static assets on its own subdomain for hosting.
- **dependencies:** few and pinned, through one dependency-management method chosen when implementation starts, such as CMake's `FetchContent` at pinned revisions. Eigen is used for the tracker's matrix operations if it meaningfully simplifies the tracker, and no heavyweight framework otherwise.
- **the architecture:** a React and TypeScript UI, over a typed Wasm adapter (input validation and conversion, batched snapshots or events, the module's lifecycle and errors), over a WebAssembly module of the C++20 engine, which a native test and benchmark executable also builds.
- **the layout:** the suggested one, with `cpp/` (include, src, bindings, tests), `web/`, `benchmarks/`, `docs/`, `CMakeLists.txt`, `CMakePresets.json`, `wrangler.jsonc` and `.github/workflows/` beside `CLAUDE.md`, `project.yaml`, `README.md`, `LICENSE` and `.gitattributes`.
- **no shared library yet:** common code is extracted only once at least two labs need it.
- **done:** each public release meets SD3's definition of done, which [M5](#m5)'s done-when cites.

## index

| ticket | title | needs | status |
| --- | --- | --- | --- |
| [B1](#b1) | bootstrap: native, WebAssembly and web, built and deployed from GitHub Actions | none | todo |
| [M1](#m1) | simulation clock, target dynamics and seeded sensor model | B1 | todo |
| [M2](#m2) | single-target Kalman filter, with tests, uncertainty and diagnostics | M1 | todo |
| [M3](#m3) | multi-target extension and simple association | M2 | todo |
| [M4](#m4) | scene, tracks and plots UI, presets and parameter explanations | M3 | todo |
| [M5](#m5) | release: validation, performance measurements, deployment and README | M4 | todo |
| [L1](#l1) | synthetic range-Doppler signals and an FFT visualization | M5 | todo |
| [L2](#l2) | detection pipeline with thresholding or CFAR, feeding the tracker | L1 | todo |
| [L3](#l3) | extended or unscented Kalman filtering for nonlinear sensors | M5 | todo |
| [L4](#l4) | more capable association and multi-object tracking | M5 | todo |
| [L5](#l5) | controlled benchmarks: accuracy, clutter resilience and computational cost | M5 | todo |

## bootstrap

the builds, checks and deploy every later ticket uses, set up once in this repository. the pathfinding playground's bootstrap, `pathfinding_playground` B1, works out the approach for all three labs: this ticket reuses that approach and the lessons in its result, not its code, so nothing here depends on that repository.

<a id="b1"></a>

### B1 · bootstrap: native, WebAssembly and web, built and deployed from GitHub Actions

- **do:** the checklist below, following `pathfinding_playground` B1's approach. this ticket adds the repository's first code, its CI and its deploy.
  - **portability:** `CMakePresets.json` has native presets for Windows (MSVC or clang-cl) and Linux (GCC or Clang), and a WebAssembly preset through Emscripten. Emscripten comes from `emsdk`, which runs on Windows natively. no build step exists only as a `.sh` or `.ps1` script, and nothing needs WSL. CI runs the native tests, with sanitizers, and the WebAssembly build on `ubuntu-latest`.
  - **delivery:** Cloudflare Workers serve the static assets on `signal.eigenomar.com`. GitHub Actions builds the WebAssembly module and deploys the result, since Cloudflare's own build doesn't run Emscripten, on every merge to `main`.
- **needs:** nothing in this repository. it comes after `pathfinding_playground` B1, whose approach it reuses: a planning dependency, so its issue lists that ticket's issue as a `planning:` line under related work, not as a blocker.
- **done when:**
  - [ ] a minimal C++20 library builds with CMake, with native tests and one benchmark target.
  - [ ] that library compiles with Emscripten, and TypeScript calls a simple function it exports.
  - [ ] a stable typed adapter handles Wasm initialization, errors and data transfer.
  - [ ] linting and formatting, native sanitizers and CI checks are configured.
  - [ ] it builds natively on Windows and in CI on Linux, with Emscripten from `emsdk` on both, and the native tests pass on Windows started from `c:\` as well as from `C:\`.
  - [ ] a minimal Vite and React shell has a canvas, basic controls and an about/source panel.
  - [ ] a production static deployment on Cloudflare, built and deployed from GitHub Actions, answers at `signal.eigenomar.com`.
  - [ ] the README lists what to install on each platform: the compiler, CMake, `emsdk` and Node. `CLAUDE.md`'s commands section lists the setup, build, test and run commands, each working on Windows and Linux.
  - [ ] in the same change as the deploy, `project.yaml`'s `delivery` says `ci: github-actions`, `deployment: cloudflare` and `deploys_on: merge`, its `links.deployment` is `https://signal.eigenomar.com`, and `CLAUDE.md`'s delivery section says what CI runs and that every merge to `main` deploys. the ecosystem's validator passes.
  - [ ] the result records where this bootstrap differs from `pathfinding_playground` B1's approach, and why, for the ecosystem's conventions on native builds.

## MVP

stage 2 of the labs' workflow: one polished, bounded release at `signal.eigenomar.com`. the simulation, the filter and their tests come before the interface, so M1–M3 come before M4. the MVP's acceptance criteria are spread over M1–M5's done-whens, each in the ticket that delivers it, and M5 releases the MVP once they all hold.

**scope:**

- 2D scene containing simulated moving targets with configurable position and velocity.
- configurable sensor update rate, detection probability, position noise, and false-alarm/clutter level.
- toggles for ground-truth trajectories, sensor detections, and estimated tracks.
- implement a **constant-velocity linear Kalman filter** for a target with state `[x, y, vx, vy]` and measurements `[x, y]`.
- display covariance as uncertainty ellipses, plus measurement and estimation error plots.
- introduce simple, explicitly documented **nearest-neighbor measurement association** for multiple targets; optionally begin with a single-target tutorial preset to keep behavior understandable.
- simulation presets (clean measurements, heavy noise, missed detections, crossing targets) and deterministic random seeds.
- controls: play, pause, step, reset, update frequency, process noise, and measurement noise.

**model boundaries and constraints:**

- this is a **teaching simulation**, not a validated radar product or implementation of operational tracking software.
- first simulate detections directly from ground-truth positions with noise; do **not** claim that these are detections reconstructed from electromagnetic returns.
- define units explicitly (for example meters and seconds), including sampling time `dt` and noise/covariance units.
- use a well-defined motion model, observation model, process covariance `Q`, and measurement covariance `R`.
- handle missed measurements through prediction-only steps; clearly show track creation/deletion or simplify the MVP to known tracks.
- when using nearest-neighbor association, document identity swaps and ambiguity during crossing. do not imply guaranteed tracking correctness.
- use numerically careful matrix operations (for example the Joseph covariance update form when needed) and test covariance symmetry and stability.
- avoid presenting arbitrary metrics as objective sensor accuracy. expose clearly defined error measures against available synthetic ground truth.
- scope all data generation to the browser; no real-world sensing input or upload is needed for the MVP.

**proposed C++ modules:**

```text
target dynamics / world simulation
sensor measurement generator
seeded stochastic noise + clutter
detection and association model
constant-velocity Kalman filter
track state and uncertainty
error metrics / snapshots
```

<a id="m1"></a>

### M1 · simulation clock, target dynamics and seeded sensor model

- **do:** the simulation clock, advancing in fixed steps of a sampling time `dt` and decoupled from any frame rate; the 2D world of moving targets, each with a configurable position and velocity under the constant-velocity motion model; and the sensor measurement generator, which makes detections directly from the ground-truth positions, with seeded position noise, a detection probability, a false-alarm/clutter level and a configurable update rate. every parameter has an explicit unit, such as meters and seconds.
- **needs:** B1.
- **done when:**
  - with zero noise, a detection probability of 1 and no clutter, a target's measurements follow its straight-line ground truth exactly, and a test checks it.
  - the same seed and configuration produce identical measurements, clutter included, and a test checks it.
  - over a long seeded run, the share of missed detections, the spread of the position noise and the amount of clutter match their configured values within a tolerance the test states.
  - each parameter's unit is documented in the engine's interface, and an invalid value, such as a negative noise, a probability outside 0 to 1, a `dt` that isn't positive, or a target count over its cap, is rejected with an error.

<a id="m2"></a>

### M2 · single-target Kalman filter, with tests, uncertainty and diagnostics

- **do:** the constant-velocity linear Kalman filter for one target, with state `[x, y, vx, vy]` and measurements `[x, y]`, its motion and observation models, and its process covariance `Q` and measurement covariance `R`, all in M1's units. a missed measurement runs a prediction-only step. the covariance update is numerically careful, using the Joseph form where needed. the track's state and uncertainty, with the uncertainty ellipse its covariance gives, and clearly defined error measures against the synthetic ground truth. whether Eigen meaningfully simplifies the matrix operations is decided here, under shared decision SD3.
- **needs:** M1.
- **done when:**
  - prediction and update match independently computed reference cases within a tolerance the test states.
  - over long runs under reasonable parameters, the covariance stays symmetric and positive semi-definite and the state stays finite, and tests check it.
  - a missed measurement makes the covariance grow, and a larger measurement noise gives a larger steady-state uncertainty and error, and tests check both.
  - the uncertainty ellipse of a known covariance has the expected axes and orientation, and a test checks it.
  - each error measure is defined in the engine's documentation and tested on a known case.
  - the result records whether Eigen is used, and why.

<a id="m3"></a>

### M3 · multi-target extension and simple association

- **do:** several targets, each with its track, and the simple, explicitly documented nearest-neighbor association of measurements to tracks. tracks are created and deleted visibly, or the MVP is simplified to known tracks, and the result records which. each step's association records where it was ambiguous and where an identity swapped, so nothing is hidden. each step yields a rendering-neutral snapshot: ground truth, detections, tracks with their covariance, associations with their ambiguities, and the error measures.
- **needs:** M2.
- **done when:**
  - with several targets and clutter, the same seed and configuration produce identical measurements, associations and tracks, and a test checks it.
  - well-separated targets under moderate noise each keep their own track, and a test checks it.
  - on a crossing-targets scenario, the snapshots mark the ambiguous steps and any identity swap, and a test checks it.
  - the rules for creating and deleting tracks are documented and tested, or the known-track simplification is documented.

<a id="m4"></a>

### M4 · scene, tracks and plots UI, presets and parameter explanations

- **do:** the MVP's scope in the app, on B1's shell: the 2D scene, with toggles for the ground-truth trajectories, the sensor detections and the estimated tracks; the covariance as uncertainty ellipses; the measurement and estimation error plots; the presets, clean measurements, heavy noise, missed detections and crossing targets, each with its seed, and optionally a single-target tutorial preset; the controls, play, pause, step, reset, update frequency, process noise and measurement noise, beside the sensor's and targets' parameters; and an explanation of each parameter, with its unit. the engine's Embind interface hands the app M3's snapshots in batches, through B1's typed adapter.
- **needs:** M3.
- **done when:**
  - each preset loads with its seed, play, pause, step and reset work on it, and reset returns it to its starting state.
  - the toggles show and hide the ground truth, the detections and the tracks, each drawn so the three are told apart.
  - raising the measurement noise or lowering the detection probability visibly widens the uncertainty ellipses and raises the error plots.
  - where association is ambiguous, as on the crossing-targets preset, the app marks it at that step and explains it, and identity swaps are shown, not hidden.
  - every parameter's control shows its unit and explanation, and accepts only values within its bounds.
  - snapshots cross the Wasm boundary in batches, never one call per target or detection, the native and WebAssembly builds pass the same correctness fixtures, and invalid input comes back through the adapter as a typed error, not a crash.
  - the footer's `eigenomar.com` button opens `https://eigenomar.com` in a new tab, so a configured scene survives.

<a id="m5"></a>

### M5 · release: validation, performance measurements, deployment and README

- **do:** the MVP's validation on the deployed build, its performance measurements, an accessibility pass, the app's tests, the MVP's deployment at `signal.eigenomar.com` through B1's pipeline, and the README.
- **needs:** M4.
- **done when:**
  - shared decision SD3's definition of done for each public release holds for everything this repository delivers. its items that other repositories deliver once the demo is live, the site's software entry among them, follow this ticket.
  - the deployed demo, the public README and the source link work without a backend.
  - the deployed demo stays interactive at a documented default target count, between 5 and 20 as suggested, with the target count and every other input capped.
  - the engine's step time is measured natively and in WebAssembly, apart from Canvas drawing, and every performance figure the README gives names the hardware, browser and conditions it was measured under.
  - the README, or a document it links, states the units, the motion and observation models, `Q` and `R`, the assumptions and limitations, how ground truth, detections and estimates differ, and that this is a teaching simulation, not a validated radar product or operational tracking software.
  - every control works by keyboard, the layout has a working mobile fallback, and a reduced-motion preference stops nonessential animation.
  - Vitest covers the app's logic and a Playwright smoke test the deployed build, both in CI.
  - M1–M4's done-whens hold on the deployed build.

## phase 2

not MVP. it starts once the MVP is released ([M5](#m5)): the MVP begins with documented synthetic detections and tracking, and signal processing comes here, later. each ticket uses the MVP's simulation, tracker, snapshots and error measures where it can.

<a id="l1"></a>

### L1 · synthetic range-Doppler signals and an FFT visualization

- **do:** synthetic **range-Doppler** or 1D/2D signal generation and an FFT visualization.
- **needs:** M5.
- **done when:** the same seed generates the same signal, a target at a known range and velocity peaks where expected in the spectrum, and tests check both. the app shows the spectrum and says the signal is synthetic.

<a id="l2"></a>

### L2 · detection pipeline with thresholding or CFAR, feeding the tracker

- **do:** a documented detection pipeline with thresholding or **CFAR** on L1's signals, and its detections connected to the tracking stage.
- **needs:** L1.
- **done when:** tests check that the detector finds a target at its known cell and that its false-alarm rate on noise alone is within a tolerance the test states. its detections feed the MVP's tracker, and the app documents the pipeline and its parameters.

<a id="l3"></a>

### L3 · extended or unscented Kalman filtering for nonlinear sensors

- **do:** extended or unscented Kalman filtering for nonlinear sensors.
- **needs:** M5.
- **done when:** on a nonlinear sensor model, the filter matches reference cases within a stated tolerance and its covariance stays symmetric and positive semi-definite, and tests check both. the app runs it beside the linear filter on the same seed and explains the difference.

<a id="l4"></a>

### L4 · more capable association and multi-object tracking

- **do:** more capable track association and multi-object tracking methods than nearest-neighbor.
- **needs:** M5.
- **done when:** each method runs beside nearest-neighbor on the crossing-targets scenario with the same seed, tests compare their identity swaps and errors against the ground truth, and the app shows the comparison and states each method's assumptions and limits.

<a id="l5"></a>

### L5 · controlled benchmarks: accuracy, clutter resilience and computational cost

- **do:** controlled benchmarks of accuracy, clutter resilience, and computational cost.
- **needs:** M5.
- **done when:** the benchmarks run from the repository with fixed seeds and documented conditions, measure the error against the ground truth across clutter levels and the engine's step time natively and in WebAssembly, and their published results name the hardware, browser and conditions, without presenting them as objective sensor accuracy.

## log

- 2026-11-14 · plan · created by the ecosystem setup session, from the three C++ labs' shared plan, where this lab's part is now a link here. the bootstrap that the labs' plan has this lab reuse from the pathfinding playground became B1, after `pathfinding_playground` B1 as a planning dependency. its section became the MVP's scope and constraints, with stage 2's five steps as M1–M5, and its phase 2 L1–L5, in the same order. the MVP's seven acceptance criteria are in M1–M5's done-whens. the shared constraints and the definition of done are shared decision SD3, cited by its id. `project.yaml` validates, with no CI, releases or deployment until B1, `CLAUDE.md` imports the ecosystem's shared rules, and `.gitattributes` holds `* text=auto eol=lf`. nothing else in the repository changed.
