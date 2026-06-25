

### <font color="blue">Tensor Calculus II


* **Lie Bracket (Commutator):** (hier hat Parallel Transport keine bedeutung!)
  * Sie misst die Nicht-Kommutativität zweier Vektorfelder $X$ und $Y$, also gewissermaßen die infinitesimale "Lücke", die entsteht, wenn man die Flüsse der beiden Felder in unterschiedlicher Reihenfolge durchläuft.
  * Coordinate lines are just flow curves along basis vectors
  * Lie Bracket / Commutator = measures how much vector field flow curves fail to close.

$$[\vec{u}, \vec{v}]=\vec{u}(\vec{v})-\vec{v}(\vec{u})$$

$$[X, Y]^i = X^j \partial_j Y^i - Y^j \partial_j X^i$$

* **Torsion Tensor: = covariant derivates  + Lie Bracket!** (hier hat parallel transport eine zentrale bedeutung!)

$$T(\vec{u}, \vec{v})=\nabla_{\vec{u}} \vec{v}-[\vec{u}, \vec{v}]-\nabla_{\vec{v}} \vec{u}$ mit $T^k_{ij} = \Gamma^k_{ij} - \Gamma^k_{ji}$$

* **wenn** 

$$T(\vec{u}, \vec{v})=\nabla_{\vec{u}} \vec{v}-[\vec{u}, \vec{v}]-\nabla_{\vec{v}} \vec{u} = 0$$

* **dann**: 

$$\nabla_{\vec{u}} \vec{v}-\nabla_{\vec{v}} \vec{u}=[\vec{u}, \vec{v}]$$

  * **bedeutet**: the difference of the covariant derivates is equal to the Lie Braket (which is difference of ordinary derivative)
  * Dieser Tensor fängt den asymmetrischen Teil des Zusammenhangs ein und gibt geometrisch an, inwiefern sich infinitesimale Parallelogramme, die durch den Paralleltransport gebildet werden, nicht schließen. Separation between parallel transported vector lines. Lie bracket is included in it. "Torsion-Free" means parallel-transported vectors close properly.
  * This is a property of connections only! Only depends on the connection coefficients, not the vector field.
  * Notation: $\nabla_{\vec{u}} \vec{v} \quad \nabla_{\vec{v}} \vec{u}$ wobei: $\vec{u}$ and $\vec{v}$ = Vector fields, und $\nabla$ = Connection $\Gamma^k_{ij}$, 
  * concretely the Levi Civita connection:
  
  $$\Gamma_{j k}^m=\frac{1}{2} \mathfrak{g}^{i m}\left(\partial_k g_{i j}+\partial_j g_{k i}-\partial_i g_{j k}\right)$$
  
  * sowie: $\nabla_{\vec{u}} \vec{v}=\overrightarrow{0}$ parallel transport

* **Riemann Curvature Tensor (Riemannscher Krümmungstensor):**
  * Krümmung ist die Unfähigkeit von Vektoren, beim Paralleltransport entlang einer geschlossenen Schleife wieder identisch bei sich selbst anzukommen (Holonomie), oder das Versagen der Kommutativität von Ableitungen.
* Two main ways to detect curvature: 1. **holonomy** or 2. **geodesic deviation**
  * Riemann Curvature Tensor: $R(\vec{u}, \vec{v}) \vec{w}=\nabla_{\vec{u}} \nabla_{\vec{v}} \vec{w}-\nabla_{\vec{v}} \nabla_{\vec{u}} \vec{w}-\nabla_{[\vec{u}, \vec{v}]} \vec{w}$
  * Riemann-Krümmungstensor $R$: Ein $(1,3)$-Tensor, der diese Nicht-Kommutativität exakt misst: $R(X,Y)Z = \nabla_X \nabla_Y Z - \nabla_Y \nabla_X Z - \nabla_{[X,Y]} Z$
  * (1 contravariant,3 covariant)-Tensor quantifiziert intrinsisch die Krümmung des Raumes, indem er die Nicht-Kommutativität der zweiten kovarianten Ableitungen misst.
  * Riemann curvature tensor **components** berechnet mit den connection components: $R_{c a b}^d=\partial_a\left(\Gamma_{b c}^d\right)-\partial_b\left(\Gamma_{a c}^d\right)+\Gamma_{b c}^i \Gamma_{a i}^d-\Gamma_{a c}^j \Gamma_{b j}^d$
* **Holonomie:** one way to detect curvature!
  * beschreibt die globale Diskrepanz – oft eine Drehung –, die ein Vektor erfährt, nachdem er auf einer gekrümmten Mannigfaltigkeit parallel entlang einer vollständig geschlossenen Schleife transportiert wurde.
  * Formel: $\text{Hol}(\gamma) = \mathcal{P} \exp \left( \oint_\gamma \Gamma_\mu dx^\mu \right)$
