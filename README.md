**Coker Solver**

**What is delayed coking?**
Delayed coking is a thermal-cracking (no catalyst) refinery process that upgrades the heaviest, lowest-value fraction of the crude barrel - vacuum residue (VR), the bottom product of the vacuum distillation column - into more valuable, lighter products. VR is heated in a furnace to a high temperature and routed into a large insulated vessel called a coke drum, where it is held (not actively heated further) for many hours. Held at reaction temperature with nowhere to go, the VR molecules crack: some break into small, volatile fragments that leave the drum as vapor, and the rest polymerize/condense into a solid, carbon-rich residue - petroleum coke - that builds up in the drum as a bed.

The process runs as a semi-batch cycle on a pair (or set) of drums: one drum fills with hot feed while the other, already full, is steamed, cooled, opened, and mechanically decoked (the coke bed is cut out with high-pressure water) before being sealed up and switched back into service. The period a single drum spends filling is usually called the "cycle time" or "fill time" in this tool.

**What comes out of the drum**
The hydrocarbon vapor leaving the top of the drum is fractionated downstream into gas, naphtha, and various gas-oil cuts. For the purposes of this tool those are grouped into two lumps:

**Gas** - light cracked gas (C1-C4 and similar very light material).
**Distillate** - the combined liquid product cuts (naphtha through gas oils).
The solid left behind in the drum is **coke**. Everything that hasn't yet reacted at a given point in time is tracked as unconverted vacuum residue (VR).

**Why a "lumped kinetic model"**
The real cracking chemistry inside a coke drum involves an enormous number of individual reactions across thousands of distinct molecules. A first-principles model of every molecule is neither practical nor necessary for predicting overall drum yields. Instead, this model uses a standard refinery-engineering simplification called lumping: every product molecule is grouped ("lumped") into one of a small number of pseudo-species - here, VR, Gas, Distillate and Coke - and the conversion of VR into each lump is described by a single empirical rate constant per lump, fitted so the lumped model reproduces observed plant/lab yield behavior.

**The reaction network**
VR is assumed to crack via three independent, parallel first-order pathways running simultaneously from the same pool of unconverted VR:

VR --K1--> Gas
VR --K2--> Distillate
VR --K3--> Coke
Each pathway is first-order in the remaining VR concentration, giving the governing differential equations:

d[VR]/dt        = -(K1 + K2 + K3) * [VR]
d[Gas]/dt       =  K1 * [VR]
d[Distillate]/dt =  K2 * [VR]
d[Coke]/dt      =  K3 * [VR]
[VR] starts at 1.0 (100% of the feed charged is unconverted at the instant it enters the drum) and the three product lumps start at 0. Because the three product equations only add material and never remove it, and the VR equation only ever loses exactly what the three gain, total mass is conserved automatically by construction: [VR] + Gas + Distillate + Coke = 1.0 at every instant. This tool checks that identity after every solve (to within 1×10-6) as a built-in integrity check on the numerics, not the chemistry.

**Where K1, K2, K3 come from**
K1, K2 and K3 are not universal physical constants - they are empirical correlations, calibrated for this specific model, expressed as functions of two feed/operating properties:

**CCR (Conradson Carbon Residue, wt%) **- a standard lab test that measures how much carbonaceous residue a feed leaves behind when heated in the absence of air. It is the single strongest indicator of how coke-prone a given VR is: high-CCR feeds make more coke and less liquid, all else equal.
**Drum temperature (°C, converted to Kelvin for the correlation)** - thermal cracking rates are strongly temperature-dependent (loosely Arrhenius-like: rate rises roughly exponentially with temperature), so drum operating temperature has a large effect on how far the reaction has progressed by a given point in the cycle.
Each K is of the general form A · CCR^m · T^n · exp(-B · CCR^p / T) - a power-law/CCR term combined with an Arrhenius-style exponential temperature term. The specific constants (A, m, n, B, p) differ between K1, K2 and K3 because gas formation, liquid formation and coke formation respond differently to feed quality and temperature. Feed density is used only to report the feed's API gravity alongside the results - it does not enter the rate correlations.

**From reacted fractions to physical results**
The ODE above gives dimensionless fractions of the original VR charge that have become each product by a given time - not yet tons or metres. To get physical results:

**Mass fed**: feed rate (m³/h) × density (kg/m³) ÷ 1000 gives feed rate in t/h; multiplied by however many hours that rate was actually fed, and summed across the whole schedule, gives total tons of VR charged to the drum.
Product tonnage: the model's final reacted fractions (once the whole fill time has elapsed) are applied to that total mass fed to get tons of gas, distillate and coke actually produced over the cycle.
**Coke bed height**: coke tonnage is converted to a coke volume using a coke bulk density, a fixed volume allowance is subtracted for the drum's bottom knuckle region, and the remaining volume is divided by the drum's cylindrical cross-sectional area to get a height - to which the knuckle section's own height is then added back on for the total bed height.

**Why throughput alone doesn't change the % yields**
A common intuition is that "feeding faster/slower" should itself change how much coke forms. In this model it doesn't, because K1/K2/K3 depend only on CCR and temperature - not on feed rate. Feed rate only scales how much total VR mass passed through the drum; it does not change the chemistry each parcel of that VR experiences. So with CCR and temperature held constant, a throughput schedule changes the total tons of coke made (and therefore the bed height), but not the wt% yield of coke on feed. If you want throughput itself to move the % yields, that would require your CCR or temperature to also change alongside throughput (which this tool supports per segment) - it isn't something feed rate does on its own in a first-order lumped model like this one.

**Why the multi-stage solve matters**
A real drum fill is rarely run at one constant rate for the whole cycle. This tool lets you break the fill into any number of stages, each with its own throughput (and optionally its own CCR/temperature). Instead of resetting the reacting mixture back to 100% fresh, unconverted VR at the start of every stage, the model carries the reacted-fraction state (VR/Gas/Distillate/Coke) forward from the end of one stage as the starting point of the next - because physically, the material already sitting in the drum doesn't "un-react" just because the feed valve setting changed. When CCR and temperature stay constant across stages, this stage-wise solve is mathematically identical to one continuous solve over the combined time (verified in the Validate panel to floating-point precision); it becomes essential the moment temperature or CCR is allowed to change mid-cycle.

**Model assumptions & limitations**
Lumped, first-order, parallel-reaction kinetics - a simplification of real cracking chemistry, not a molecular-level simulation.
K1/K2/K3 are functions of CCR and temperature only; other feed properties (e.g. aromaticity, asphaltene content, sulfur) are not explicit inputs, though CCR correlates with several of them in practice.
Assumes uniform temperature and composition at any instant (well-mixed reacting fraction), not a spatial/plug-flow profile up the height of the drum.
Coke bulk density and the drum's geometric constants (chamber area, volume, knuckle height) are fixed inputs to the height calculation, not solved for.
Valid input ranges enforced by the solver: CCR 0-40 wt%, density 700-1100 kg/m³, drum temperature 400-550°C. Results outside a feed/operating envelope similar to what the correlations were calibrated against should be treated with appropriate caution.

**Nomenclature
Symbol	Meaning**
**VR	Vacuum residue** - unconverted feed fraction remaining
**K1, K2, K3**	Rate constants for VR→Gas, VR→Distillate, VR→Coke (per hour)
**CCR**	Conradson Carbon Residue of the feed, wt%
**T**	Drum temperature, K (°C + 273.15) in the rate correlations
**Cycle time / fill time**	Total hours a drum spends receiving feed before switchover
**Segment / stage**	A sub-period of the cycle with its own throughput (and optionally CCR/temperature)
