# ⚠️ FOURIER CONVENTION: CONJUGATE GLoW's F BEFORE MULTIPLYING A LAL / bilby WAVEFORM

```python
h_lensed = np.conj(F_glow(w)) * h_lal          # w = 8 pi G M_Lz f / c^3
```

**If you multiply a LAL/bilby/pycbc/gwsignal/dingo waveform by an F(w) from GLoW
without `np.conj`, the lensing is time-reversed.** The delayed images arrive
*before* the main one and their Morse phases flip. The result is still a
smooth, plausible-looking F(f) with the same |F|, so nothing crashes and
amplitude checks pass. The parameter estimation is simply wrong.

## The two conventions

| | lensing / GLoW | LAL, bilby, pycbc, gwsignal, dingo |
|---|---|---|
| Fourier transform | h(f) = ∫ h(t) e^{**+**2πift} dt | h(f) = ∫ h(t) e^{**−**2πift} dt |
| image delayed by T | e^{**+**iwT} | e^{**−**2πifT} |
| point lens, GO | √μ₊ − i √\|μ₋\| e^{**+**iwT} | √μ₊ **+** i √\|μ₋\| e^{**−**2πifT} |

GLoW defines F(w) = (w/2πi) ∫ d²x exp[**+**i w φ(x, y)], so in geometric
optics F = Σ_j √|μ_j| exp(**+**i w T_j − iπ n_j). The lensing is a real filter
in the time domain, so going from one column to the other is exactly a complex
conjugation.

## What is in GLoW's convention (→ needs `np.conj` at the waveform)

- GLoW v1 (`glow`, `GLoW_public`), GLoW2 (`glow2`, `glow_v2`, `GLoW2_prerelease`):
  every `Fw(...)`, `eval_Fw`, `Fw_PL`, `Fw_FFT*`, `Fw_MixedFT`, `Fw_DirectFT`, ….
- Anything trained on, fitted to, or interpolated from the above.


## Rules

1. **Conjugate once, at the point where F multiplies the waveform.** Keep
   templates, bases and fits in GLoW's convention; don't conjugate halfway
   through a pipeline.
2. **Conjugate the whole factor.** For a basis expansion with real
   coefficients, conj(1 + Σ α_k ψ_k) = 1 + Σ α_k conj(ψ_k). This changes the
   model and can't be absorbed by α. Real-coefficient bases are in general *not* closed under conjugation.
3. **These checks are blind to a missing conjugation:** the no-lens limit
   F → 1, anything using only |F|, GLoW-vs-GLoW comparisons, and
   mismatches maximised over a symmetric time shift.
4. **Validate every new lensed source model on a *lensed* reference.**
   - The time-domain test below.
   - **and** reproduce the stored lnL of a published lensed posterior sample.

## 10-second test (the delayed image must come out at t = +T)

```python
import numpy as np
from glow2 import freq_domain
y, MLz, fs, Tseg = 2.0, 1000.0, 4096, 8.0
f = np.fft.rfftfreq(int(fs*Tseg), 1/fs); w = 1.23791089411462657335e-4*MLz*f
F = np.ones(f.size, complex); F[1:] = freq_domain.Fw_PL(y)(w[1:])
G = np.conj(F)                         # <- the factor you apply to LAL waveforms
h = np.fft.irfft(G*np.exp(-(f/400)**2))   # numpy irfft = inverse of LAL's e^{-2 pi i f t}
t = np.arange(h.size)/fs; t[t > Tseg/2] -= Tseg; m = abs(t) > 0.01
print(1e3*t[m][np.argmax(abs(h[m]))])  # +89.6 ms (GO delay 90.5 ms); -89.6 without conj
```

## In this repository

- `FwGeneral` / `FwGeneral_C` docstrings state the definition. The sign is implemented in `wrapper/glow_lib/source/fourier_lib.c`.
- `waveform.py` already applies the conjugation when lensing LAL-convention waveforms: `Fws = np.abs(Fws)*np.exp(-1j*np.angle(Fws))`. Use it, or `np.conj`, in any external PE code.
- The 10-second test uses the GLoW2 API. With this package, compute F with `freq_domain_c.Fw_AnalyticPointLens_C(y).eval_Fw(w)` instead.
