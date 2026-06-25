## <font color="blue">Tensor Algebra

![science](https://raw.githubusercontent.com/deltorobarba/science/main/nature.JPG)


### <font color="blue">*Tensoranalysis (Tensorkalkül)*

Während die multilineare Algebra Tensoren in flachen Vektorräumen betrachtet, bringt die Tensoranalysis **Analysis und Geometrie** ins Spiel: Sie untersucht Tensorfelder auf gekrümmten Räumen. Das ist die mathematische Basis der Allgemeinen Relativitätstheorie und der Kontinuumsmechanik.

1. Grundkonzepte und Notation

* **Mannigfaltigkeit (Manifold):** Ein (oft gekrümmter) Raum, der lokal wie ein flacher euklidischer Raum aussieht – etwa so, wie die kugelige Erdoberfläche aus der Nähe flach wirkt. Du kannst ihn lokal mit Koordinaten $x^\mu$ beschreiben.
* **Einstein'sche Summenkonvention:** Um das ständige Schreiben von Summenzeichen zu vermeiden, wird über Indizes, die in einem Produkt einmal oben (kontravariant) und einmal unten (kovariant) stehen, automatisch summiert:

$$a^\mu b_\mu = \sum_{\mu} a^\mu b_\mu$$


* **Tensorfeld:** Eine Zuordnung, die jedem Punkt der Mannigfaltigkeit einen Tensor zuweist (analog zu einem Vektorfeld, das z. B. die Windgeschwindigkeit an jedem Ort angibt).

2. Transformationsverhalten (Kontra- und Kovarianz)

In der Tensoranalysis definiert man Tensoren meist darüber, wie sich ihre Komponenten ändern, wenn du das Koordinatensystem von $x$ auf ein neues System $x'$ wechselst.

* **Kontravarianter Vektor (Index oben):** Vektoren im Tangentialraum (wie Geschwindigkeiten). Sie transformieren sich mit der Jacobi-Matrix der Koordinatentransformation:

$$V'^\mu = \frac{\partial x'^\mu}{\partial x^\nu} V^\nu$$


* **Kovarianter Vektor / 1-Form (Index unten):** Elemente des Dualraums (wie der Gradient einer skalaren Funktion). Sie transformieren sich mit der inversen Jacobi-Matrix:

$$W'_\mu = \frac{\partial x^\nu}{\partial x'^\mu} W_\nu$$


* **Allgemeiner $(p, q)$-Tensor:** Transformiert sich als Produkt aus kontravarianten und kovarianten Faktoren. Ein gemischter Tensor vom Typ $(1,1)$ transformiert sich beispielsweise so:

