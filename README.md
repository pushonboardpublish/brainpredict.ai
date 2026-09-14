# brainpredict.ai

A prediction platform with two live channels — **signal** and **language** —
built on one shared pipeline shape: `ingest → normalize → model → serve → store`.

The attached `brainpredict-app.jsx` is the working demo of this: a Signal
tab (synthetic waveform + naive extrapolation, clearly labeled as
simulated — no real EEG/fMRI hardware is wired up here), a Language tab
(real LLM-based text continuation via the Claude API), a Perception tab
(real camera capture sampled every few seconds, analyzed by a vision model
in a continuous agentic loop, each frame tagged by scene category and
checked against how many times that category has recurred — flagging
"recognized" repeats the way a paramedic notices a presentation they've
seen before — and each frame also scored for priority, so the single
most attention-worthy detail surfaces first, the way one blip pulls
focus on a busy radar screen; this is a general-purpose salience score
for an ordinary camera feed, not radar signal processing or military
target classification), an Evolve tab (reads accumulated
cross-channel history and asks Claude to propose ranked upgrade
suggestions), a Phone Sensors tab (real accelerometer motion feed and a
camera-based approximate heart-rate reading), and an Architecture tab
documenting the shared pipeline.

**Note on brain signals:** a phone has no EEG or fMRI hardware, so nothing
in this app generates or claims to generate genuine brain-signal data.
The Phone Sensors tab uses what a phone actually has — motion and an
approximate camera heart-rate estimate — clearly labeled as neither
medical-grade nor brain activity. The Signal tab's waveform remains a
labeled simulation. Real EEG would require actual hardware (e.g. a Muse
or OpenBCI headset) connected over Web Bluetooth.

**Note on "quantum mainframe":** quantum computing hardware isn't
something a software build can provide — it's physical infrastructure, not
code, and nothing here claims to use it. The Perception tab is the real,
working equivalent of "continuously read the surroundings and reason about
them": ordinary compute, a camera, and a vision-language model in a loop.

**Note on "self-evolving":** it doesn't rewrite its own code unsupervised
— unreviewed changes shipping with no one checking them isn't something to
build without oversight. What it does do: every prediction across all
three channels is persisted, the Evolve tab reads that real history to
generate grounded suggestions instead of generic ones, and "Apply best"
queues the top suggestion as the next thing to actually build — same
review step as any other change to this repo.

## Why two channels, one app

"AGI brainpredict" reads as two different products depending on which word
you emphasize — a brain-signal predictor, or a general prediction engine.
Rather than pick one, this scaffold treats them as two implementations of
the same interface, so a real signal-ingestion module can be dropped in
later without touching the rest of the app.

## Folder structure

```
brainpredict/
├── apps/
│   └── web/                     # React front end (Vite)
│       ├── src/
│       │   ├── App.jsx          # shell + tab routing (see brainpredict-app.jsx)
│       │   ├── panels/
│       │   │   ├── SignalPanel.jsx
│       │   │   ├── LanguagePanel.jsx
│       │   │   └── ArchitecturePanel.jsx
│       │   ├── lib/
│       │   │   ├── claudeClient.js     # wraps calls to /v1/messages
│       │   │   └── signalModel.js      # extrapolation / real model hook
│       │   └── main.jsx
│       ├── index.html
│       ├── package.json
│       └── vite.config.js
│
├── mobile/
│   └── capacitor.config.ts       # wraps apps/web for Android/iOS export
│
├── server/                        # only needed once a real backend replaces localStorage
│   ├── src/
│   │   ├── routes/
│   │   │   ├── predict-text.ts    # proxies to Claude API, keeps the key server-side
│   │   │   └── predict-signal.ts  # would call a real signal-prediction model
│   │   └── index.ts
│   └── package.json
│
├── .env.example
└── README.md
```

## Swapping in a real signal source

The Signal panel's `useNeuralSeries` and `extrapolate` functions are the
two seams to replace:

1. **Ingest** — replace the synthetic sine generator with a WebSocket or
   polling connection to actual hardware (a BCI headset, an EEG amplifier's
   SDK, or a stored `.edf`/`.fif` recording parsed on the server).
2. **Model** — replace `extrapolate()` (a linear + harmonic heuristic) with
   a real trained model — e.g. a small transformer or LSTM trained on
   labeled signal windows, served from `server/src/routes/predict-signal.ts`.

Everything downstream (charting, tab layout, prediction/observed styling)
stays the same, because both are just arrays of `{ i, v, kind }` points.

## Language channel

`LanguagePanel.jsx` calls the Claude API directly from the client for this
demo. For production, move the `fetch` call into
`server/src/routes/predict-text.ts` so the API key never reaches the
browser, and have the client call your own `/api/predict-text` instead.

## Mobile export

Per your usual stack: once `apps/web` is stable, `mobile/capacitor.config.ts`
wraps it for Capacitor-based Android/iOS builds — same approach as
ComplaintBox.

## Stack

React (JSX), Recharts for the signal chart, Claude API for language
prediction, Capacitor for mobile export — consistent with your existing
toolchain.
# RMG