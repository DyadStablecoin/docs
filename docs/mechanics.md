---
sidebar_position: 5
---

# Mechanics

Let us define the key mechanics of the protocol:

- Minting
- Burning
- Accrual
- Legend

# Minting

The creation of DYAD is defined by pre-defined minting equations whenever the
underlying collateral (ETH) increases in value.
The total amount of DYAD ($\Gamma$) is obtained by adding deposited DYAD that is available in
the damping vault ($dD$) to DYAD that is withdrawn (DYAD)

$$
\Gamma = dD + \text{DYAD}.
$$

In order to understand the process of generating new DYAD, we want to exemplify this process.
In this example, the collateral value is to increase by 10% while only 60% of $\Gamma$ is deposited in the damping vault.

Since we need to map the collateral increase directly to the dD positions, the percentage increase for the dD-liquidity providers must be higher than the percentage increase of the collateral.
Thus, with an averaged increase in positions, an additional $100/60\% = 16.66\%$ DYAD would be added to each liquidity provider’s metadata.
We have intentionally decided against an averaged increase and have designed a mechanism to introduce more competition between liquidity providers.

Before diving into this mechanism, however, we first need to define how to participate as a liquidity provider in the DYAD protocol.
In order to provide liquidity, a DYAD non-fungible token (dNFT) must be purchased, from which DYAD can be withdrawn as an ERC-20 token.
The already named mechanism for the distribution of newly minted DYAD is based on experience points ($XP$), which reflects an intrinsic property of a dNFT.

In general, $XP$ are directly correlated with the active participation of a dNFT within the protocol.
If there is a loss in collateral value (negative delta), dD must be burned so that a rebalance with respect to the collateral can occur.
As this process decreases a dNFT's dD position, a certain amount of $XP$ is generated as an incentive for each dD burnt.
Since a dNFT with a high $XP$ value has already showed high activity in the protocol, we see the distribution of newly minted dD based on the $XP$ value as an incentive to actively participate in the protocol and thus to reward loyal participants.

The distribution of dD is based on two criteria:

- Relative $XP$ position of dNFT with respect to all other dNFTs.
- dD deposit in the damping vault.

In relation to these points, we have developed a function that helps us distributing newly minted dD to participants that fulfill the two criteria presented.
The distribution function is defined for dNFT $j$ as follows

$$
\Theta_j = \frac{1}{2}\left(\frac{XP_j}{\sum\limits_i^N\,XP_i} + \frac{dD_j}{\sum\limits_i^N\,dD_i}\right),
$$

where $\Omega_j$ defines the $XP_j$ function for dNFT $j$ and $dD_j$ is the deposit of dNFT $j$ to the damping vault.
Note that the sum of all $XP$ and $dD$ fractions will add up to unity and hence we normalize by two.
In order to distribute $dD$ to all active positions, we calculate $\Theta_j$-values for every dNFT.

## Minting distribution

Let us exemplify the logic with ten dNFTs having random properties.
Here, $XP$ is the actual $XP$-counter and $dD$ is the deposit to the damping vault.
For dNFT(7), we take the minted amount ($M = 10,238\,dD$) and multiply the normalised contribution to obtain the minting allocation ($\zeta_7$)

$$
\zeta_7 = \Theta_7 \times M = 0.14 \times 10,238\,dD \approx 1,419\,dD
$$

