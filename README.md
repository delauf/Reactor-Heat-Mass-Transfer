# Reactor-Heat-Mass-Transfer
Simulating a tubular reactor - a long pipe where a liquid (or gas) carrying reactant A flows in one end, a chemical reaction happens as it flows through, and products come out the other end. The pipe is surrounded by a cooling jacket (like a shell-and-tube heat exchanger) to remove heat, because the reaction is exothermic

https://colab.research.google.com/github/delauf/Reactor-Heat-Mass-Transfer/blob/main/Heat_Mass_Transfer_Reactor_Advanced.ipynb

This project models the steady-state behavior of an exothermic first-order reaction (A → B) inside a jacketed tubular reactor. The simulation solves coupled partial differential equations for temperature and concentration in 2D (axial + radial) using finite difference methods, capturing phenomena that 1D plug-flow models miss — particularly radial temperature gradients and hot spot formation.
Governing Equations
Species balance (reactant A):
u∂CA∂z=Deff(∂2CA∂r2+1r∂CA∂r+∂2CA∂z2)−k0 e−Ea/RTCAu \frac{\partial C_A}{\partial z} = D_{eff} \left( \frac{\partial^2 C_A}{\partial r^2} + \frac{1}{r}\frac{\partial C_A}{\partial r} + \frac{\partial^2 C_A}{\partial z^2} \right) - k_0 \, e^{-E_a/RT} C_Au∂z∂CA​​=Deff​(∂r2∂2CA​​+r1​∂r∂CA​​+∂z2∂2CA​​)−k0​e−Ea​/RTCA​
Energy balance:
uρcp∂T∂z=kth(∂2T∂r2+1r∂T∂r+∂2T∂z2)+(−ΔHr) k0 e−Ea/RTCAu \rho c_p \frac{\partial T}{\partial z} = k_{th} \left( \frac{\partial^2 T}{\partial r^2} + \frac{1}{r}\frac{\partial T}{\partial r} + \frac{\partial^2 T}{\partial z^2} \right) + (-\Delta H_r) \, k_0 \, e^{-E_a/RT} C_Auρcp​∂z∂T​=kth​(∂r2∂2T​+r1​∂r∂T​+∂z2∂2T​)+(−ΔHr​)k0​e−Ea​/RTCA​
Boundary Conditions
BoundaryTemperatureConcentrationInlet (z = 0)T = T_inlet (Danckwerts)C_A = C_A,inletCenterline (r = 0)∂T/∂r = 0 (symmetry)∂C/∂r = 0 (symmetry)Wall (r = R)-k_th ∂T/∂r = h(T - T_cool)∂C/∂r = 0 (no flux)Outlet (z = L)∂T/∂z = 0 (Neumann)∂C/∂z = 0 (Neumann)

Features
Core Simulation

2D finite difference PDE solver with Gauss-Seidel iteration and successive over-relaxation (SOR)
Fully vectorized NumPy implementation — no Python-level loops over grid points
Coupled physics: temperature field drives Arrhenius kinetics, reaction heat feeds back into temperature
Upwind differencing for convection, central differencing for diffusion, cylindrical coordinate corrections

Analysis Tools

Hot spot detection — automatically locates peak temperature and flags thermal runaway risk
Parametric sweep — sweeps coolant temperature to map the thermal runaway boundary using the sensitivity metric dT_peak/dT_cool
Velocity sensitivity analysis — explores the residence time vs. conversion trade-off, tied to Damköhler number
Global energy balance validation — checks that reaction heat = wall cooling + convective transport
Dimensionless analysis — computes Da, Pe (mass & heat), Bi, and adiabatic temperature rise

Visualization

6-panel diagnostic figure: 2D temperature field, conversion field, axial profiles, radial profiles, convergence history
Reaction rate and Arrhenius rate constant field maps
Parametric sensitivity curves with automatic runaway onset detection

Interactive Dashboard

ipywidgets sliders for coolant temperature, flow velocity, inlet temperature, and inlet concentration
Re-solves the full 2D problem on button click with updated visualizations


Dimensionless Numbers
The simulation computes and reports key dimensionless groups that characterize reactor behavior:
NumberDefinitionPhysical MeaningDamköhler (Da)k(T₀)·L / uReaction rate vs. convection rate. Da >> 1 → high conversionPeclet, mass (Pe_m)u·L / D_effConvective vs. diffusive mass transportPeclet, heat (Pe_h)u·L·ρcₚ / k_thConvective vs. conductive heat transportBiot (Bi)h·R / k_thWall transfer resistance vs. internal conduction resistanceβ (adiabatic rise)(-ΔH)·C₀ / (ρcₚT₀)Max possible temperature rise if no heat removal

Numerical Method
The steady-state coupled PDEs are solved iteratively using Gauss-Seidel relaxation with SOR (ω = 1.4). Key implementation details:

Convection: First-order upwind differencing for numerical stability
Diffusion: Second-order central differencing with 1/r cylindrical correction term
Coupling: Fully implicit — both T and C fields are updated each iteration using the latest values of the other
Convergence: Monitored via max relative change in T and C fields; default tolerance = 1×10⁻⁶
Stability: Damping factor (0.25) applied to updates to prevent oscillation in strongly coupled regimes
Grid: 30 radial × 120 axial points (configurable)


Default Parameters
ParameterValueDescriptionR_tube2.5 cmTube radiusL_tube150 cmTube lengthu0.3 m/sInlet velocityT_inlet600 KFeed temperatureT_coolant590 KCoolant jacket temperatureC_A,02000 mol/m³Inlet concentrationk₀2.5 × 10⁵ s⁻¹Pre-exponential factorEₐ50 kJ/molActivation energyΔH_rxn-45 kJ/molHeat of reaction (exothermic)k_th0.8 W/(m·K)Thermal conductivityD_eff1 × 10⁻⁵ m²/sEffective diffusivityh_wall200 W/(m²·K)Wall heat transfer coefficient
