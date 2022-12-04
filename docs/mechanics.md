---
sidebar_position: 5
---

# Mechanics

Let us define the key mechanics of the protocol:

- Minting (positive delta)
- Burning (negative delta)
- $XP$ accrual
- Liquidations

# Miniting

The creation of DYAD is defined by pre-defined minting equations whenever the
underlying collateral (ETH, abbreviated as Ξ in the following) increases in value.
The total amount of DYAD ($\Gamma$) is obtained by adding DYAD that is available in
the damping vault ($\nu$) to DYAD that is withdrawn ($\kappa$)

$$
\Gamma = \nu + \kappa.
$$

Minting allocations are assigned exclusively to active positions.
We define a position as active when DYAD is made available to the protocol by a deposit into the damping vault hence newly generated DYAD is distributed according to the $\nu$-liquidity providers.
In order to understand the process of generating new DYAD, we want to exemplify this process through an example. In this example, the collateral value is to increase by 10% while only 60% of $\Gamma$ is deposited in the damping vault.

Since we need to map the collateral increase directly to the DYAD positions and not all of $\Gamma$ is available to the protocol to generate DYAD, the percentage increase for the liquidity providers must be higher than the percentage increase of the collateral.
hus, with an averaged increase in positions, an additional $100/60\% = 16.66\%$ DYAD would be added to each liquidity provider’s position.

We have decided against an averaged increase and have designed a mechanism to introduce more competition between liquidity providers.
Before introducing this mechanism, however, we first need to define how to participate as a liquidity provider in the DYAD protocol.
In order to provide liquidity, a DYAD non-fungible token (dNFT) must first be purchased, from which minted DYAD can be withdrawn as an ERC-20 token.
The already named mechanism for the distribution of newly created DYAD is based on experience points ($XP$), which reflects an intrinsic property of a dNFT.

In general, $XP$ are directly correlated with the active participation of a dNFT within the DYAD protocol.
If there is a loss in collateral value, DYAD must be burned so that a rebalance between collateral and DYAD can occur.
As this process decreases the active DYAD position of a dNFT, $XP$ is generated as an incentive for each DYAD burnt.
Since a dNFT with a high $XP$ value has already helped the protocol many times by rebalancing the DYAD peg, we see the distribution of newly created DYAD based on the $XP$ value as an incentive to actively participate in the protocol and thus to reward loyal participants.
The distribution of new DYAD is based on two different points:

- Relative $XP$ position of dNFT with respect to all other dNFTs.
- DYAD deposit in the damping vault relative to a minted DYAD using a dNFT's balance sheet.

In relation to these points, we have developed a distribution function that distributes newly created DYAD to participants that fulfill the two criteria presented.
The distribution function is defined for dNFT $j$ as follows

$$
\Theta_j = \Omega_j \times \dfrac{\nu_j}{\kappa_j},
$$

where $\Omega_j$ defines the $XP$ distribution function for dNFT $j$, $\nu_j$ is the active DYAD deposit of dNFT $j$ to the damping vault, and $\kappa_j$ is the total amount of minted DYAD for dNFT $j$.

## The $XP$ function

The $XP$ function is the most important component in our protocol.
As already mentioned, the $XP$ counter of a dNFT represents the activity within the protocol.
Active dNFTs are thus to be preferred when minting new DYAD for positive price deltas with respect to the collateral.
The functional form of this function thus needs to generate a higher multiplier if the $XP$ counter of a dNFT is relatively high and $\textit{vice versa}$.
To ensure the comparison with other dNFTs, we normalise the $XP$ count of each dNFT by a minimum-maximum standardization technique.

When syncing over all $N$ dNFTs, we save the maximum ($XP_{max}$)

$$
    XP_{max} = max({\text{dNFT}^{XP}_1, \dots, \text{dNFT}^{XP}_N}),
$$

and the minimum $XP$ ($XP_{min}$)

