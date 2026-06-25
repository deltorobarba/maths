<font color="blue">**Schrödinger Equation  $|\psi(t)\rangle = e^{-iHt} |\psi(0)\rangle$**

Quantum systems in chemistry and physics can be described and computed with either classical and quantum methods:

* **Model**: Classical (Newton Mechanics) vs Quantum (Schrödinger equation)
* **Type**: Static (State) vs Dynamical (Evolution)
* **Computing**: Classical Computing vs Quantum Computing

<font color="blue">*1. Classical (Newtonian) Mechanics (**Static and Dynamic**): $F = m \cdot a$*
* Ignores electrons and consider atoms as spheres connected by springs (force fields)
* **Classical - Dynamic**: Molecular dynamics (MD). How proteins enfold and move? GROMACS, NAMD, AMBER.
* **Classical - Static**: Molecular Docking / Energy Minimization. Geometrically: Does medication fit to enzyme? AutoDock, Rosetta.
* **Quantum computing methods:** Future: Navier Stokes (fluid dynamics) with HHL to solve linear systems and differential equations exponentially faster than classical computers. For weather and climate: simulate weather with a much finer grid (e.g. 100m)

<font color="blue">*2. Quantum Mechanics (**Static**): Time-independent Schrödinger Equation $\hat{H}|\psi\rangle = E|\psi\rangle$*
* Includes electrons. **Approximates Ground State via optimization** with variation $\delta \langle H \rangle = 0$ to find global optimum in energy landscape. Stationary state important for calculating binding energies: If $\psi$ is an eigenstate of $\hat{H}$, then time evolution is trivial: $\Psi(t) = \psi e^{-iEt/\hbar}$. Probability density $|\Psi(t)|^2$ remains constant over time = states are stationary.
* **Classical computing:** Hartree-Fock (mean field of $\psi$) but ignore correlation energies, DFT (electron density $\rho$) approximates correlation energies with functionals (but require correct functional assumed), Post-HF (Coupled cluster, configuration interaction) calculate correlation energies but only small systems since their complexity grows exponentially
* **Quantum computing** (**NISQ**): Variational Quantum Eigensolver. Quantum Advantage: finds correlation energy with entanglement.
  * In classical quantum chemistry, correlation energies are either ignored (HF), approximated (DFT), or, while theoretically calculable exactly, are only feasible for small quantum systems (post-Hartree-Fock methods like coupled cluster or configuration interaction).
  * For classical computers, complexity grows exponentially, this is why quantum computers will play a significant role in analyzing larger, more complex systems.

