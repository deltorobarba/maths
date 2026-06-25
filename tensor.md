### <font color="blue">Tensor Calculus I

* **Paralleltransport und Geodäten**
  * **Paralleltransport:** Die Operation, einen Vektor entlang einer Kurve so zu verschieben, dass er "in sich parallel" bleibt (sich also aus seiner eigenen Perspektive nicht ändert). Ein Vektorfeld $V^\mu$ ist entlang einer Kurve mit dem Parameter $\tau$ parallel verschoben, wenn die absolute Ableitung null ist: $\frac{DV^\mu}{d\tau} = \frac{dx^\nu}{d\tau} \nabla_\nu V^\mu = 0$
  * **Die Geodätengleichung:** Eine Geodäte ist die geradestmögliche Bahn in einem gekrümmten Raum (die Verallgemeinerung einer Geraden). In der Physik ist das beispielsweise die Flugbahn eines Objekts im freien Fall. Ein Teilchen bewegt sich auf einer Geodäte, wenn sein eigener Geschwindigkeitsvektor entlang seiner Bahn parallel transportiert wird: $\frac{d^2 x^\mu}{d\tau^2} + \Gamma^\mu_{\alpha\beta} \frac{dx^\alpha}{d\tau} \frac{dx^\beta}{d\tau} = 0$

* **Geodesic equation**: Geodesics are where the tangential component are zero (because geodesics are curves where the acceleration vector is always normal, light will follow this path)
  * geradeste Kurve in einem gekrümmten Raum, entlang derer der eigene **Tangentialvektor parallel transportiert** wird, sodass ihre intrinsische Beschleunigung verschwindet.
  * In der Geodesic equation ist der covariant derivate (connection) = Null!
  * In curved space, a straight path has **zero tangential acceleration** when we travel along it at constant speed.
  * **To compute geodesic curves, we need to find curves where the acceleration vector is <u>normal</u> to the surface:**

<font color="blue">$$0 = \Gamma_{i j}^k \frac{d u^i}{d \lambda} \frac{d u^j}{d \lambda} + \frac{d^2 u^k}{d \lambda^2}$$


* **Acceleration Vector = Christoffel Symbols + Second Fundamental Form**
  * Extrinsic approach: in geodesic curves, the acceleration vector is normal to the surface: $\frac{d^2 \vec{R}}{d \lambda^2}= \quad \quad \underbrace{\left(\frac{d^2 \vec{R}}{d \lambda^2}\right)^{\text {tangential }}}_{=0} \quad \quad \quad + \quad \quad \left(\frac{d^2 \vec{R}}{d \lambda^2}\right)^{\text {normal }}$
  * **Acceleration Vector = Christoffel Symbols Γ (tangential) + Second Fundamental Form L (normal)**

$$ \frac{d^2 \vec{R}}{d \lambda^2} =\left(\frac{d^2 u^k}{d \lambda^2}+\Gamma_{i j}^k \frac{d u^i}{d \lambda} \frac{d u^j}{d \lambda}\right) \frac{\partial \vec{R}}{\partial u^k}\quad + \quad L_{i j} \frac{d u^i}{d \lambda} \frac{d u^j}{d \lambda} \widehat{n}$$

  * $\frac{\partial \overline{e_j}}{\partial u^i}=\Gamma_{i j}^1 \overrightarrow{e_1}+\Gamma_{i j}^2 \overrightarrow{e_2}+L_{i j} \hat{n}$
  * $\frac{\partial^2 \vec{R}}{\partial u^i \partial u^j}=\Gamma_{i j}^1 \frac{\partial \vec{R}}{\partial u^1}+\Gamma_{i j}^2 \frac{\partial \vec{R}}{\partial u^2}+L_{i j} \hat{n}$