* **Ricci-Curvature-Tensor**
  * Aus dem Riemann-Tensor gewinnt man durch Spurbildung (Kontraktion) den **Ricci-Tensor** $R_{ij}$ (beschreibt Volumenverzerrungen) und den **Ricci-Skalar** $R$, welche das Herzstück der Einsteinschen Feldgleichungen in der Allgemeinen Relativitätstheorie bilden. Both considered a summary of the Riemann curvature tensor
  * **Ricci-Curvature-Tensor** tracks how volume change along geodesic (that's why the Ricci tensor represents gravity $R_{\mu \nu}$ in the general theory of relativity in Einstein field equations: $R_{\mu \nu}-\frac{1}{2} g_{\mu \nu}+\Lambda g_{\mu \nu}=\frac{8 \pi G}{c^4} T_{\mu \nu}$)
  * Formel: $\text{Ric}(\vec{v}, \vec{v})=\sum_{i=1}^D K\left(\overrightarrow{e_i}, \vec{v}\right)=\sum_i \frac{\left[R\left(\overrightarrow{e_i}, \vec{v}\right) \vec{v}\right] \cdot \overrightarrow{e_i}}{\left(\overrightarrow{e_i} \cdot \overrightarrow{e_i}\right)(\vec{v} \cdot \vec{v})-\left(\overrightarrow{e_i} \cdot \vec{v}\right)^2}$ for an orthonormal basis $\overrightarrow{e_1}, \overrightarrow{e_2}, \ldots, \overrightarrow{e_n}$ and a direction vector $\vec{v}=\overrightarrow{e_n}$. The Ricci curvature is the sum of all scalar curvatures in every basis vector direction $\sum_{i=1}^D K\left(\overrightarrow{e_i}, \vec{v}\right)$.
  * The Ricci curvature (left) can also be computed using the Ricci tensor $R_{i j}$ with: $\text{Ric}(\vec{v}, \vec{v})=R_{i j} v^i v^j$
* There are two geometrical approaches in understanding Ricci-Curvature-Tensor:
  * <u>**Sectional Curvature**</u> for an orthonormal base (with Levi Civita symbol). The Ricci-Curvature-Tensor is zero: $\text{Ric}(\vec{v}, \vec{v})=\sum_{i=1}^D K\left(\overrightarrow{e_i}, \vec{v}\right)=0$ for following **Sectional curvatures** (whose sum is the Ricci curvature): $K\left(\overrightarrow{e_1}, \vec{v}\right)=0 \quad K\left(\overrightarrow{e_2}, \vec{v}\right)=0$. General Sectional curvatures: $K(\vec{s}, \vec{v})=\frac{[R(\vec{s}, \vec{v}) \vec{v}] \cdot \vec{s}}{(\vec{s} \cdot \vec{s})(\vec{v} \cdot \vec{v})-(\vec{s} \cdot \vec{v})^2}$
  * <u>**Volume Element Derivative**:</u> for any base!
  * Achtung: Ricci Curvature-Tensor can only tell us about volume, not shape! because if $K\left(\overrightarrow{e_1}, \vec{v}\right)>0 \quad K\left(\overrightarrow{e_2}, \vec{v}\right) < 0$, then the overall Ricci tensor is the same: $\text{Ric}(\vec{v}, \vec{v})=\sum_{i=1}^D K\left(\overrightarrow{e_i}, \vec{v}\right)=0$, only the shape changed.
  * **Volume Form $ω$** ??
    * Take 2nd derivative of volume form along a geodesic path to get the Ricci tensor $\nabla_{\vec{v}} \nabla_{\vec{v}} \omega(\vec{u}, \vec{w}, \vec{t})$.
    * Metric compatability of Levi-Civita connection preserves vector lengths and angles, and therefore preserves volume.
    * This means covariant derivate is zero: $\nabla_{\vec{v}} V=0$ where this formula is: $=\nabla_{\vec{v}}[\omega(\vec{a}, \vec{b}, \vec{c})]$

* **Ricci-Skalar**: compare volume of a ball in curved space to volume of a ball of same radius in flat space (measures deviation)
    * curved space means we can fit a large area into a smaller boundary
  * Der Ricci-Tensor entsteht durch Spur-Bildung (Kontraktion) des Riemann-Tensors und misst die Volumenänderung eines geodätischen Balls; eine weitere Kontraktion liefert den Ricci-Skalar, eine einzige Zahl pro Punkt, die die totale Skalarkrümmung angibt. $R_{ij} = R^k_{\ ikj} \quad \text{und} \quad R = g^{ij} R_{ij}$
  * Erinnerung, das sind alle die gleiche Notationen für Basis vectors: $\overrightarrow{e_k} \equiv \frac{\partial}{\partial x^k} \equiv \partial_k$

* **Bianchi-Identität:**
  * Diese differentiellen Identitäten sind die geometrischen Entsprechungen der Erhaltungssätze (wie Energie und Impuls) und diktieren, wie sich die Krümmung von Punkt zu Punkt stetig und konsistent verhalten muss.
  * Formel: $\nabla_l R^i_{\ jkm} + \nabla_k R^i_{\ jml} + \nabla_m R^i_{\ jlk} = 0$

* **Schnittkrümmung (Sectional Curvature):**
  * liefert eine anschauliche zweidimensionale Krümmungsmaßzahl (vergleichbar mit der Gaußschen Krümmung), die angibt, wie stark eine durch zwei Tangentialvektoren $X$ und $Y$ aufgespannte "Ebene" gekrümmt ist.
  * Formel: $K(X,Y) = \frac{\langle R(X,Y)Y, X \rangle}{\langle X,X \rangle \langle Y,Y \rangle - \langle X,Y \rangle^2}$


Videos: Eigenchris: [Tensor Calculus](https://youtube.com/playlist?list=PLJHszsWbB6hpk5h8lSfBkVrpjsqvUGTCx&si=vJ64vdbqtgjDXHbv) and [Tensors for Beginners](https://youtube.com/playlist?list=PLJHszsWbB6hrkmmq57lX8BV-o-YIOFsiG&si=Ibojdh_9nI3SIYgA)