| Property       | 1      |   2    | 3      | 4      | 5      | 6      |   7    | 8      | 9      | 10     |
| -------------- | ------ | :----: | ------ | ------ | ------ | ------ | :----: | ------ | ------ | ------ |
| $XP$           | 2,161  | 7,588  | 3,892  | 3,350  | 3,012  | 5,496  | 8,048  | 7,333  | 3,435  | 1,079  |
| $dD$           | 10,000 | 10,000 | 10,000 | 10,000 | 10,000 | 10,000 | 10,000 | 10,000 | 10,000 | 10,000 |
| DYAD           | 0,31   |  0,10  | 0,20   | 0,00   | 0,12   | 0,05   |  0,67  | 0,80   | 0,12   | 0,01   |
| $\Theta$ in %  | 7      |   13   | 9      | 9      | 8      | 11     |   14   | 13     | 9      | 6      |
| Minted in $dD$ | 0,756  | 1,368  | 0,951  | 0,890  | 0,852  | 1,132  | 1,419  | 1,339  | 0,899  | 0,634  |
| Exposure       | 0.7    |  1.3   | 0.9    | 0.9    | 0.8    | 1.1    |  1.4   | 1.3    | 0.9    | 0.6    |

Note that the average mint would be 1,024 $dD$.

We will subsequently discuss the burning of DYAD with negative deltas.

## Burning

In the case of negative deltas against the collateral, $dD$ present in the damping vault must be burned.
Once again, the properties of a dNFT play a decisive role in determining the burning allocation for each participant.

Two criteria determine the respective amount that each dNFT must burn in the event of a negative delta:

- Relative $XP$ position of dNFT with respect to all other dNFTs.
- Amount of $dD$ and DYAD.

We want to get the inverse effect compared to minting allocations where high $XP$ values burn less $dD$ and $\textit{vice versa}$.
We therefore scale the $XP$-value of dNFT $j$ by the maximum $XP$-value available to shift it into a range $\in$ [0, 1]

$$
XP_{j,s} = \frac{XP_j}{XP_{max}}.
$$

We then calculate the scaling factor by dividing $XP_{j,s}$ by the relative $XP$-count with respect to the total $XP$

$$
\alpha = \frac{XP_{j,s}}{\frac{XP_j}{\sum\limits_i^N\,XP_i}} = \frac{\frac{XP_j}{XP_{max}}}{\frac{XP_j}{\sum\limits_i^N\,XP_i}} =\frac{\sum\limits_i^N\,XP_i}{XP_{max}}.
$$

We create then the inverse by subtracting $XP_{j,s}$ from one

$$
\hat{XP}_{j,s} = 1 - XP_{j,s}.
$$

The norm is obtained by dividing $\hat{XP}_{j,s}$ by the dNFT count subtracted by the scaling factor

$$
XP_{j,s}^n = \frac{\hat{XP_{j,s}}}{N - \alpha}.
$$

The second dependency for burning allocations is dependent on $dD$ and DYAD count. Here, we sum both counts together and divide by the total sum over all dNFTs

$$
\omega_j = \frac{dD_j + \text{DYAD}_j}{\sum\limits_i^N\,(dD_i + \text{DYAD}_i)}.
$$

The burning distribution function is defined for dNFT $j$ as follows

$$
\Omega_j = \frac{1}{2}\left( XP_{j,s}^n + \omega_j \right).
$$

Burning allocations are determined by multiplying the burining distribution function by the total amount that needs to be burnt ($B$)

$$
\mu_j = \Omega_j B
$$

The distribution of burn allocations follows the same principle as described above and hence we do not repeat the procedure for burning.

## $XP$ accrual

A dNFT receives new $XP$ points with each DYAD that is burned at a negative delta to keep the peg of the stable asset intact.
Again, we take the $XP$ position to determine the accrual in $XP$.
This equation has a direct proportionality to the value of DYAD burnt ($\mu$), and is scaled by the the scaled $XP$-value

$$
\Alpha_j = \dfrac{\mu_j}{XP_{j,s} + 0.05}.
$$

Note that we include 0.05 in the denominator to set an accrual limit for dNFTs with very low scaled $XP$ counts.

# Legend

| Symbol   | Definition                    |
| -------- | ----------------------------- |
| $XP$     | Experience points of a dNFT   |
| $dD$     | dD in damping vault           |
| DYAD     | ERC-20 DYAD (withdrawn)       |
| $\Theta$ | Minting distribution function |
| $\Omega$ | Burning distribution function |
| $\Alpha$ | $XP$ accrual                  |