* **Second fundamental form $L_{i j} \hat{n}$ (normal vector)**
  * tells us how much of the normal vector we take
  * Formel: $L_{i j}=\frac{\partial^2 \vec{R}}{{\partial} u^i {\partial} u^j} \cdot \frac{\overrightarrow{e_i} \times \overrightarrow{e_j}}{\left\|\overrightarrow{e_i} \times \overrightarrow{e_j}\right\|}$

* **Christoffel symbols $\Gamma_{i j}^k$ or $\Gamma_{i j}^1$ and $\Gamma_{i j}^2$ = correction term or 'connection coefficients' in covariant derivative!**
  * **Christoffel-Symbole $\Gamma^k_{ij}$:** Das sind die Korrekturterme, die beschreiben, wie sich das Koordinatensystem selbst krümmt und dreht. Im Levi-Civita-Zusammenhang (torsionsfrei & metrikverträglich) werden sie direkt aus den Ableitungen der Metrik berechnet: $\Gamma^k_{ij} = \frac{1}{2} g^{kl} (\partial_i g_{jl} + \partial_j g_{il} - \partial_l g_{ij})$
  * Christoffel Symbols give tangential components of $\frac{\partial^2 \vec{R}}{\partial u^i \partial u^j}$ (how much of each tangent basis vector in tangent plane we take)
  * How local basis vectors change from point to point on a surface? (to define derivate vector in curved space)
  * Formel: $\Gamma^k_{ij} = \frac{1}{2} g^{kl} (\partial_i g_{jl} + \partial_j g_{il} - \partial_l g_{ij}) =\frac{\partial^2 \vec{R}}{\partial u^i \partial u^j} \cdot \frac{\partial \vec{R}}{\partial u^l} \mathfrak{g}^{l k}$

