# Genetic_drift
A genetic drift simulator

# Migration Effect in Genetic Drift Simulation

The migration effect (also called gene flow) in this genetic drift simulation represents the movement of individuals between populations, which introduces new genetic material and affects allele frequencies.

## What Migration Represents

Migration is the transfer of genetic variation from one population to another. In this simulation:
- **Migration Rate**: The proportion of individuals that migrate into the population each generation
- **Migration A1 Frequency**: The frequency of the A1 allele in the migrant population

## How Migration Works in the Simulation Logic

The migration effect is implemented in the simulation with this logic:

```javascript
// Apply migration if enabled
if (migrationRate > 0 && Math.random() < migrationRate) {
    // Migrant alleles come from a source population with fixed A1 frequency
    currentP = (1 - migrationRate) * currentP + migrationRate * migrationA1;
}
```

This formula calculates the new allele frequency after migration using:
- `(1 - migrationRate) * currentP`: The proportion of alleles from the original population
- `migrationRate * migrationA1`: The proportion of alleles from migrants

## Biological Significance

Migration counteracts genetic drift by:
1. **Introducing new alleles** that might have been lost through drift
2. **Preventing fixation** of alleles by maintaining genetic diversity
3. **Homogenizing populations** that are connected by migration
4. **Rescuing small populations** from inbreeding depression

## Real-World Examples

- **Island populations** receiving migrants from the mainland
- **Wildlife corridors** allowing animal movement between fragmented habitats
- **Plant pollen** being carried by wind or pollinators between populations
- **Human populations** with intermarriage between different groups

The migration effect demonstrates how isolated populations can maintain genetic diversity and how connected populations tend to become more genetically similar over time.






### Core Concept: Fitness and Mean Fitness

The simulation assigns a **fitness value** (a measure of reproductive success) to each genotype (A1A1, A1A2, A2A2). Selection works by altering the probability that an allele is passed on based on these fitness values.

The key calculation is the **mean fitness of the population ($\bar{w}$)**. The frequency of alleles in the next generation is determined by their contribution to the current generation's fitness relative to this mean fitness.