$$T'^\mu_\nu = \frac{\partial x'^\mu}{\partial x^\alpha} \frac{\partial x^\beta}{\partial x'^\nu} T^\alpha_\beta$$


3. Der Metrische Tensor ($g_{\mu\nu}$)

Der metrische Tensor ist der absolut wichtigste $(0,2)$-Tensor einer Riemannschen Mannigfaltigkeit. Er verleiht dem Raum seine Geometrie, da er Abstände und Winkel definiert.

* **Linienelement:** Das Quadrat des infinitesimalen Abstands $ds$ zwischen zwei unendlich nah benachbarten Punkten:

$$ds^2 = g_{\mu\nu} dx^\mu dx^\nu$$


* **Indizes heben und senken:** Die Metrik $g_{\mu\nu}$ und ihre Inverse $g^{\mu\nu}$ (für die $g_{\mu\alpha} g^{\alpha\nu} = \delta_\mu^\nu$ gilt) werden als Werkzeuge genutzt, um zwischen ko- und kontravarianten Komponenten hin- und herzuwechseln:

$$V_\mu = g_{\mu\nu} V^\nu \quad \text{und} \quad W^\mu = g^{\mu\nu} W_\nu$$


4. Die Kovariante Ableitung ($\nabla$)

Wenn du auf einem gekrümmten Raum (oder in Polarkoordinaten) die normale partielle Ableitung ($\partial_\mu$) eines Vektorfeldes bildest, ist das Ergebnis *kein* Tensor mehr. Der Grund: Die Basisvektoren selbst ändern sich von Punkt zu Punkt! Die kovariante Ableitung repariert dieses Problem.

* **Christoffel-Symbole ($\Gamma^\lambda_{\mu\nu}$):** Diese (nicht-tensoriellen!) Hilfsgrößen beschreiben, wie sich das Koordinatennetz krümmt und die Basisvektoren variieren. Sie werden direkt aus der Metrik und deren Ableitungen berechnet:

$$\Gamma^\lambda_{\mu\nu} = \frac{1}{2} g^{\lambda\rho} (\partial_\mu g_{\nu\rho} + \partial_\nu g_{\rho\mu} - \partial_\rho g_{\mu\nu})$$


* **Kovariante Ableitung ($\nabla_\mu$ oder $;_\mu$):** Sie ergänzt die gewöhnliche Ableitung um einen Korrekturterm mit dem Christoffel-Symbol, sodass das Resultat wieder ein echter Tensor ist.
* Für einen kontravarianten Vektor:

$$\nabla_\mu V^\nu = \partial_\mu V^\nu + \Gamma^\nu_{\mu\lambda} V^\lambda$$


* Für einen kovarianten Vektor:

$$\nabla_\mu W_\nu = \partial_\mu W_\nu - \Gamma^\lambda_{\mu\nu} W_\lambda$$


5. Paralleltransport und Geodäten

* **Paralleltransport:** Die Operation, einen Vektor entlang einer Kurve so zu verschieben, dass er "in sich parallel" bleibt (sich also aus seiner eigenen Perspektive nicht ändert). Ein Vektorfeld $V^\mu$ ist entlang einer Kurve mit dem Parameter $\tau$ parallel verschoben, wenn die absolute Ableitung null ist:

$$\frac{DV^\mu}{d\tau} = \frac{dx^\nu}{d\tau} \nabla_\nu V^\mu = 0$$


* **Die Geodätengleichung:** Eine Geodäte ist die geradestmögliche Bahn in einem gekrümmten Raum (die Verallgemeinerung einer Geraden). In der Physik ist das beispielsweise die Flugbahn eines Objekts im freien Fall. Ein Teilchen bewegt sich auf einer Geodäte, wenn sein eigener Geschwindigkeitsvektor entlang seiner Bahn parallel transportiert wird:

$$\frac{d^2 x^\mu}{d\tau^2} + \Gamma^\mu_{\alpha\beta} \frac{dx^\alpha}{d\tau} \frac{dx^\beta}{d\tau} = 0$$


6. Die Krümmungstensoren

In einem flachen euklidischen Raum spielt es keine Rolle, in welcher Reihenfolge man partielle Ableitungen ausführt. In einem gekrümmten Raum kommutieren kovariante Ableitungen hingegen *nicht*. Die Differenz dieser Ableitungen misst exakt die Krümmung des Raumes.

* **Riemannscher Krümmungstensor ($R^\rho_{\sigma\mu\nu}$):** Er misst, wie sehr ein Vektor rotiert wird, wenn du ihn entlang einer winzigen geschlossenen Schleife parallel verschiebst:

$$[\nabla_\mu, \nabla_\nu] V^\rho = \nabla_\mu \nabla_\nu V^\rho - \nabla_\nu \nabla_\mu V^\rho = R^\rho_{\sigma\mu\nu} V^\sigma$$



Die Formel zur Berechnung aus den Christoffel-Symbolen lautet:

$$R^\rho_{\sigma\mu\nu} = \partial_\mu \Gamma^\rho_{\nu\sigma} - \partial_\nu \Gamma^\rho_{\mu\sigma} + \Gamma^\rho_{\mu\lambda} \Gamma^\lambda_{\nu\sigma} - \Gamma^\rho_{\nu\lambda} \Gamma^\lambda_{\mu\sigma}$$


* **Ricci-Tensor ($R_{\mu\nu}$):** Eine bestimmte Verjüngung (Kontraktion) des Riemann-Tensors. Er beschreibt, wie das Volumen eines Schwarmes von Testteilchen im freien Fall schrumpft oder wächst:

$$R_{\mu\nu} = R^\rho_{\mu\rho\nu}$$


* **Ricci-Skalar ($R$):** Die weitere Verjüngung des Ricci-Tensors mit der Metrik ($R = g^{\mu\nu} R_{\mu\nu}$). Es ist eine einzelne Zahl, die an jedem Punkt die gesamte skalare Krümmung angibt.

Zum Abschluss: Die berühmten **Einstein'schen Feldgleichungen** kombinieren genau diese drei Bausteine (Metrik, Ricci-Tensor, Ricci-Skalar), um zu beschreiben, wie Masse und Energie die Geometrie des Universums krümmen:


$$R_{\mu\nu} - \frac{1}{2} R g_{\mu\nu} + \Lambda g_{\mu\nu} = \frac{8\pi G}{c^4} T_{\mu\nu}$$

---

<font color="blue">***Tensor Algebra***

<font color="blue">*Vektor und Kovektor*

* **Vektoren als Derivationen** (kontravariant) als Richtungsableitungen einer Funktion, zB: $\vec{e_x} = \frac{\delta}{ \delta_x}$
  * Vector: $\vec{v} = v^i \vec{e}_i = \widetilde{v}^i \vec{\widetilde{e}}_i$
  * **Basis Vectors / Covariant (1,0-Tensor)**: $\vec{\widetilde{e}}_j = F_j^i \vec{e}_i \qquad \vec{e}_j = B_j^i \vec{\widetilde{e}}_i$
  * **Vector components / Contravariant (0,1-Tensor)**: $\widetilde{v}^i = B_j^i v^j \qquad v^i = F_j^i \widetilde{v}^j$
* **Kovektoren als Differentialformen** (kovariant) als "Schichten", die von Vektoren durchstoßen werden
  * Kovector: $\alpha = \alpha_i \epsilon^i = \widetilde{\alpha}_i \widetilde{\epsilon}^i$
  * **Basis Covectors / Contravariant (0,1-Tensor)**: $\widetilde{\epsilon}^i = B_j^i \epsilon^j \qquad \epsilon^i = F_j^i \widetilde{\epsilon}^j$
  * **Covector Components / Covariant (1,0-Tensor)**: $\widetilde{\alpha}_j = F_j^i \alpha_i \qquad \alpha_j = B_j^i \widetilde{\alpha}_i$
  * Note: **Differential Forms are totally antisymmetric rank (0,k) tensors!** [video](https://www.youtube.com/watch?v=dNT8OLNMsQ0&list=PLBn8lN0DcvpljsQ8SbFGJxyd_0IlRwiMf&index=5$0)
  * Algebraic Topology meets Differential Geometry
  * Beispiel: Erste Ableitung bilden von $z = f(x,y)$: wir benötigen $Δx$ und $Δy$ für $Δf$
    * $\Delta f=\frac{\partial f}{\partial x} \Delta x+\frac{\partial f}{\partial y} \Delta y$
    * $d f=\frac{\partial f}{\partial x} d x+\frac{\partial f}{\partial y} d y$ (we can replace the deltas $Δ$ by $d$)
    * $d f=\frac{\partial f}{\partial x_1} d x_1+\frac{\partial f}{\partial x_2} d x_2+\cdots+\frac{\partial f}{\partial x_n} d x_n$ (can generalize to many variables)
  * Path integral $\gamma$: has Integrability: integral only depends on the boundary (Poincare 1887), e..g between points a and b: $\int_\gamma A_1 d x_1+A_2 d x_2+\cdots+A_n d x_n=\int_\gamma d f=f(b)-f(a)$
    * **Integrability Lemma from Poincare**: differential form omega $ω$ (as some combinations of $dx_i$ is integrable if some combinations of derivates equal 0: $d ω = 0$
  * Christoffel symbol are Differential 2-forms (Curvature 2 forms), easier written than Christoffel
    * $\begin{array}{rlrl}\Gamma_{j k}^i & =\frac{1}{2} g^{i l}\left(\partial_k g_{l j}+\partial_j g_{l k}-\partial_l g_{j k}\right) & d \theta^i+\omega_j^i \wedge \theta^j & =0 \\ R_{j k l}^i & =\partial_k \Gamma_{l j}^i-\partial_l \Gamma_{k j}^i+\Gamma_{k m}^i \Gamma_{l j}^m-\Gamma_{l m}^i \Gamma_{k j}^m & d \omega_j^i+\omega_k^i \wedge \omega_j^k & =\Omega_j^i\end{array}$
  * Differential 2-forms (Curvature 2 forms) also appeard in Gauge theory:
    * Differential geometry: $\quad d \omega_j^i+\omega_k^i \wedge \omega_j^k=\Omega_j^i$
    * Gauge theory: $\quad d \mathbf{A}+\mathbf{A} \wedge \mathbf{A}=\mathbf{F}$
  * Examples of differential forms:
    * Stoke's theorem: $\int_{\partial D} \overrightarrow{\mathbf{F}} \cdot d \overrightarrow{\mathbf{r}}=\int_D \nabla \times \overrightarrow{\mathbf{F}} \cdot d \overrightarrow{\mathbf{S}}$ - If curl is 0, there is Scalar potential: $\nabla \times \overrightarrow{\mathbf{E}}=\overrightarrow{\mathbf{0}} \Longrightarrow \overrightarrow{\mathbf{E}}=-\nabla V$
    * Divergence theorem: $\int_{\partial D} \overrightarrow{\mathbf{F}} \cdot d \overrightarrow{\mathbf{S}}=\int_D \nabla \cdot \overrightarrow{\mathbf{F}} d V$ - If divergence is 0, there is Vector Potential $\nabla \cdot \overrightarrow{\mathbf{B}}=0 \Longrightarrow \overrightarrow{\mathbf{B}}=\nabla \times \overrightarrow{\mathbf{A}}$
    * Generalized Stoke's theorem: $\int_{\partial D} \omega=\int_D d \omega$ - Poincare lemma $d F=0 \Longrightarrow F=d A$ (generalization of Scalar potential and Vector Potential)
* **Vektoren (Derivationen) vs Kovektoren (Differentialformen):**
  * **Vektoren (Derivationen):** Leben im Tangentialraum $T_pM$. Sie sind Richtungsableitungen. Basis: $\frac{\partial}{\partial x^i}$.
  * **Kovektoren (Differentialformen):** Leben im dualen Kotangentialraum $T^*_pM$. Sie messen Vektoren. Basis: $dx^i$.
  * Die Dualität ist fundamental: $dx^i\left(\frac{\partial}{\partial x^j}\right) = \delta^i_j$ (Kronecker-Delta).


<font color="blue">*Felder (Vektorfeld, Kovektorfeld/Differentialform)*

* **Vector fields** mit Basiswechsel (Funktor) und Jacobimatrix
  * Vector: $\vec{e}_1 \longleftrightarrow \frac{\partial}{\partial x} \qquad \vec{e}_2 \longleftrightarrow \frac{\partial}{\partial y}$ (derivatives)
  * Vector Fields (Kettenregel für Tangentialvektoren): $\frac{d}{d\lambda} = \frac{dc^i}{d\lambda} \frac{\partial}{\partial c^i} = \frac{dp^i}{d\lambda} \frac{\partial}{\partial p^i}$
  * Basis Vectors / Covariant: $\frac{\partial}{\partial p^j} = \frac{\partial c^i}{\partial p^j} \frac{\partial}{\partial c^i} \qquad \frac{\partial}{\partial c^j} = \frac{\partial p^i}{\partial c^j} \frac{\partial}{\partial p^i}$
  * Vector Components / Contravariant: $\frac{dp^i}{d\lambda} = \frac{\partial p^i}{\partial c^j} \frac{dc^j}{d\lambda} \qquad \frac{dc^i}{d\lambda} = \frac{\partial c^i}{\partial p^j} \frac{dp^j}{d\lambda}$
* **Covector fields** mit Basiswechsel (Funktor) und Jacobimatrix
  * Covector: $\epsilon^1 \longleftrightarrow dx \qquad \epsilon^2 \longleftrightarrow dy$ (differentials)
  * Covector Fields (Totales Differential einer Funktion): $df = \frac{\partial f}{\partial c^i} dc^i = \frac{\partial f}{\partial p^i} dp^i$
  * Basis Covectors / Contravariant: $dp^i = \frac{\partial p^i}{\partial c^j} dc^j \qquad dc^i = \frac{\partial c^i}{\partial p^j} dp^j$


<font color="blue">*Räume*

* **Tangentialraum $T_pM$:** ein Punkt $p$ auf Mannigfaltigkeit und untersuchen die Vektorräume, die an diesem Punkt "kleben".
  * Vektoren (Derivationen) und Kovektoren (1-Formen) koordinatenfreie, abstrakte Objekte
* **Kotangentialraum**


<font color="blue">*Metrik*

* **Metrik Tensor (Bilinearform / First Fundamental Form):**
  * (0,2)-Tensor. Instrument zur Messung von Abständen und Winkeln, Indezes ziehen. Index Raising and Lowering: The most powerful feature of the metric tensor is its ability to convert covariant indices into contravariant ones, and vice versa (musikalische Isomorphismus)
  * Metrischer Tensor: symmetrisches, nicht-ausgeartetes Tensorfeld ein. Eine Bilinearform, die zwei Vektoren eine Zahl zuordnet: $ds^2 = g = g_{ij} \, dx^i \otimes dx^j$
  * $\widetilde{g}_{ij} = F^k_i F^l_j g_{kl}$
  * $g_{kl} = B^i_k B^j_l \widetilde{g}_{ij}$
  * **Metric Tensor:** covariant $g_{i,j}$ from $V$ with $\vec{v}$ to $V^*$, lives in $V^* \otimes V^*$.   * *Convert from vector to covector with metric tensor:*
    * $\vec{v} = v^j \vec{e}_j = \widetilde{v}^j \widetilde{\vec{e}}_j$ (vector)
    * $\vec{v} \cdot \_ = v_i \epsilon^i = \widetilde{v}_i \widetilde{\epsilon}^i$ (covector)
    * $v_i = g_{ij} v^j$ bzw. $\widetilde{v}_i = \widetilde{g}_{ij} \widetilde{v}^j$
    * $v_i = g_{ij} v^j$ bzw. $\widetilde{v}_i = \widetilde{g}_{ij} \widetilde{v}^j$
* Metric Tensor for Curved Surfaces:

$$
\begin{bmatrix}
\frac{d \vec{R}}{d u} \cdot \frac{d \vec{R}}{d u} & \frac{d \vec{R}}{d u} \cdot \frac{d \vec{R}}{d v} \\ 
\frac{d \vec{R}}{d v} \cdot \frac{d \vec{R}}{d u} & \frac{d \vec{R}}{d v} \cdot \frac{d \vec{R}}{d v}
\end{bmatrix}
$$

$\left[\begin{array}{cc}\frac{d \vec{R}}{d u} \cdot \frac{d \vec{R}}{d u} & \frac{d \vec{R}}{d u} \cdot \frac{d \vec{R}}{d v} \\ \frac{d \vec{R}}{d v} \cdot \frac{d \vec{R}}{d u} & \frac{d \vec{R}}{d v} \cdot \frac{d \vec{R}}{d v}\end{array}\right]$ bzw. als Vector: $\left[\begin{array}{ll}\overrightarrow{e_u} \cdot \overrightarrow{e_u} & \overrightarrow{e_u} \cdot \overrightarrow{e_v} \\ \overrightarrow{e_v} \cdot \overrightarrow{e_u} & \overrightarrow{e_v} \cdot \overrightarrow{e_v}\end{array}\right]$


* Sandwich Metric tensor between covector and vector: $g(\vec{v}, \vec{w})=v^i w^j g_{i j}$ = $=\left[\begin{array}{ll}v^1 & v^2\end{array}\right]\left[\begin{array}{ll}g_{11} & g_{12} \\ g_{21} & g_{22}\end{array}\right]\left[\begin{array}{l}w^1 \\ w^2\end{array}\right]$
    * Anwendung in u, v: $\left\|\frac{d \vec{R}}{d \lambda}\right\|^2=\frac{d \vec{R}}{d \lambda} \cdot \frac{d \vec{R}}{d \lambda}$ $=\left[\begin{array}{ll}\frac{d u}{d \lambda} & \frac{d v}{d \lambda}\end{array}\right]\left[\begin{array}{ll}\frac{d \vec{R}}{d u} \cdot \frac{d \vec{R}}{d u} & \frac{d \vec{R}}{d u} \cdot \frac{d \vec{R}}{d v} \\ \frac{d \vec{R}}{d v} \cdot \frac{d \vec{R}}{d u} & \frac{d \vec{R}}{d v} \cdot \frac{d \vec{R}}{d v}\end{array}\right]\left[\begin{array}{l}\frac{d u}{d \lambda} \\ \frac{d v}{d \lambda}\end{array}\right]$
    * Anwendung in X, Y, Z: $\left\|\frac{d \vec{R}}{d \lambda}\right\|^2=\frac{d \vec{R}}{d \lambda} \cdot \frac{d \vec{R}}{d \lambda}$ = $\left[\begin{array}{lll}\frac{d X}{d \lambda} & \frac{d Y}{d \lambda} & \frac{d Z}{d \lambda}\end{array}\right]\left[\begin{array}{lll}\frac{\partial \vec{R}}{\partial X} \cdot \frac{\partial \vec{R}}{\partial X} & \frac{\partial \vec{R}}{\partial X} \cdot \frac{\partial \vec{R}}{\partial Y} & \frac{\partial \vec{R}}{\partial X} \cdot \frac{\partial \vec{R}}{\partial Z} \\ \frac{\partial \vec{R}}{\partial Y} \cdot \frac{\partial \vec{R}}{\partial X} & \frac{\partial \vec{R}}{\partial Y} \cdot \frac{\partial \vec{R}}{\partial Y} & \frac{\partial \vec{R}}{\partial Y} \cdot \frac{\partial \vec{R}}{\partial Z} \\ \frac{\partial \vec{R}}{\partial Z} \cdot \frac{\partial \vec{R}}{\partial X} & \frac{\partial \vec{R}}{\partial Z} \cdot \frac{\partial \vec{R}}{\partial Y} & \frac{\partial \vec{R}}{\partial Z} \cdot \frac{\partial \vec{R}}{\partial Z}\end{array}\right]\left[\begin{array}{l}\frac{d X}{d \lambda} \\ \frac{d Y}{d \lambda} \\ \frac{d Z}{d \lambda}\end{array}\right]$
  * **Inverse Metric Tensor:** contravariant $\mathfrak{g_{i,j}}$, lives in $V \otimes V$
  * **Metric tensor and invers metric tensor with $∇ f$ (gradient, vector) and $d f$ (differential, covector):**
    * Use metric tensor $g = (\nabla f)^i g_{i j}=\frac{\partial f}{\partial c^j}$ to go from vector field **$∇ f$ (gradient)** to covector field **$d f$ (differential, exterior derivative of $f$)**.
    * Use inverse metric tensor $\mathfrak{g}$ = $(\nabla f)^k= \mathfrak{g}^{j k} \frac{\partial f}{\partial c^j}$ to go from covector field **$d f$ (differential, exterior derivative of $f$)** back to vector field **$∇ f$ (gradient)** .
  * **Kronecker delta** when both combined: $\mathfrak{g}^{k i} \otimes g_{i j}=\delta_j^k$
  * **(m,n)- Tensor** Transformation rule
    * Definition der Indizes: $T^{ijk\dots \quad  → \text{ contravariant indexes}}_{rst\dots \quad  → \text{ covariant indexes}}$
    * Transformation zu den neuen Indizes: $\widetilde{T}^{abc\dots}_{xyz\dots} = (B^a_i B^b_j B^c_k \dots) T^{ijk\dots}_{rst\dots} (F^r_x F^s_y F^t_z \dots)$
    * Transformation zu den alten Indizes: $T^{ijk\dots}_{rst\dots} = (F^i_a F^j_b F^k_c \dots) \widetilde{T}^{abc\dots}_{xyz\dots} (B^x_r B^y_s B^z_t \dots)$
  * **Musikalische Isomorphismen:** Die Metrik erlaubt es uns, Indizes zu heben (Sharp $\sharp$) und zu senken (Flat $\flat$). Sie verwandelt Vektoren in Formen und umgekehrt. $v_i = g_{ij} v^j \quad (\text{Flat-Operator: Vektor } \to \text{ Form})$
    * **Lowering an index:** $v_i = g_{ij} v^j$ which is the $\flat$ operator (from vector arrow to flat covector stack)
    * **Raising an index:** $v^i = g^{ij} v_j$ which is the $\sharp$ operator (from covector $\alpha$ to vector $\vec{a}$
  * (Note: The contravariant metric tensor $g^{ij}$ is the matrix inverse of the covariant tensor $g_{ij}$).
* (1,1)-Tensor
  * $\widetilde{L}^i_j = B^i_k L^k_l F^l_j$
  * $L^i_j = F^i_k \widetilde{L}^k_l B^l_j$

---

<font color="blue">*Tensor Calculus I*

* **Geodesic equation**: Geodesics are where the tangential component are zero (because geodesics are curves where the acceleration vector is always normal, light will follow this path)
  * geradeste Kurve in einem gekrümmten Raum, entlang derer der eigene **Tangentialvektor parallel transportiert** wird, sodass ihre intrinsische Beschleunigung verschwindet.
  * In der Geodesic equation ist der covariant derivate (connection) = Null!
  * In curved space, a straight path has **zero tangential acceleration** when we travel along it at constant speed.
  * **To compute geodesic curves, we need to find curves where the acceleration vector is <u>normal</u> to the surface:**

<font color="blue">$$0 = \Gamma_{i j}^k \frac{d u^i}{d \lambda} \frac{d u^j}{d \lambda} + \frac{d^2 u^k}{d \lambda^2}$$


* **Acceleration Vector = Christoffel Symbols + Second Fundamental Form**
  * Extrinsic approach: in geodesic curves, the acceleration vector is normal to the surface: $\frac{d^2 \vec{R}}{d \lambda^2}= \quad \quad \underbrace{\left(\frac{d^2 \vec{R}}{d \lambda^2}\right)^{\text {tangential }}}_{=0} \quad \quad \quad + \quad \quad \left(\frac{d^2 \vec{R}}{d \lambda^2}\right)^{\text {normal }}$
  * **Acceleration Vector = Christoffel Symbols Γ (tangential) + Second Fundamental Form L (normal)**
  * $ \frac{d^2 \vec{R}}{d \lambda^2} =\left(\frac{d^2 u^k}{d \lambda^2}+\Gamma_{i j}^k \frac{d u^i}{d \lambda} \frac{d u^j}{d \lambda}\right) \frac{\partial \vec{R}}{\partial u^k}\quad + \quad L_{i j} \frac{d u^i}{d \lambda} \frac{d u^j}{d \lambda} \widehat{n}$
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
  * **'Boring' connection**: $\widetilde{\nabla}_{\frac{\partial}{\partial u^i}} \vec{v}=\left(\frac{\partial v^k}{\partial u^i}+v^j \widetilde{\Gamma_{i j}^k}\right) \overrightarrow{e_k}$ wobei: $\widetilde{\Gamma_{i j}^k}=0$
  * Der Unterschied zwischen den Endvektoren der perfekt parallel transportierten Vektoren in Vektorfeldern wird durch den **Torsion Tensor** gemessen.
  * Der Unterschied zwischen den Endvektoren der tatsächlich transportierten Vektoren in Vektorfeldern wird durch die **Lie Braket** gemessen.



---

<font color="blue">*Tensor Calculus II*


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
  * Formel: $\operatorname{Ric}(\vec{v}, \vec{v})=\sum_{i=1}^D K\left(\overrightarrow{e_i}, \vec{v}\right)=\sum_i \frac{\left[R\left(\overrightarrow{e_i}, \vec{v}\right) \vec{v}\right] \cdot \overrightarrow{e_i}}{\left(\overrightarrow{e_i} \cdot \overrightarrow{e_i}\right)(\vec{v} \cdot \vec{v})-\left(\overrightarrow{e_i} \cdot \vec{v}\right)^2}$ for an orthonormal basis $\left\{\overrightarrow{e_1}, \overrightarrow{e_2}, \ldots, \overrightarrow{e_n}\right\}$ and a direction vector $\vec{v}=\overrightarrow{e_n}$. The Ricci curvature is the sum of all scalar curvatures in every basis vector direction $\sum_{i=1}^D K\left(\overrightarrow{e_i}, \vec{v}\right)$.
  * The Ricci curvature (left) can also be computed using the Ricci tensor $R_{i j}$ with: $\operatorname{Ric}(\vec{v}, \vec{v})=R_{i j} v^i v^j$
* There are two geometrical approaches in understanding Ricci-Curvature-Tensor:
  * <u>**Sectional Curvature**</u> for an orthonormal base (with Levi Civita symbol). The Ricci-Curvature-Tensor is zero: $\operatorname{Ric}(\vec{v}, \vec{v})=\sum_{i=1}^D K\left(\overrightarrow{e_i}, \vec{v}\right)=0$ for following **Sectional curvatures** (whose sum is the Ricci curvature): $K\left(\overrightarrow{e_1}, \vec{v}\right)=0 \quad K\left(\overrightarrow{e_2}, \vec{v}\right)=0$. General Sectional curvatures: $K(\vec{s}, \vec{v})=\frac{[R(\vec{s}, \vec{v}) \vec{v}] \cdot \vec{s}}{(\vec{s} \cdot \vec{s})(\vec{v} \cdot \vec{v})-(\vec{s} \cdot \vec{v})^2}$
  * <u>**Volume Element Derivative**:</u> for any base!
  * Achtung: Ricci Curvature-Tensor can only tell us about volume, not shape! because if $K\left(\overrightarrow{e_1}, \vec{v}\right)>0 \quad K\left(\overrightarrow{e_2}, \vec{v}\right) < 0$, then the overall Ricci tensor is the same: $\operatorname{Ric}(\vec{v}, \vec{v})=\sum_{i=1}^D K\left(\overrightarrow{e_i}, \vec{v}\right)=0$, only the shape changed.
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
