# Reactor-Heat-Mass-Transfer
Simulating a tubular reactor - a long pipe where a liquid (or gas) carrying reactant A flows in one end, a chemical reaction happens as it flows through, and products come out the other end. The pipe is surrounded by a cooling jacket (like a shell-and-tube heat exchanger) to remove heat, because the reaction is exothermic

https://colab.research.google.com/github/delauf/Reactor-Heat-Mass-Transfer/blob/main/Heat_Mass_Transfer_Reactor_Advanced.ipynb

## Overview

This project models the steady-state behavior of an exothermic first-order reaction (A → B) inside a jacketed tubular reactor. The simulation solves coupled partial differential equations for temperature and concentration in 2D (axial + radial) using finite difference methods, capturing phenomena that 1D plug-flow models miss — particularly radial temperature gradients and hot spot formation.

### Governing Equations

**Species balance (reactant A):**

$$u \frac{\partial C_A}{\partial z} = D_{eff} \left( \frac{\partial^2 C_A}{\partial r^2} + \frac{1}{r}\frac{\partial C_A}{\partial r} + \frac{\partial^2 C_A}{\partial z^2} \right) - k_0 \, e^{-E_a/RT} C_A$$

**Energy balance:**

$$u \rho c_p \frac{\partial T}{\partial z} = k_{th} \left( \frac{\partial^2 T}{\partial r^2} + \frac{1}{r}\frac{\partial T}{\partial r} + \frac{\partial^2 T}{\partial z^2} \right) + (-\Delta H_r) \, k_0 \, e^{-E_a/RT} C_A$$

### Boundary Conditions

| Boundary | Temperature | Concentration |
|----------|------------|---------------|
| **Inlet** (z = 0) | T = T_inlet (Danckwerts) | C_A = C_A,inlet |
| **Centerline** (r = 0) | ∂T/∂r = 0 (symmetry) | ∂C/∂r = 0 (symmetry) |
| **Wall** (r = R) | -k_th ∂T/∂r = h(T - T_cool) | ∂C/∂r = 0 (no flux) |
| **Outlet** (z = L) | ∂T/∂z = 0 (Neumann) | ∂C/∂z = 0 (Neumann) |

---

## Features

### Core Simulation
- **2D finite difference PDE solver** with Gauss-Seidel iteration and successive over-relaxation (SOR)
- **Fully vectorized** NumPy implementation — no Python-level loops over grid points
- **Coupled physics**: temperature field drives Arrhenius kinetics, reaction heat feeds back into temperature
- **Upwind differencing** for convection, central differencing for diffusion, cylindrical coordinate corrections

### Analysis Tools
- **Hot spot detection** — automatically locates peak temperature and flags thermal runaway risk
- **Parametric sweep** — sweeps coolant temperature to map the thermal runaway boundary using the sensitivity metric dT_peak/dT_cool
- **Velocity sensitivity analysis** — explores the residence time vs. conversion trade-off, tied to Damköhler number
- **Global energy balance validation** — checks that reaction heat = wall cooling + convective transport
- **Dimensionless analysis** — computes Da, Pe (mass & heat), Bi, and adiabatic temperature rise

### Visualization
- 6-panel diagnostic figure: 2D temperature field, conversion field, axial profiles, radial profiles, convergence history
- Reaction rate and Arrhenius rate constant field maps
- Parametric sensitivity curves with automatic runaway onset detection

### Interactive Dashboard
- `ipywidgets` sliders for coolant temperature, flow velocity, inlet temperature, and inlet concentration
- Re-solves the full 2D problem on button click with updated visualizations

---

## Dimensionless Numbers

The simulation computes and reports key dimensionless groups that characterize reactor behavior:

| Number | Definition | Physical Meaning |
|--------|-----------|-----------------|
| **Damköhler (Da)** | k(T₀)·L / u | Reaction rate vs. convection rate. Da >> 1 → high conversion |
| **Peclet, mass (Pe_m)** | u·L / D_eff | Convective vs. diffusive mass transport |
| **Peclet, heat (Pe_h)** | u·L·ρcₚ / k_th | Convective vs. conductive heat transport |
| **Biot (Bi)** | h·R / k_th | Wall transfer resistance vs. internal conduction resistance |
| **β (adiabatic rise)** | (-ΔH)·C₀ / (ρcₚT₀) | Max possible temperature rise if no heat removal |

---

## Numerical Method

The steady-state coupled PDEs are solved iteratively using **Gauss-Seidel relaxation with SOR** (ω = 1.4). Key implementation details:

- **Convection**: First-order upwind differencing for numerical stability
- **Diffusion**: Second-order central differencing with 1/r cylindrical correction term
- **Coupling**: Fully implicit — both T and C fields are updated each iteration using the latest values of the other
- **Convergence**: Monitored via max relative change in T and C fields; default tolerance = 1×10⁻⁶
- **Stability**: Damping factor (0.25) applied to updates to prevent oscillation in strongly coupled regimes
- **Grid**: 30 radial × 120 axial points (configurable)

---

## Default Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| R_tube | 2.5 cm | Tube radius |
| L_tube | 150 cm | Tube length |
| u | 0.3 m/s | Inlet velocity |
| T_inlet | 600 K | Feed temperature |
| T_coolant | 590 K | Coolant jacket temperature |
| C_A,0 | 2000 mol/m³ | Inlet concentration |
| k₀ | 2.5 × 10⁵ s⁻¹ | Pre-exponential factor |
| Eₐ | 50 kJ/mol | Activation energy |
| ΔH_rxn | -45 kJ/mol | Heat of reaction (exothermic) |
| k_th | 0.8 W/(m·K) | Thermal conductivity |
| D_eff | 1 × 10⁻⁵ m²/s | Effective diffusivity |
| h_wall | 200 W/(m²·K) | Wall heat transfer coefficient |

---
