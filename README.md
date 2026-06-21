# Advanced Jeffcott Rotor Simulator

**Author:** Akshat Verma
**Stack:** Python · NumPy · SciPy · PyWavelets · Matplotlib

---

## What is the Jeffcott Rotor?

The **Jeffcott rotor** (derived by H.H. Jeffcott in 1919) is the canonical model of a flexible rotating shaft. It consists of:

- A **massless elastic shaft** of stiffness $k$ (N/m), simply supported at both ends
- A **rigid disk** of mass $m$ (kg) mounted at the shaft midspan
- **Viscous damping** $c$ (N·s/m) representing bearing dissipation
- A **mass eccentricity** $e$ (m) — the offset of the centre of mass from the geometric centre

Despite its simplicity, the Jeffcott rotor captures the three most important phenomena in rotating machinery dynamics:

1. **Critical speed resonance** — the operating speed at which shaft flexibility causes catastrophic vibration amplification
2. **Self-centring above critical speed** — the counter-intuitive reduction in vibration at supercritical speeds
3. **Phase reversal at resonance** — the 90° → 180° phase shift that is the basis of industrial balancing procedures

---

## Equations of Motion

The governing equations in the inertial $(x, y)$ frame:

$$m\ddot{x} + c\dot{x} + kx = me\Omega^2\cos(\Omega t)$$

$$m\ddot{y} + c\dot{y} + ky = me\Omega^2\sin(\Omega t)$$

The right-hand side is the **centrifugal unbalance force** rotating at shaft speed $\Omega$. At resonance ($\Omega = \omega_n = \sqrt{k/m}$), the steady-state amplitude is:

$$X_{resonance} = \frac{e}{2\zeta}$$

where $\zeta = c / (2\sqrt{km})$ is the damping ratio. See the notebook for the complete derivation from first principles.

---

## Fault Configurations

The simulator implements four configurations, switchable via the `CONFIG` dictionary:

### 1. Healthy Rotor (`fault='healthy'`)
Small residual eccentricity only. Produces a **pure 1× spectrum** and a **circular orbit**. Used as the diagnostic baseline.

### 2. Mass Unbalance (`fault='unbalance'`)
Elevated eccentricity $e_u$ at a specified phase angle $\phi$. Produces an **elevated 1× component** with no other spectral changes — the orbit remains circular but larger.

**Key diagnostic:** 1× amplitude is elevated; 2× and higher are at the noise floor; orbit is round.

### 3. Breathing Crack — Mayes-Davies Model (`fault='crack'`)
A transverse shaft crack opens and closes once per revolution under gravity bending. Modelled as:

$$k(\theta) = k_0\left(1 - \mu\cos\theta\right), \quad \theta = \Omega t$$

where $\mu \in [0,1]$ is the crack depth ratio (crack depth / shaft radius). This **parametric stiffness variation** generates super-harmonic response at 2×, even under purely 1× excitation.

**Key diagnostic:** Growing 2× component in the spectrum; inner loop in the orbit plot. 2×/1× amplitude ratio increases monotonically with crack depth $\mu$.

### 4. Bearing Clearance Nonlinearity (`fault='bearing'`)
When shaft displacement exceeds radial bearing clearance $\delta$, the restoring force stiffens abruptly (journal contacts bearing shell):

$$F_{bearing}(x) = \begin{cases} -k_0 x & |x| \leq \delta \\ -k_0 x - k_c(|x| - \delta)\,\text{sgn}(x) & |x| > \delta \end{cases}$$

This nonlinear switching generates **super-harmonics** (2×, 3×, ...) and potentially sub-harmonics (½×) depending on the severity.

**Key diagnostic:** Multiple harmonics (2×, 3×) in FFT; elevated kurtosis (> 4.0); clipped or flattened orbit.

---

## Signal Processing Outputs

For each fault configuration, the notebook produces:

| Output | Tool | What it shows |
|---|---|---|
| Time domain $x(t)$, $y(t)$ | Direct integration | Overall vibration level and waveform shape |
| Orbit plot | Parametric $x$ vs $y$ | Shaft centreline trajectory — shape is fault-specific |
| FFT order spectrum | `np.fft.rfft` + Hann window | Frequency content normalised to running speed |
| STFT spectrogram | `scipy.signal.stft` | Time-varying frequency content |
| CWT scalogram | `pywt.cwt` (Morlet) | Multi-resolution time-frequency decomposition |
| Statistical features | RMS, kurtosis, crest factor | Scalar fault indicators for classification |
| Campbell diagram | Speed sweep | Amplitude vs speed — identifies critical speed |

---
### Requirements

```
numpy
scipy
matplotlib
pywt          # pip install PyWavelets
pandas
```

Install all at once:
```bash
pip install numpy scipy matplotlib PyWavelets pandas jupyter
```

### Running the Notebook

```bash
git clone https://github.com/akshatverma1602/Advanced-Jeffcott-Rotor-Dynamics-Simulator.git
jupyter notebook jeffcott_rotor_simulator.ipynb
```
### Changing Parameters

**All parameters are in the `CONFIG` dictionary at the top of Section 1.

---

## Physical Context

In industrial practice, the Jeffcott rotor is the conceptual backbone of every rotating machinery vibration analysis. When a vibration analyst looks at data from a centrifugal pump, compressor, or steam turbine, they are mentally applying the Jeffcott model to interpret what they see:

- **1× dominant spectrum + round orbit** → unbalance → ISO 1940 balancing procedure
- **Elevated 2× appearing** → crack or misalignment → shut down and inspect
- **Multiple harmonics + high kurtosis** → bearing clearance or rub → expedited maintenance
- **Sub-synchronous component** → oil whirl/whip → bearing redesign or speed change

This simulator generates vibration data that reproduces these exact signatures, providing a controlled environment to study fault evolution and develop/validate diagnostic algorithms.

---

## References

1. Jeffcott, H.H. (1919). *The lateral vibration of loaded shafts in the neighbourhood of a whirling speed.* Philosophical Magazine, Series 6, 37(219), 304–314.
2. Mayes, I.W. & Davies, W.G.R. (1984). *Analysis of the response of a multi-rotor-bearing system containing a transverse crack in a rotor.* Journal of Vibration, Acoustics, Stress, and Reliability in Design, 106(1), 139–145.
3. Vance, J.M., Zeidan, F. & Murphy, B. (2010). *Machinery Vibration and Rotordynamics.* Wiley.
4. Rao, J.S. (1996). *Rotor Dynamics*, 3rd ed. New Age International.
5. ISO 1940-1:2003 — *Mechanical vibration — Balance quality requirements for rotors in a constant (rigid) state.*
6. API Standard 610, 12th ed. — *Centrifugal Pumps for Petroleum, Petrochemical and Natural Gas Industries.*

---
