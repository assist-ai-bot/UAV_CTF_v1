# Exercise Kestrel — UAV track

A lab survey drone flew a test sortie. A passive receiver recorded 150 seconds of the
915 MHz band around it. Afterwards the lab noticed something wrong with the flight and
suspects two things: that a command reached the aircraft from someone who was not the real
ground station, and that its satellite navigation was interfered with.

Neither suspicion has been confirmed. That is your job.

**Everything you need is one recording.** No radio hardware. No internet lookups against
live systems. Nothing here asks you to transmit, and nothing in the exercise requires it.

## Get the files

The recording is 300 MB, so it is attached to the
[latest release](../../releases/latest) rather than committed here.

| File | Where | What |
| --- | --- | --- |
| `capture.cs8` | [Releases](../../releases/latest) | The recording. 8-bit signed I/Q, interleaved, 300,000,000 bytes |
| `capture.meta` | this repo | Recorder metadata from the operator. **Unverified** |
| `briefing/` | this repo | The full briefing. Read it first |

Check your download before you start:

```bash
shasum -a 256 capture.cs8     # macOS
sha256sum capture.cs8         # Linux
```

It must be exactly `661f854868cd2ed88ef42f52481295a7205eaf20a850d2dd4e6cbb5a92397387`
and exactly 300,000,000 bytes. If it is not, the download is damaged — say so before you
spend a session on it.

## Read this first

**`briefing/KESTREL-7_briefing.pdf`** is the real instruction set. It has, for every one of
the six stages: what you must do, the exact answer format, and a worked example of that
format. It also lists the facts the lab can vouch for and a starter snippet that opens the
file correctly.

This README is only the orientation. The briefing is the exercise.

## Set up

Python 3.10 or newer, about 1.5 GB of free RAM.

```bash
python3 -m venv .venv
.venv/bin/pip install numpy scipy matplotlib pymavlink
```

The briefing lists other tools you may find useful — `inspectrum` is worth having. None of
them are mandatory and everything can be done from Python.

## The first thing to get right

```python
import numpy as np
x = np.memmap('capture.cs8', dtype=np.int8).reshape(-1, 2)   # 150 million I,Q pairs
dc = x[:2_000_000].astype(np.float32).mean(0)                # the DC offset is significant
seg = x[:5_000_000].astype(np.float32)                       # work in slices, never the whole file
iq  = (seg[:, 0] - dc[0]) + 1j * (seg[:, 1] - dc[1])
```

Do not convert the whole file to complex numbers at once — that is several gigabytes and
your machine will stall. Subtract the DC offset before you measure anything.

## The six stages

Fourteen answers in total. They run in order, because each stage needs the output of the
one before it. Stage S1 is a gate: if your demodulator is not exactly right, nothing after
it will be either.

| Stage | What you are doing | Answers |
| --- | --- | --- |
| S0 | Survey the band. Establish what the recorder really did and who is transmitting | 1 |
| S1 | Get bytes off the downlink | 2 |
| S2 | Read the telemetry, and find three messages hidden inside it | 3 |
| S3 | Find two messages that are not carried by any message field | 2 |
| S4 | Audit the uplink: who sent what, and what the aircraft did about it | 3 |
| S5 | Decide whether satellite navigation was attacked, and characterise it | 3 |

The exact task list and answer format for each stage is in the briefing.

## Submitting

Every answer is a `KST{...}` string, submitted on the Hack-Intern portal under the
**UAV Track**. Braces matter, case matters, and there are no spaces anywhere. Each
challenge allows a limited number of attempts, so check an answer before you spend one.

Some strings in this recording look like valid answers and are not. They are there
deliberately. Anything that fails an integrity check — at any layer — is not evidence.

## A note on what this is

Everything in the recording is synthetic and was generated for this exercise. No real
aircraft, operator or network is involved. The scenario is lab-scaled: the navigation event
unfolds considerably faster than a real one would, so that it fits inside 150 seconds.

Work on your own machine. Do not share the files, your answers or your approach.
