# PV Integration in a Distribution Network

## Voltage Impacts, Reverse Power Flow, and Smart-Inverter Mitigation

This project uses the IEEE 33-bus radial distribution test system in `pandapower`[1] to study how increasing solar PV penetration affects distribution-network operation.

The analysis focuses on:

- feeder voltage profiles
- active-power losses
- the effect of PV location
- reverse active-power flow at high PV penetration
- reactive-power control using an illustrative Volt-VAR controller

The study uses steady-state AC power flow. PV penetration is calculated as installed PV active-power capacity divided by the feeder's total active load.

---

## Objectives

The project was developed to understand several practical issues associated with integrating distributed solar PV into a distribution feeder:

1. How does increasing PV penetration affect feeder voltage?
2. How does PV affect active-power losses?
3. Does the location of a PV plant affect its impact on the network?
4. What happens when PV generation becomes larger than the local load?
5. Can reactive-power control reduce voltage rise caused by high PV penetration?
6. How does the required reactive power change as PV penetration increases?

---

## Network Model

The analysis uses the IEEE 33-bus distribution test system available in `pandapower`.

The feeder has:

- 33 buses
- 32 active distribution lines
- 32 loads
- 1 external grid
- 3.715 MW total active load
- 2.300 MVAr total reactive load

The base-case AC power flow gives:

| Quantity | Base case |
|---|---:|
| Minimum voltage | 0.9131 p.u. |
| Maximum voltage | 1.0000 p.u. |
| Active-power losses | 0.2027 MW |
| Weakest bus | Bus 17 |

The low starting voltage is important when interpreting the PV results.

---

## Methodology

### 1. Increasing PV penetration

PV was initially connected at Bus 17 with `Q = 0 MVAr` so that the effect of active-power injection could be examined independently of reactive-power control.

PV capacity was increased over several operating points.

PV penetration was calculated as:

$$\
PV\ penetration =
\frac{PV\ capacity}{3.715\ MW}\times100
\$$

Increasing PV generally improved the minimum feeder voltage and initially reduced active-power losses.

For example, at 0.40 MW of PV at Bus 17:

- minimum voltage increased from 0.9131 to 0.9230 p.u.
- losses decreased from 0.2027 MW to 0.1596 MW

The reduction in losses did not continue indefinitely. In the tested cases, losses reached their lowest value around 0.8 MW before increasing again at higher PV output.

---

### 2. Effect of PV location

A fixed 0.40 MW PV plant was tested at several buses:

- Bus 3
- Bus 6
- Bus 10
- Bus 14
- Bus 17
- Bus 24
- Bus 32

The results showed that the same PV capacity can produce different voltage and loss results depending on its electrical location.

Among the tested locations, Bus 14 produced:

- minimum voltage: approximately 0.9230 p.u.
- losses: approximately 0.1587 MW

Bus 17 produced a very similar minimum voltage but slightly higher losses.

This experiment demonstrates that PV placement matters when integrating distributed generation into a distribution feeder.

---

### 3. PV penetration at Bus 14 and Bus 17

PV capacities from 0 to 1.5 MW were compared at Bus 14 and Bus 17.

Bus 14 generally produced slightly lower losses and slightly higher minimum voltage across the tested range.

Bus 17 was retained as a high-PV stress case because it produced a clear local voltage rise at high PV penetration.

At 1.5 MW:

| Case | Minimum V | Maximum V | Losses |
|---|---:|---:|---:|
| Bus 14 | 0.9386 p.u. | 0.9975 p.u. | 0.1408 MW |
| Bus 17 | 0.9379 p.u. | 1.0163 p.u. | 0.1720 MW |

---

## Reverse Power Flow

The 1.5 MW PV case at Bus 17 was examined in more detail.

At this operating point, PV capacity is approximately 40.38% of the feeder's total active load.

The local load at Bus 17 is only 0.09 MW. Therefore, the 1.5 MW PV plant supplies the local load and exports the remaining power upstream.

For example, on the line between Bus 16 and Bus 17:

- power at the Bus 16 side: approximately -1.401 MW
- power at the Bus 17 side: approximately +1.410 MW

The negative `p_from_mw` indicates that power is flowing opposite to the line's stored `from_bus → to_bus` orientation.

This demonstrates that high PV penetration can cause reverse active-power flow on sections of a distribution feeder even while the substation continues supplying other parts of the network.

---

## Reactive-Power Sensitivity

Before implementing automatic control, several fixed reactive-power values were tested for the 1.5 MW PV plant at Bus 17.

