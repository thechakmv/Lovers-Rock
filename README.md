# Dual-Stage Overdrive & Distortion Guitar Pedal

I created analog guitar effects pedal with two independent, switchable gain stageto achieve classic Thrash metal rhythm and lead tones: a soft-clipping overdrive with a **mid-boost** (+6 dB at 800 Hz) and a hard-clipping distortion with a **mid-scoop** (−6 to −10 dB across 400–800 Hz). The Pedal features a single-knob volume control, and gain controls for each stage as well. Each stage has true-bypass switching, so the two can run independently or be cascaded for stacked high-gain lead tones.

## Highlights

- Designed and simulated in LTSpice
- Two-stage analog signal chain built around TL072 op-amps
- True-bypass per stage via 3PDT footswitches, both stages can be activated simultaneously for cascaded gain
- Gyrator filter network for scooped voicings on distortion
- Single 9 V DC supply, ~15 mA total current draw
- PCB layout in KiCad with attention to signal integrity, and inter-stage isolation

## Signal Flow

```
                                                         
 Guitar ─> Input ─┬─> OD Stage ──┬─> Distortion ───> Tone ──┬> Volume ─> Output ─> Amp
          Buffer  │   (soft clip,│   (hard clip + (LPF)     │
          (TL072) │   mid-boost) │   Twin-T scoop)          │
                  │              │                          │
                  ▼              ▼                          ▼
                SW1 3PDT        SW2 3PDT                    Always
                true bypass     true bypass                 on signal path
```

When both switches are engaged, the OD output feeds the distortion stage input, producing a mid boosted overdrive. Each stage can also run alone for classic rock overdrive or for chuggy thrash metal rhythm tones.

## Design Specifications

| Parameter | Value |
|---|---|
| Supply | 9 V DC, center-negative, ~30 mA |
| OD Specs | peak at ~800Hz |
| OD clipping | Soft, symmetric, ±0.6 V (1N4148 anti-parallel in feedback) |
| Distortion Specs | Notch at around 800Hz |
| Distortion clipping | Hard, symmetric, ±0.6 V (1N4148 to Vref) |

## Design Philosophy

Thrash metal dictates that you have wo pedals. One Distortion, and one overdrive. For my pedal I wanted to combine these two topologies into one pedal. This design uses two intentionally distinct gain stages whose characters complement each other when cascaded overdrive to distortion.

The **overdrive stage** is a Tube Screamer-derived non-inverting topology with silicon diodes inside the feedback loop. The feedback RC network forms a band-pass response: lows roll off below 720 Hz and highs roll off above ~6 kHz at maximum gain. Inside the band the gradual diode conduction produces soft clipping that preserves the dry signal underneath the saturation, giving the characteristic transparent overdrive sound.

The **distortion stage** uses an inverting topology with diodes clamping the op-amp output directly to the Vref node. This produces hard, square-wave clipping rich in odd harmonics. The clipped signal then feeds a dual gyrator bandpass filter with peaks at 150 Hz, and 3.3KHz, carving out the middle frequencies by boosting the low and high frquencies.

The two stages are independently switchable: OD alone for classic rock, distortion alone for chunky rhythm, and both stacked for full-spectrum lead tones.

## Repository Structure

```
├── ltspice/
│   ├── Draft2.asc         
├── kicad/
│   ├── Lovers Rock.kicad_pro             Project file
│   ├── Lovers Rock.kicad_sch             Schematic
│   ├── Lovers Rock.kicad_pcb             Board layout
│   └── gerbers/                    Fabrication output (zipped Gerbers + drill)
└── images/
    ├──        
```

## Tools

**Software:** LTSpice, KiCad 10

**Test equipment:** multimeter, soldering iron, electrical components (caps, resistors etc.)

## Build Process

1. **Simulate.** Run AC and transient analyses in LTSpice for each stage. Verify gain curves, clipping shape, and Vref stability under load. Use `.step` to sweep gain pot values and see the response variation across the control range.

2. **Prototype.** Build stage-by-stage with bench power. tune by ear — swap diodes (Si vs. LED vs. asymmetric), tweak the frequencies, and adjust the gain ranges.

3. **PCB.** Re-capture the validated schematic in KiCad, assign through-hole footprints, lay out the board with the external component sockets neat their enclosure mounts. Generate Gerbers and fabricate via JLCPCB.


**TBD:**

1. **Verify.** Measure assembled pedal's frequency response and FFT using spectrul analyzer to compare against simulation predictions.
2. Order the PCBS, and prepare for final assembly



## Design Issues Encountered (and Fixed)

Some that problems came up during simulation and breadboarding that are worth documenting:

**Ridiculous power supply noise.** The power supply was being unrully, and I added bypass capacitors to each power trace to help mitigate the noise I was experiencing.

**Vref instability under heavy distortion.** The hard-clipping diodes were pumping current pulses into the Vref node, dragging it from 4.5 V down toward 1 V during the clipping phase. I Fixed it by adding a 1 kΩ series resistor between the distortion op-amp's output and the diode pair, limiting peak diode current and preserving Vref node.

**High-frequency oscillation at high gain.** The TL072 stages were prone to oscillation when driving purely capacitive loads, so I fixed it by adding a 470 Ω series isolation resistor between each op-amp output and its capacitive load.


## Skills Demonstrated

- Analog circuit design with op-amps
- Frequency-response shaping via active gyrator filter networks
- Soft-clipping vs. hard-clipping topology selection and harmonic analysis
- Single-supply audio biasing: Vref generation, AC coupling, bypass strategy
- SPICE simulation: AC sweep, transient analysis, FFT, parametric stepping
- PCB layout for analog audio: ground topology, decoupling, inter-stage isolation