<font color="blue">*3. Quantum Mechanics (**Dynamic**): Time-dependent Schrödinger Equation $i\hbar \frac{\partial}{\partial t} \Psi(\mathbf{r}, t) = \hat{H} \Psi(\mathbf{r}, t)$*
* **Simulate Time Evolution** $e^{-i\hat{H}t}$ of Quantum State $|\Psi(t)\rangle = e^{-i\hat{H}t/\hbar} |\Psi(0)\rangle$ via Unitary Transformation. For Quantum Chaos, OTOC, Scrambling (Black Holes) reaction dynamics (observe how bonds break and reform during a collision). No forward theorem!
* **Classical computing**: Time-Dependent DFT (TD-DFT) used to classically calculate excitations (fluorescence, spectra). However only for short times and small systems. To solve the time-dependent equation $i\hbar \frac{\partial}{\partial t} \Psi = \hat{H} \Psi$ classically at scale, you have to apply operator $e^{-i\hat{H}t/\hbar}$ to wavefunction. In system with $N$ particles, complexity grows exponentially with $N$.
* **Quantum computing** (**Fault-Tolerant**): Central challenge in quantum dynamics (Simulating Quantum Time Evolution):
  * Molecule evolves under all its forces (Kinetic + Potential) simultaneously, but QC can only do one. But unitary propagator $U(t) = e^{-i\hat{H}t}$ is implemented on quantum hardware that only supports a discrete set of elementary gates at once.
  * Three approaches: LCU, Trotterization, Qubitization (with Quantum Walks and Block Encoding). And with the OTOC metric to measure information scrambling during quantum evolution (https://research.google/blog/a-verifiable-quantum-advantage/)


<font color="blue">*Appendix: Why we can solve the Schrödinger equation analytically only for very small systems (Quantum Chemistry)*
* **Analytical** solution for Schrödinger in Hydrogen atom. But a second electron add electron-electron repulsion = three-body problem.
* **Approximate** Schrödinger with DFT and HF on classical computers via **optimization (variation)** for larger systems than Hydrogen
* **Simulate** time evolution for even larger systems = Quantum computer doesn't store information anymore - **it is the Hilbert space!**
* Weil die Schrödinger Gleichung nicht exakt lösbar für größere Systeme: Rayleigh Ritz Verfahren aus Variationsanalyse und Born-Oppenheimer Aporoximation (Nukleus fixed, Elektronen für Energy Potential). Modelle und Verfahren:
  * Valenzbindungstheorie (Hybridisierung, lokale Bindungen)
  * Molekülorbitaltheorie (Delokalisierung, Konjugation, Homo/Lumo, LCAO-Methode)
  * Hartee-Fock, Post-Hartree Fock (correlation energies), DFT
  * Quantum Numbers (Besonderheit: Spinquantenzahl. Normale“ Schrödinger-Gleichung liefert nur $n$, $l$ und $m_l$. Spin ($m_s$=± 1/2) quanten- mechanische Eigenschaft. Erscheint wenn Quantenmechanik + Speziellen RT kombiniert (1928 Dirac-Gleichung).



# @title Qubitization: Quantum Walk, Block Encoding, LCU
# ==========================================
# OTOC (Quantum Implementation)
# ==========================================
from IPython.display import display, Markdown

print(40*"=")
print("Qubitization (Quantum Walk, Block Encoding, LCU)")
print(40*"=")

display(Markdown(r"* Qubitization is an algebraic trick. It transforms the problem of walking through time into a problem of walking through the eigenvalues of an operator."))
display(Markdown(r"* **LCU Decomposition:** Approximate operator by decomposing propagator $e^{-iHt}$ via Taylor series in weightes sum of unitaries $H = \sum_{l=0}^{L-1} \alpha_l U_l$. (non-unitary, represents energy). Then **Normalization:** $\lambda = \sum |\alpha_k|$ $\rightarrow$ $H/\lambda$. If $H=$ Dirac $\mathcal{D}$ = as benchmark, its algebraic structure (sum of Paulis) efficient."))
display(Markdown(r"* **LCU Oracles**: 1. ***PREPARE Oracle*** ($P$): encodes coefficients $\alpha_j$ into amplitudes of ancilla. Output is state where probability of measuring state $j$ is proportional to $\alpha_j$: $P|0\rangle = \frac{1}{\sqrt{\|\alpha\|_1}} \sum_{j=0}^{m-1} \sqrt{\alpha_j} |j\rangle$. ***2. SELECT Oracle*** ($S$) apply unitaries $U_l$ to target register state, conditioned on state of ancilla (entangles). If ancilla is state $|j\rangle$, it applies $U_j$ to target state $|\psi\rangle$. $S = \sum_{j=0}^{m-1} |j\rangle\langle j| \otimes U_j$ ***3. Un-prepare***: Apply inverse $P^\dagger$ to ancilla. ***4. Measure***: ancilla."))
display(Markdown(r"* **Block Encoding $U_H$:** LCU embeds $\hat{H}/\lambda$ in top-left corner of quantum walk $U_H = \begin{pmatrix} \hat{H}/\lambda & \dots \\ \dots & \dots \end{pmatrix}$. If Ancilla is $|0\rangle$, we apply $H$. If Ancilla is $|1\rangle$, apply junk."))
display(Markdown(r"* **Quantum Walk** $Q$: add reflections to $U$ turns it into walk that moves through energy spectrum. By reflecting junk back into system ($Q = R \cdot U$), a rotation is created. Eigenvalues of $Q$ are linked to eigenvalues $E$ of $H$ via $\mu = e^{\pm i \arccos(E/\lambda)}$ (energy mapping). Physics is in angle (Isomorphism): $E_k = \lambda \cos(\theta_k)$. QW transforms *energy information* (scalar magnitude) into *phase information* (how fast rotates). High Energy $\rightarrow$ Small Angle. Zero Energy $\rightarrow$ $\cos(90^\circ)=0$."))

print(40*"=")
print("Simulation: Qubitization for OTOC")
print(40*"=")

display(Markdown(r"* Example: OTOC (scrambling of black hole or complex molecule). Time Reversal in Qubitization: invert Reflection and Oracles. **Shows that TIME in Qubitization is actually an ANGLE in Hilbert space**."))
display(Markdown(r"* Scrambling: Even with 1 system qubit, measurement changes if Forward and Backward walks don't perfectly cancel out because of the $X$ gate (butterfly). Code is gate-based with no god-mode access to $e^{-iHt}$."))
print("\n")

# --- 1. Define the Oracles (Same as before) ---

class PrepareOracle(cirq.Gate):
    def __init__(self, coeffs):
        super(PrepareOracle, self).__init__()
        self.coeffs = coeffs
        self.l1 = sum(coeffs)
        self.theta = 2 * np.arccos(np.sqrt(coeffs[0] / self.l1))
    def _num_qubits_(self): return 1
    def _decompose_(self, qubits):
        yield cirq.ry(self.theta).on(qubits[0])

class SelectOracle(cirq.Gate):
    def _num_qubits_(self): return 2
    def _decompose_(self, qubits):
        anc, sys = qubits
        yield cirq.X(sys).controlled_by(anc, control_values=[0])
        yield cirq.Z(sys).controlled_by(anc, control_values=[1])

# --- 2. Define the Qubitization Walk Step (The Engine) ---

class QubitizationWalkStep(cirq.Gate):
    """A single step of the Quantum Walk Q."""
    def __init__(self, coeffs):
        super(QubitizationWalkStep, self).__init__()
        self.prep = PrepareOracle(coeffs)
        self.sel = SelectOracle()

    def _num_qubits_(self): return 2

    def _decompose_(self, qubits):
        anc, sys = qubits
        # Q = Reflection * LCU_Operator
        # Part 1: LCU Operator (Block Encoding)
        yield self.prep.on(anc)
        yield self.sel.on(anc, sys)
        yield cirq.inverse(self.prep).on(anc)
        # Part 2: Reflection on Ancilla
        yield cirq.Z(anc)

# --- 3. The OTOC Experiment as example use case for qubitization ---

def run_qubitization_otoc(steps=2):
    q_anc = cirq.NamedQubit("Ancilla")
    q_sys = cirq.NamedQubit("System")
    circuit = cirq.Circuit()

    walk_step = QubitizationWalkStep(coeffs=[0.6, 0.8])

    # STEP A: Forward Evolution (Forward Walk)
    for _ in range(steps):
        circuit.append(walk_step.on(q_anc, q_sys))

    # STEP B: The Butterfly Perturbation (V)
    # We apply X to the system
    circuit.append(cirq.X(q_sys))

    # STEP C: Backward Evolution (Inverse Walk)
    # This is the 'Time Reversal' unique to OTOCs
    for _ in range(steps):
        circuit.append(cirq.inverse(walk_step).on(q_anc, q_sys))

    # STEP D: Measurement (W)
    circuit.append(cirq.measure(q_sys, key='otoc_result'))

    print(f"--- Qubitization OTOC Circuit ({steps} steps) ---")
    print(circuit)

    simulator = cirq.Simulator()
    result = simulator.run(circuit, repetitions=1000)
    print("\nResults (Histogram):")
    print(result.histogram(key='otoc_result'))

run_qubitization_otoc(steps=4)