$$
    XP_{min} = min({\text{dNFT}^{XP}_1, \dots, \text{dNFT}^{XP}_N}),
$$

in terms of $XP$ counter.

We shift all $XP$ counters by the minimum and divide by the range obtained from the difference of the maximum and the minimum.
Below, we exemplify this process for dNFT $j$ and obtain the scaled $XP$ count that is in the boundaries $\in [0, 1]$

$$
    XP^{scaled}_j = \frac{XP_j - XP_{min}}{XP_{max} - XP_{min} + 1}.
$$

Note that we increase the denominator by one to circumvent division by zero in the case of $XP_{max} = XP_{min}$.
The scaled $XP$ counter is used as a parameter for the $XP$ function.
This function shall calculate a multiplier of $\approx 0.5$ for dNFTs with a low scaled $XP$ counter and a multiplier of $\approx 2.5$ for dNFTs with a high scaled $XP$ counter.

We have created a tangent hyperbolic distribution function to model exactly these properties

$$
\Omega_j = \alpha + tanh(\beta (XP^{scaled}_j  - \gamma))
$$

with $\alpha = 1.5$ (shift on the multiplier axis), $\beta = 11.0$ (slope of the function), $\gamma = 0.85$ (shift on the $XP$ axis).
This function enables us to obtain multiplier of $\approx 0.5$ for low-$XP$ lying dNFT and $\approx 2.5$ for high-$XP$ lying dNFTs.

## Minting distribution

In order to distribute minted DYAD to all active positions, we calculate $\Theta_j$-values for every dNFT and normalise each value by the sync-specific norm ($\omega_p$)

$$
\omega_p = \sum\limits_{j=1}^N\,\Theta_j,
$$

to finally obtain the amount of DYAD that a dNFT obtains within sync $p$.

Let us exemplify the logic with ten dNFTs having example properties.
Here, $XP$ is the actual $XP$-counter and DY is the active DYAD deposit and we simplify the example by keeping the fraction between deposit and withdrawn for all dNFTs as one.

| Property      | 1     |   2   | 3     | 4     | 5     | 6     |   7   | 8     | 9     | 10    |
| ------------- | ----- | :---: | ----- | ----- | ----- | ----- | :---: | ----- | ----- | ----- |
| $XP$          | 2,161 | 7,588 | 3,892 | 3,350 | 3,012 | 5,496 | 8,048 | 7,333 | 3,435 | 1,079 |
| $XP^{scaled}$ | 0.16  | 0.93  | 0.40  | 0.33  | 0.28  | 0.63  | 1.00  | 0.90  | 0.34  | 0.00  |
| $\Theta$      | 0.50  | 2.23  | 0.50  | 0.50  | 0.50  | 0.52  | 2.43  | 1.98  | 0.50  | 0.50  |

The norm within sync $p$ is calculated as

$$
\omega_p = 7 \times 0.5 + 2.2 + 2.4 + 2.0 = 10.2.
$$

For a delta ($+10\%$) that represents a wanted mint value of 10,000 DYAD, minting allocations ($\zeta$) are given as follows:

| Property | 1    |   2   | 3    | 4    | 5    | 6    |   7   | 8     | 9    | 10   |
| -------- | ---- | :---: | ---- | ---- | ---- | ---- | :---: | ----- | ---- | ---- |
| $\zeta$  | 492  | 2,194 | 493  | 493  | 493  | 510  | 2,392 | 1,949 | 493  | 493  |
| exposure | 0.49 |  2.2  | 0.49 | 0.49 | 0.49 | 0.51 | 2.39  | 1.95  | 0.49 | 0.49 |

Note that the average mint would be 1,000 DYAD.
All low-$XP$ lying dNFTs have an exposure of $\approx 0.5$ while the high-$XP$ lying dNFTs have an exposure of $\approx 2.5$.

## Burning

tba

## XP accrual

tba

## Liquidiations

tba
