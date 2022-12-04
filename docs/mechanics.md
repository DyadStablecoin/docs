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
We define a position as active if DYAD is made available to the protocol in form of a deposit into the damping vault hence newly generated DYAD is distributed according to the $\nu$-liquidity providers.
In order to understand the process of generating new DYAD, we want to exemplify this process.
In this example, the collateral value is to increase by 10% while only 60% of $\Gamma$ is deposited in the damping vault.

Since we need to map the collateral increase directly to the active DYAD positions and not all of $\Gamma$ is available to the protocol, the percentage increase for the $\nu$-liquidity providers must be higher than the percentage increase of the collateral.
Thus, with an averaged increase in positions, an additional $100/60\% = 16.66\%$ DYAD would be added to each liquidity provider’s balance.
We have intentionally decided against an averaged increase and have designed a mechanism to introduce more competition between liquidity providers.

Before diving into this mechanism, however, we first need to define how to participate as a liquidity provider in the DYAD protocol.
In order to provide liquidity, a DYAD non-fungible token (dNFT) must be purchased, from which DYAD stable coins can be withdrawn as an ERC-20 token.
The already named mechanism for the distribution of newly minted DYAD is based on experience points ($XP$), which reflects an intrinsic property of a dNFT.

In general, $XP$ are directly correlated with the active participation of a dNFT within the protocol.
If there is a loss in collateral value (negative delta), funds must be burned so that a rebalance between collateral and DYAD can occur to hold the peg of the stable asset.
As this process decreases a dNFT's active position, a certain amount of $XP$ is generated as an incentive for each DYAD burnt.
Since a dNFT with a high $XP$ value has already helped out many times by rebalancing the peg, we see the distribution of newly minted DYAD based on the $XP$ value as an incentive to actively participate in the protocol and thus to reward loyal participants.

The distribution of new DYAD is based on two criteria:

- Relative $XP$ position of dNFT with respect to all other dNFTs.
- DYAD deposit in the damping vault relative to the minted amount using a dNFT's balance sheet.

In relation to these points, we have developed a non-linear curve that helps us distributing newly minted DYAD to participants that fulfill the two criteria presented.
The distribution function is defined for dNFT $j$ as follows

$$
\Theta_j = \Omega_j \times \dfrac{\nu_j}{\kappa_j},
$$

where $\Omega_j$ defines the $XP$ function for dNFT $j$, $\nu_j$ is the active DYAD deposit of dNFT $j$ to the damping vault, and $\kappa_j$ is the amount of minted DYAD for dNFT $j$.
Note that $\nu_j = \kappa_j$ if no DYAD has been withdrawn and $\nu_j < \kappa_j$ if DYAD has been withdrawn.

## The $XP$ function

The $XP$ function is the most important component in the protocol.
As already mentioned, the $XP$ counter of a dNFT represents activity within the protocol
Active dNFTs are thus to be preferred while minting new DYAD (positive price deltas).
The form of this function thus needs to generate higher multipliers for higher $XP$ counters and $\textit{vice versa}$.

When syncing over all $N$ dNFTs, we save the maximum ($XP_{max}$)

$$
    XP_{max} = max({\text{dNFT}^{XP}_1, \dots, \text{dNFT}^{XP}_N}),
$$

and the minimum $XP$ ($XP_{min}$)

$$
    XP_{min} = min({\text{dNFT}^{XP}_1, \dots, \text{dNFT}^{XP}_N}),
$$

in terms of $XP$ counter.
To ensure the comparison with other dNFTs, we normalise $XP$ counts by a minimum-maximum standardization technique.
We shift all $XP$ counters by the minimum and divide by the range obtained from the difference of the maximum and the minimum.

Below, we exemplify this process for dNFT $j$ and obtain the scaled $XP$ count that is in the boundaries $\in [0, 1]$

$$
    XP^{scaled}_j = \frac{XP_j - XP_{min}}{XP_{max} - XP_{min} + 1}.
$$

Note that we increase the denominator by one to circumvent division by zero in the case that $XP_{max} = XP_{min}$.
The scaled $XP$ counter is used as a parameter for the $XP$ function.
This function calculates multipliers of $\approx 0.5$ for dNFTs with a low scaled $XP$ counter and a multiplier of $\approx 2.5$ for dNFTs with a high scaled $XP$ counter.