Negative Q represents reactive-power absorption in the model.

For example:

| Q | Bus 17 voltage | Losses |
|---:|---:|---:|
| 0 MVAr | 1.0163 p.u. | 0.1720 MW |
| -0.1 MVAr | 1.0104 p.u. | 0.1815 MW |
| -0.2 MVAr | 1.0044 p.u. | 0.1926 MW |
| -0.3 MVAr | 0.9983 p.u. | 0.2054 MW |
| -0.5 MVAr | 0.9855 p.u. | 0.2365 MW |

The results show that reactive-power absorption can reduce the local voltage rise, but it can also increase feeder losses.

---

## Volt-VAR Control

An illustrative Volt-VAR controller was implemented to automatically adjust inverter reactive power according to the voltage at Bus 17.

The control curve was:

- below 0.98 p.u.: reactive-power injection
- 0.98–1.00 p.u.: reduce injection toward zero
- 1.00–1.02 p.u.: increase reactive-power absorption
- above 1.02 p.u.: maximum reactive-power absorption

These voltage thresholds are illustrative and are not intended to represent a specific utility interconnection standard.

A damping factor of 0.25 was used to prevent the iterative controller from oscillating.

At 1.5 MW PV:

| Quantity | Q = 0 | Volt-VAR |
|---|---:|---:|
| Bus 17 voltage | 1.0163 p.u. | 1.0066 p.u. |
| Minimum feeder voltage | 0.9379 p.u. | 0.9362 p.u. |
| Maximum feeder voltage | 1.0163 p.u. | 1.0066 p.u. |
| Active-power losses | 0.1720 MW | 0.1884 MW |
| Reactive power | 0 MVAr | -0.1642 MVAr |

The controller reduced the local overvoltage while increasing active-power losses in this particular high-PV case.

---

## Inverter Capability

An illustrative 1.6 MVA inverter rating was assumed.

The physical reactive-power capability at a given active-power output was calculated from:

$$\
Q_{max} = \sqrt{S^2-P^2}
\$$

The Volt-VAR controller was separately limited to ±0.5 MVAr.

The physical capability was checked to ensure that the controller's ±0.5 MVAr operating range was feasible at the tested PV operating points.

At 1.5 MW:

$$\
Q_{max} =
\sqrt{1.6^2-1.5^2}
\approx0.557\ MVAr
\$$

Therefore, the ±0.5 MVAr controller limit remains within the assumed inverter capability.

---

## Volt-VAR Sensitivity to PV Penetration

The Volt-VAR controller was tested at several PV capacities at Bus 17.

| PV | Penetration | Q from controller |
|---:|---:|---:|
| 0.4 MW | 10.77% | +0.500 MVAr |
| 0.8 MW | 21.53% | +0.286 MVAr |
| 1.0 MW | 26.92% | +0.151 MVAr |
| 1.2 MW | 32.30% | +0.022 MVAr |
| 1.5 MW | 40.38% | -0.164 MVAr |

An important observation is that the controller changes from reactive-power injection at lower PV penetration to reactive-power absorption at high PV penetration.

At lower PV output, the feeder still has relatively low voltage, so reactive-power injection is useful.

At high PV output, the PV begins causing local voltage rise, so the controller switches to reactive-power absorption.

---

## Limitations
- PV output is represented by fixed operating points rather than time-series data
- the Volt-VAR curve is illustrative
- the inverter rating is an explicit modelling assumption
- realistic line thermal limits were not evaluated because the benchmark's supplied line current ratings are not meaningful physical limits for this study.
- unbalanced three-phase effects are not modeled

---

## Conclusion
Increasing PV can improve voltage and reduce losses at lower penetration levels, but these effects change as PV penetration increases. The location of the PV plant also matters because the same capacity can produce different voltage and loss results at different buses.

At high PV penetration, the PV plant can export power upstream and cause local overvoltage. The Volt-VAR controller demonstrated one way an inverter can respond to this voltage rise by adjusting reactive power.

The project therefore focuses on understanding the relationship between PV penetration, PV location, voltage, losses, reverse power flow, and inverter reactive-power control.

## Tools Used

- Python
- pandapower
- pandas
- NumPy
- Matplotlib

---

## Reference
[1]  L. Thurner et al., "pandapower — an Open Source Python Tool for Convenient Modeling, Analysis and Optimization of Electric Power Systems," in IEEE Transactions on Power Systems, vol. 33, no. 6, pp. 6510-6521, Nov. 2018.
