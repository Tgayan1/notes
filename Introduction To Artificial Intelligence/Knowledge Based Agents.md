### 1. Knowledge and Knowledge Base 

**Knowledge** : What AI agents know about the world/Environment
**Knowledge Base** : The collection of Knowledge

### 2. Knowledge Representation

#### 1. Propositional Logics

- **Proposition** : fact about a world which is either true or false
- **Sentence** : expression describing the world. consists with one or more propositions
- **Entailment** : a relation that means that if all the information in α is true, then all the information in β is true.$$\alpha \Vdash \beta$$
- **Inferencing** : deriving new sentences using existing KB $$\frac{if \ this \  is \ true (or \ in \ KB)}{can \ derive}$$
- **Inferencing Rules** :
	1. Modus Phonons$$\frac{\alpha \rightarrow \beta ,\alpha}{\beta}$$
	2. And Elimination$$\frac{\alpha \wedge \beta}{\alpha} \ \  or \ \ \frac{\alpha \wedge \beta}{\beta}$$
	3. Implication Elimination$$\frac{\alpha \rightarrow \beta}{\lnot\alpha \vee \beta}$$
	4. Biconditional Elimination$$\frac{\alpha \leftrightarrow \beta}{(\alpha \rightarrow \beta) \wedge (\beta \rightarrow \alpha)}$$
	5. Distributive $$\frac{\alpha \wedge(\beta \vee \gamma)}{(\alpha \wedge\beta)\vee(\alpha \wedge\gamma)} \ \ or \ \ \frac{\alpha \vee(\beta \wedge \gamma)}{(\alpha \vee\beta)\wedge(\alpha \vee\gamma)} $$
	6. De Morgan Laws$$\frac{\lnot(\alpha \vee \beta)}{\lnot\alpha\wedge\lnot\beta} \ \ or \ \ \frac{\lnot(\alpha \wedge \beta)}{\lnot\alpha\vee\lnot\beta}$$
	7. Resolution$$\frac{(\alpha \vee \beta_1 \vee \beta_2...\vee\beta_n)\wedge\lnot \alpha}{\beta_1 \vee \beta_2...\beta_n}$$