We have created a tangent hyperbolic function to model exactly these properties (fixed limits to those multipliers)

$$
\Omega_j = \alpha + tanh(\beta (XP^{scaled}_j  - \gamma))
$$

with $\alpha = 1.5$ (shift on the multiplier axis), $\beta = 11.0$ (slope of the function, switch between 0.5 and 2.5 multiplier), $\gamma = 0.85$ (shift on the $XP$ axis).
This function enables us to obtain multiplier of $\approx 0.5$ for low-$XP$ lying dNFT and $\approx 2.5$ for high-$XP$ lying dNFTs.
The shift on the $XP$ axis enables us to define when we have a one-to-one exposure, which is achieved around the 80$^{\text{th}}$ position (0.80) of the range.

## Minting distribution

In order to distribute minted DYAD to all active positions, we calculate $\Theta_j$-values for every dNFT and normalise each value by the sync-specific norm ($\omega_p$)

$$
\omega_p = \sum\limits_{j=1}^N\,\Theta_j,
$$

to finally obtain the amount of DYAD that a dNFT obtains within sync $p$.

Let us exemplify the logic with ten dNFTs having random properties.
Here, $XP$ is the actual $XP$-counter and we simplify the example by keeping the fraction between deposit and withdrawn for all dNFTs as one ($\nu_j / \kappa_j = 1.0$).

| Property      | 1     |   2   | 3     | 4     | 5     | 6     |   7   | 8     | 9     | 10    |
| ------------- | ----- | :---: | ----- | ----- | ----- | ----- | :---: | ----- | ----- | ----- |
| $XP$          | 2,161 | 7,588 | 3,892 | 3,350 | 3,012 | 5,496 | 8,048 | 7,333 | 3,435 | 1,079 |
| $XP^{scaled}$ | 0.16  | 0.93  | 0.40  | 0.33  | 0.28  | 0.63  | 1.00  | 0.90  | 0.34  | 0.00  |
| $\Theta$      | 0.50  | 2.23  | 0.50  | 0.50  | 0.50  | 0.52  | 2.43  | 1.98  | 0.50  | 0.50  |

The norm within sync $p$ is calculated as

$$
\omega_p = 7 \times (0.5 \times 1.00) + (2.2 \times 1.00) + (2.4 \times 1.00) + (2.0 \times 1.00) = 10.2,
$$

and hence we calculate the contribution for dNFT(7), with an $XP$ multiplier of 2.43 and a deposit/minted multiplier of 1.00 as follows.

First, we calculate the product of both multipliers

$$
z = \Theta_7 \times \dfrac{\nu_7}{\kappa_7} = 2.43 \times 1.00 = 2.43,
$$

and then we normalise the multiplier product

$$
\varepsilon_7^p = \dfrac{z}{\omega_p} = \dfrac{2.43}{10.2} = 0.24.
$$

Finally, we take the minted amount ($M$) and multiply the normalised contribution to obtain the minting allocation ($\zeta_7^p$) for dNFT(7) in sync $p$

$$
\zeta_7^p = \varepsilon_7^p \times M = 0.24 \times 10,000\,\text{DYAD} \approx 2400\,\text{DYAD}
$$

The table below shows minting allocations for a delta of $+10\%$ that represents a wanted mint value of 10,000 DYAD:

| Property  | 1    |   2   | 3    | 4    | 5    | 6    |   7   | 8     | 9    | 10   |
| --------- | ---- | :---: | ---- | ---- | ---- | ---- | :---: | ----- | ---- | ---- |
| $\zeta^p$ | 492  | 2,194 | 493  | 493  | 493  | 510  | 2,392 | 1,949 | 493  | 493  |
| exposure  | 0.49 | 2.19  | 0.49 | 0.49 | 0.49 | 0.51 | 2.39  | 1.95  | 0.49 | 0.49 |

Note that the average mint would be 1,000 DYAD.
All low-$XP$ lying dNFTs have an exposure of $\approx 0.5$ while the high-$XP$ lying dNFTs have an exposure of $\approx 2.5$.

## Burning

tba

## XP accrual

tba

## Liquidiations

tba