* **Connection/Covariant Derivative = Navigationssystem (Zusammenhang/Kovariante Ableitung) = Lie Algebra**
  * Wie vergleicht man Vektoren in verschiedenen Tangentialräumen? Man braucht eine Vorschrift, wie man ein Koordinatensystem "parallel" von einem Punkt zum anderen schiebt.
  * **Der Zusammenhang $\nabla$:** Er verallgemeinert die Richtungsableitung. Wenn Sie ein Vektorfeld $Y$ in Richtung von $X$ ableiten, lautet das Ergebnis in lokalen Koordinaten: $\nabla_X Y = \left( X^i \frac{\partial Y^k}{\partial x^i} + \Gamma^k_{ij} X^i Y^j \right) \frac{\partial}{\partial x^k}$
  * **Covariant derivative (= Connection = Lie Algebra) = infinitesimal!**
  * Covariant derivative called connection because connects between tangent spaces in a curved space, e.g. from $T_pS$ to $T_qS$
  * **Die Geodesic equation ist eine spezielle Lösung der covariant derivate (connection), wo covariant derivative = Null ist!**
  * Covariant derivative is a tool used to measure the rate of change of a tensor field along a specific direction in a curved space or curved spacetime. It is a derivative with normal componentent substracted of a vector field on curved surfaces: $\nabla_{\text{Direction}} \text{Input Field} = \text{Output Field (rate of change)} = \text{Connection}$
  * Background: In flat Cartesian spaces, calculating derivative of vector or tensor simple: take partial derivative of its components. In curved spaces (or curvilinear like polar or spherical coordinates), coordinate basis vectors change from point to point.
  * If you take a standard partial derivative, the result incorrectly mixes the change of the tensor's components with the change in the basis vectors themselves, meaning the result is not a true tensor. The covariant derivative fixes this by properly adjusting for the twisting, turning, and stretching of the coordinate system.
  * The covariant derivative introduces **correction terms using mathematical objects called connection coefficients (most commonly the Christoffel symbols, denoted as ($\Gamma$)**
  * Standard partial derivative $\frac{{\partial} v^k}{{\partial} u^i}$ of the vector component and correction term $\Gamma$: $\nabla_{\frac{\partial}{\partial u^i}} \vec{v} = \left[\frac{\boldsymbol{\partial} v^k}{\boldsymbol{\partial} u^i}+v^j \boldsymbol{\Gamma}_{i j}^k\right] \overrightarrow{e_k}$
    * $\text{extrinsic (with dot product):} \quad \Gamma_{i j}^m = \left(\frac{\partial \overrightarrow{e_j}}{\partial u^i} \cdot \overrightarrow{e_l}\right) g^{l m}$
    * $\text{intrinsic:} \quad \Gamma_{j k}^m=\frac{1}{2} g^{m i}\left(\frac{\partial\left(g_{i j}\right)}{\partial u^k}+\frac{\partial\left(g_{k i}\right)}{\partial u^j}-\frac{\partial\left(g_{j k}\right)}{\partial u^i}\right)$
  * Example of a covariant derivative: $\nabla_{\frac{d}{d \lambda}} \frac{d}{d \lambda}$
    * $\lambda$ : Curve = some function of parameter $\lambda$. Tangent Vectors: $\nabla \frac{d}{d \lambda}$. Geodesic: $\nabla_{\frac{d}{d \lambda}} \frac{d}{d \lambda}$
    * Parallel transport if covariant derivative is 0: $\nabla_{\frac{d}{d \lambda}} \vec{v}=\overrightarrow{0}$ (covariant derivative of the vector field is zero)
  * Properties of Covariant Derivates - generalizes partial derivatives so that resulting derivative transforms properly as a tensor:
    * $\nabla_{a \vec{w}+b \vec{t}} \vec{v}=a \nabla_{\vec{w}} \vec{v}+b \nabla_{\vec{t}} \vec{v}$
    * $\nabla_{\vec{w}}(\vec{v}+\vec{u})=\nabla_{\vec{w}} \vec{v}+\nabla_{\vec{w}} \vec{u}$
    * $\nabla_{\vec{w}}(a \vec{v})=\left(\nabla_{\vec{w}} a\right) \vec{v}+a\left(\nabla_{\vec{w}} \vec{v}\right)$
    * $\nabla_{\partial_i}(a)=\frac{\partial a}{\partial u^i}$


* **Parallel Transport (global) = zero covariant derivative (infinitesimal)! = Exponential der Lie Algebra**  
  * ist eine Vorschrift, die bestimmt, wie ein Vektor entlang einer Kurve verschoben wird, sodass er im geometrischen Sinne "konstant" bleibt und die lokale Krümmung der Koordinaten exakt ausgleicht.
  Formel: $\nabla_{\dot{\gamma}} V = 0 \iff \frac{dV^i}{d\lambda} + \Gamma^i_{jk} V^j \frac{dx^k}{d\lambda} = 0$
  * Parallel Transport provides a connection between tangent spaces in a curved space. e.g. from $T_pS$ to $T_qS$
  * And the covariant derivative provides the connection between these tangent spaces in curved space $\nabla_{\frac{d}{d \lambda}} \vec{v}=\overrightarrow{0}$
  * Thats why covariant derivate is something called connection, because it connects!
  * This means: Parallel Transport = zero covariant derivative = special case of connection
  * And Christoffel symbols $\Gamma$ are connection coefficients! $\nabla_{\overrightarrow{e_i}} \overrightarrow{e_j}=\Gamma_{i j}^k \overrightarrow{e_k}$

> <font color="blue">**A connection is a Lie Algebra** ! Und der **Paralleltransport ist das Exponential** e dieses Lie-Algebra-Wertes entlang des Weges (das "path-ordered exponential"), was uns ein Element der Lie-Gruppe liefert. - Die Konnektion ($\nabla$, in der Algebra) ist also wirklich das Infinitesimale, und der Paralleltransport ($P_\gamma$, in der Gruppe) ist dessen Exponentiation!

* **Different connection coefficients will give us different instructions for performing parallel transport!**
  * Insight 1: The coveriant derivate is zero $\nabla_{\vec{v}}(\vec{u})=\overrightarrow{0}$ and $\nabla_{\vec{u}}(\vec{v})=\overrightarrow{0}$, but results can look vastly different.
  * Insight 2: Christoffel symbols $\Gamma$ are NOT unique, there are many different ways to describe the connection coefficients!
  * **Levi-Civita Connection = most important covariant derivative!:** = Two properies make Christoffel symbols unique:
    * **Torsion free property**: $\nabla_{\vec{w}} \vec{v}=\nabla_{\vec{v}} \vec{w}$ or written: $\nabla_{\vec{w}} \vec{v}-\nabla_{\vec{v}} \vec{w}=[\vec{v}, \vec{w}]$ with **Lie Bracket** included $[\vec{v}, \vec{w}]=\vec{v} \vec{w}-\vec{w} \vec{v}$
    * **Metric compatability property**: $\nabla_{\vec{w}}(\vec{v} \cdot \vec{u})=\left(\nabla_{\vec{w}} \vec{v}\right) \cdot \vec{u}+\vec{v} \cdot\left(\nabla_{\vec{w}} \vec{u}\right)$
    * Ein affiner Zusammenhang ermöglicht die richtungsabhängige (kovariante) Ableitung von Tensorfeldern; der Levi-Civita-Zusammenhang ist der eindeutige Zusammenhang, der sowohl torsionsfrei ist als auch die Metrik respektiert (die Längen und Winkel bei Paralleltransport erhält) = Torsion free + Metric compatability
    * $\nabla_Z g = 0 \quad \text{und} \quad \nabla_X Y - \nabla_Y X = [X,Y]$
    * $\nabla_{\frac{\partial}{\partial u^i}} \vec{v}=\left(\frac{\partial v^k}{\partial u^i}+v^j \Gamma_{i j}^k\right) \overrightarrow{e_k} $
    * $\Gamma_{j k}^m=\frac{1}{2} \mathfrak{g}^{i m}\left(\partial_k g_{i j}+\partial_j g_{k i}-\partial_i g_{j k}\right)$
  * **'Boring' connection**:

$$\widetilde{\nabla}_{\frac{\partial}{\partial u^i}} \vec{v}=\left(\frac{\partial v^k}{\partial u^i}+v^j \widetilde{\Gamma_{i j}^k}\right) \overrightarrow{e_k}$ wobei: $\widetilde{\Gamma_{i j}^k}=0$$

  * Der Unterschied zwischen den Endvektoren der perfekt parallel transportierten Vektoren in Vektorfeldern wird durch den **Torsion Tensor** gemessen.
  * Der Unterschied zwischen den Endvektoren der tatsächlich transportierten Vektoren in Vektorfeldern wird durch die **Lie Braket** gemessen.



---

### <font color="blue">Tensor Calculus II


* **Lie Bracket (Commutator):** (hier hat Parallel Transport keine bedeutung!)
  * Sie misst die Nicht-Kommutativität zweier Vektorfelder $X$ und $Y$, also gewissermaßen die infinitesimale "Lücke", die entsteht, wenn man die Flüsse der beiden Felder in unterschiedlicher Reihenfolge durchläuft.
  * Coordinate lines are just flow curves along basis vectors
  * Lie Bracket / Commutator = measures how much vector field flow curves fail to close.
  * $[\vec{u}, \vec{v}]=\vec{u}(\vec{v})-\vec{v}(\vec{u})$
  * $[X, Y]^i = X^j \partial_j Y^i - Y^j \partial_j X^i$

* **Torsion Tensor: = covariant derivates  + Lie Bracket!** (hier hat parallel transport eine zentrale bedeutung!)
  * Formel: $T(\vec{u}, \vec{v})=\nabla_{\vec{u}} \vec{v}-[\vec{u}, \vec{v}]-\nabla_{\vec{v}} \vec{u}$ mit $T^k_{ij} = \Gamma^k_{ij} - \Gamma^k_{ji}$
  * **wenn** $T(\vec{u}, \vec{v})=\nabla_{\vec{u}} \vec{v}-[\vec{u}, \vec{v}]-\nabla_{\vec{v}} \vec{u} = 0$, **dann**: $\nabla_{\vec{u}} \vec{v}-\nabla_{\vec{v}} \vec{u}=[\vec{u}, \vec{v}]$, **bedeutet**: the difference of the covariant derivates is equal to the Lie Braket (which is difference of ordinary derivative)
  * Dieser Tensor fängt den asymmetrischen Teil des Zusammenhangs ein und gibt geometrisch an, inwiefern sich infinitesimale Parallelogramme, die durch den Paralleltransport gebildet werden, nicht schließen. Separation between parallel transported vector lines. Lie bracket is included in it. "Torsion-Free" means parallel-transported vectors close properly.
  * This is a property of connections only! Only depends on the connection coefficients, not the vector field.
  * Notation: $\nabla_{\vec{u}} \vec{v} \quad \nabla_{\vec{v}} \vec{u}$ wobei: $\vec{u}$ and $\vec{v}$ = Vector fields, und $\nabla$ = Connection $\Gamma^k_{ij}$, concretely the Levi Civita connection $\Gamma_{j k}^m=\frac{1}{2} \mathfrak{g}^{i m}\left(\partial_k g_{i j}+\partial_j g_{k i}-\partial_i g_{j k}\right)$, sowie: $\nabla_{\vec{u}} \vec{v}=\overrightarrow{0}$ parallel transport
 
* **Krümmungstensoren**
  * In einem flachen euklidischen Raum spielt es keine Rolle, in welcher Reihenfolge man partielle Ableitungen ausführt. In einem gekrümmten Raum kommutieren kovariante Ableitungen hingegen *nicht*. Die Differenz dieser Ableitungen misst exakt die Krümmung des Raumes.
  * **Riemannscher Krümmungstensor ($R^\rho_{\sigma\mu\nu}$):** Er misst, wie sehr ein Vektor rotiert wird, wenn du ihn entlang einer winzigen geschlossenen Schleife parallel verschiebst: $[\nabla_\mu, \nabla_\nu] V^\rho = \nabla_\mu \nabla_\nu V^\rho - \nabla_\nu \nabla_\mu V^\rho = R^\rho_{\sigma\mu\nu} V^\sigma$
  * Die Formel zur Berechnung aus den Christoffel-Symbolen lautet: $R^\rho_{\sigma\mu\nu} = \partial_\mu \Gamma^\rho_{\nu\sigma} - \partial_\nu \Gamma^\rho_{\mu\sigma} + \Gamma^\rho_{\mu\lambda} \Gamma^\lambda_{\nu\sigma} - \Gamma^\rho_{\nu\lambda} \Gamma^\lambda_{\mu\sigma}$
  * **Ricci-Tensor ($R_{\mu\nu}$):** Eine bestimmte Verjüngung (Kontraktion) des Riemann-Tensors. Er beschreibt, wie das Volumen eines Schwarmes von Testteilchen im freien Fall schrumpft oder wächst: $R_{\mu\nu} = R^\rho_{\mu\rho\nu}$
  * **Ricci-Skalar ($R$):** Die weitere Verjüngung des Ricci-Tensors mit der Metrik ($R = g^{\mu\nu} R_{\mu\nu}$). Es ist eine einzelne Zahl, die an jedem Punkt die gesamte skalare Krümmung angibt.
  * Die **Einstein'schen Feldgleichungen** kombinieren genau diese drei Bausteine (Metrik, Ricci-Tensor, Ricci-Skalar), um zu beschreiben, wie Masse und Energie die Geometrie des Universums krümmen:


$$R_{\mu\nu} - \frac{1}{2} R g_{\mu\nu} + \Lambda g_{\mu\nu} = \frac{8\pi G}{c^4} T_{\mu\nu}$$

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
