# Triplet Tweak (Crypto)

## Challenge Description
A paranoid archivist locked the same message three times using three different RSA public keys, but carelessly shared a single hidden master private exponent $d$ across all three keys. 

## Vulnerability & Analysis
1. **Shared Private Exponent (Common Private Exponent)**:
   The challenge provides three RSA public keys $(n_1, e_1), (n_2, e_2), (n_3, e_3)$ and three ciphertexts $c_1, c_2, c_3$ encrypting the same message. All three keys share the same private key exponent $d$.
   
2. **Mathematical Vulnerability**:
   For each RSA instance $i \in \{1, 2, 3\}$, we have the key relation:
   $$e_i \cdot d - k_i \cdot \phi(n_i) = 1$$
   Since $\phi(n_i) = n_i - p_i - q_i + 1$, we can approximate this as:
   $$e_i \cdot d - k_i \cdot n_i = 1 - k_i(p_i + q_i - 1) = \epsilon_i$$
   where $\epsilon_i \approx -k_i(p_i + q_i) \approx -d \cdot 2^{512}$ (since $n_i$ is 1024 bits).

3. **Lattice Construction**:
   We can formulate a Shortest Vector Problem (SVP) in a lattice to find $d$. Let $X$ be a scaling factor around $\sqrt{n_i} \approx 2^{500}$.
   We define the lattice basis matrix:
   $$
   B = \begin{pmatrix}
   X & e_1 & e_2 & e_3 \\
   0 & -n_1 & 0 & 0 \\
   0 & 0 & -n_2 & 0 \\
   0 & 0 & 0 & -n_3
   \end{pmatrix}
   $$

   Consider the linear combination of rows:
   $$v = d \cdot \text{Row}_0 + k_1 \cdot \text{Row}_1 + k_2 \cdot \text{Row}_2 + k_3 \cdot \text{Row}_3$$
   $$v = (d \cdot X, e_1 d - k_1 n_1, e_2 d - k_2 n_2, e_3 d - k_3 n_3) = (d \cdot X, \epsilon_1, \epsilon_2, \epsilon_3)$$

   Since $X \approx 2^{500}$ and $\epsilon_i \approx -d(p_i+q_i) \approx -d \cdot 2^{512}$, all components of $v$ are of similar magnitude. This ensures that $v$ will be one of the shortest vectors in the lattice, which can be found using the Lenstra–Lenstra–Lovász (LLL) algorithm.

## Exploitation
1. **Lattice Reduction (LLL)**:
   We construct the lattice using the moduli and public exponents from `pub (1).txt` with $X = 2^{500}$ and reduce it using SymPy's LLL algorithm (`to_DM().lll().to_Matrix()`).
   
2. **Key Recovery**:
   From the reduced basis, we search for a row where the first element is a multiple of $X$. Dividing this element by $X$ yields the candidate private exponent:
   - Recovered $d = 1893819211103489343773099046624149417932128366381507045208886540210169694551492341730746515272509120631$

3. **Decryption**:
   Using the recovered private exponent $d$, we decrypt the ciphertext $c_1$ under modulus $n_1$:
   $$m \equiv c_1^d \pmod{n_1}$$
   Converting the integer $m$ back to bytes yields the decrypted flag.

## Flag
`athena{sh4r3d_d_3xp}`