The general formula for the frequency of A1 in the next generation ($p'$) is:
$p' = \frac{p^2 \cdot w_{11} + p \cdot q \cdot w_{12}}{\bar{w}}$
where:
*   $p$ = current frequency of A1
*   $q = 1 - p$ = current frequency of A2
*   $w_{11}$ = fitness of genotype A1A1
*   $w_{12}$ = fitness of genotype A1A2 (heterozygote)
*   $w_{22}$ = fitness of genotype A2A2
*   $\bar{w} = p^2 \cdot w_{11} + 2pq \cdot w_{12} + q^2 \cdot w_{22}$ (the mean fitness)

---

### 1. Directional Selection

**Goal:** To favor one allele over the other.

**How it's set in the UI:**
*   **Type:** "Directional (Favor A1)" or "Directional (Favor A2)"
*   **Strength:** A value between 0 and 1. This defines the *selective advantage*.

**Implementation Logic:**
The code simplifies the model by giving a fitness advantage to the favored homozygous genotype.

*   **For "Favor A1":**
    *   A1A1 fitness ($w_{11}$) = `1 + selectionStrength`
    *   A1A2 fitness ($w_{12}$) = 1
    *   A2A2 fitness ($w_{22}$) = 1
    *   The code then calculates the probability of selecting an A1 allele based on these fitness values. Because A1A1 individuals are more successful, they contribute more A1 alleles to the next generation, causing the frequency of A1 ($p$) to increase over time.

*   **For "Favor A2":**
    *   A1A1 fitness ($w_{11}$) = 1
    *   A1A2 fitness ($w_{12}$) = 1
    *   A2A2 fitness ($w_{22}$) = `1 + selectionStrength`
    *   This has the opposite effect, increasing the frequency of A2 ($q$) over time.

```javascript
// From the simulation code:
if (selectionType === 'directionalA1') {
    // Favor A1 allele: A1A1 has highest fitness
    selectionProbability = currentP * (1 + selectionStrength) / 
                          (currentP * (1 + selectionStrength) + (1 - currentP));
} else if (selectionType === 'directionalA2') {
    // Favor A2 allele: A2A2 has highest fitness
    selectionProbability = currentP / 
                          (currentP + (1 - currentP) * (1 + selectionStrength));
}
```

---

### 2. Stabilizing Selection

**Goal:** To favor intermediate phenotypes (heterozygotes) and select against both extremes.

**How it's set in the UI:**
*   **Type:** "Stabilizing"
*   **Strength:** A value between 0 and 1. This defines the *disadvantage* for the homozygous genotypes.

**Implementation Logic:**
This is classic **heterozygote advantage** or **overdominance**.

*   A1A1 fitness ($w_{11}$) = `1 - selectionStrength`
*   A1A2 fitness ($w_{12}$) = 1 (highest fitness)
*   A2A2 fitness ($w_{22}$) = `1 - selectionStrength`

The calculation uses the full formula for $p'$ shown above. The heterozygotes (A1A2) have the highest fitness, so they contribute disproportionately to the next generation. This maintains both alleles A1 and A2 in the population at a stable equilibrium and prevents fixation.

```javascript
// From the simulation code:
} else if (selectionType === 'stabilizing') {
    // Favor heterozygotes (overdominance)
    const fitnessAA = 1 - selectionStrength; // A1A1 fitness reduced
    const fitnessAa = 1;                     // A1A2 fitness is highest
    const fitnessaa = 1 - selectionStrength; // A2A2 fitness reduced

    const meanFitness = 
        currentP * currentP * fitnessAA + 
        2 * currentP * (1 - currentP) * fitnessAa + 
        (1 - currentP) * (1 - currentP) * fitnessaa;

    selectionProbability = (currentP * currentP * fitnessAA + 
                          currentP * (1 - currentP) * fitnessAa) / meanFitness;
}
```

---

### 3. Disruptive Selection

**Goal:** To favor both extreme phenotypes and select against intermediates.

**How it's set in the UI:**
*   **Type:** "Disruptive"
*   **Strength:** A value between 0 and 1. This defines the *advantage* for the homozygous genotypes.

**Implementation Logic:**
This is **heterozygote disadvantage** or **underdominance**.

*   A1A1 fitness ($w_{11}$) = `1 + selectionStrength`
*   A1A2 fitness ($w_{12}$) = 1 (lowest fitness)
*   A2A2 fitness ($w_{22}$) = `1 + selectionStrength`

The calculation again uses the full formula. The homozygotes are more fit than the heterozygotes. This creates an unstable equilibrium. If the frequency of A1 ($p$) is above 0.5, selection will drive it towards fixation (p=1). If it's below 0.5, selection will drive it towards loss (p=0). This can be a powerful mechanism for driving speciation.

```javascript
// From the simulation code:
} else if (selectionType === 'disruptive') {
    // Favor homozygotes (underdominance)
    const fitnessAA = 1 + selectionStrength; // A1A1 fitness increased
    const fitnessAa = 1;                     // A1A2 fitness is lowest
    const fitnessaa = 1 + selectionStrength; // A2A2 fitness increased

    const meanFitness = 
        currentP * currentP * fitnessAA + 
        2 * currentP * (1 - currentP) * fitnessAa + 
        (1 - currentP) * (1 - currentP) * fitnessaa;

    selectionProbability = (currentP * currentP * fitnessAA + 
                          currentP * (1 - currentP) * fitnessAa) / meanFitness;
}
```

### Summary Table

| Selection Type | Fitness A1A1 | Fitness A1A2 | Fitness A2A2 | Effect on Population |
| :--- | :--- | :--- | :--- | :--- |
| **Directional (Favor A1)** | **1 + s** | 1 | 1 | A1 increases to fixation |
| **Directional (Favor A2)** | 1 | 1 | **1 + s** | A2 increases to fixation |
| **Stabilizing** | **1 - s** | **1** | **1 - s** | **Maintains polymorphism** |
| **Disruptive** | **1 + s** | **1** | **1 + s** | **Drives fixation or loss** |

*Note: In the code, `s` represents the `selectionStrength` parameter.*

The simulation elegantly combines this selection logic with the random sampling of genetic drift, allowing you to see how the deterministic force of selection interacts with the random force of drift, especially in small populations.




In population genetics, **`s`** is the **selection coefficient**.

## What `s` Represents

The selection coefficient (`s`) is a measure of the **relative fitness difference** between genotypes. It quantifies the strength of natural selection acting on a specific genotype.

*   **`s` is always a value between 0 and 1.**
*   `s = 0` means **no selection** (the genotypes are equally fit).
*   `s = 1` means **complete selection** against a genotype (it has zero fitness and leaves no offspring).

## How `s` is Used in the Fitness Calculations

The fitness of a genotype is typically calculated as `1 - s` for a disfavored genotype or `1 + s` for a favored one, relative to a baseline fitness of 1.

Let's look at the examples from the simulation:

### 1. Directional Selection (Favor A1)
*   **Fitness of A1A1 (`w₁₁`) = `1 + s`**
    *   `s` is the **selective advantage** of the A1A1 homozygote.
    *   Example: If `s = 0.1`, A1A1 individuals have a 10% fitness advantage. On average, they produce 10% more offspring than other genotypes.
*   Fitness of A1A2 (`w₁₂`) = `1` (the baseline)
*   Fitness of A2A2 (`w₂₂`) = `1`

### 2. Directional Selection (Favor A2)
*   Fitness of A1A1 (`w₁₁`) = `1`
*   Fitness of A1A2 (`w₁₂`) = `1`
*   **Fitness of A2A2 (`w₂₂`) = `1 + s`**
    *   `s` is now the selective advantage of the A2A2 homozygote.

### 3. Stabilizing Selection (Heterozygote Advantage)
*   **Fitness of A1A1 (`w₁₁`) = `1 - s`**
    *   `s` is the **selective disadvantage** or "cost" of being a homozygote.
*   **Fitness of A1A2 (`w₁₂`) = `1`** (this is the most fit genotype)
*   **Fitness of A2A2 (`w₂₂`) = `1 - s`**
    *   Example: If `s = 0.3`, both homozygotes have a 30% fitness reduction compared to the heterozygote.

### 4. Disruptive Selection (Heterozygote Disadvantage)
*   **Fitness of A1A1 (`w₁₁`) = `1 + s`**
    *   `s` is the selective advantage of being a homozygote.
*   **Fitness of A1A2 (`w₁₂`) = `1`** (this is the least fit genotype)
*   **Fitness of A2A2 (`w₂₂`) = `1 + s`**
    *   Example: If `s = 0.2`, both homozygotes have a 20% fitness advantage over the heterozygote.

## In the Context of the Simulation

In the HTML/JS code you provided, the `selectionStrength` parameter that you set in the user interface **is the selection coefficient `s`**.

```javascript
const selectionStrength = parseFloat(selectionStrengthInput.value); // This value is 's'
```

So, when you set "Selection Strength" to 0.1, you are setting `s = 0.1`.

## Biological Interpretation

The value of `s` has real-world meaning:
*   **`s < 0.001`**: Very weak selection, often overwhelmed by genetic drift.
*   **`s ~ 0.01`**: Moderate selection. Observable over many generations.
*   **`s > 0.1`**: Strong selection. Can cause rapid evolutionary change.
*   **`s = 1`**: Lethal selection (e.g., a genetic disease that prevents reproduction).

This is why the simulation allows you to play with the `s` value and population size (N). You can directly observe the classic rule of thumb: **Genetic drift overwhelms selection when `s < 1/(2N)`**. In a small population, even a strongly disadvantageous allele (`s = 0.1`) can sometimes fix by chance.




