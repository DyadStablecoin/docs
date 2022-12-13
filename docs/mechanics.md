---
sidebar_position: 5
---

# Mechanics

Let us define the key mechanics of the protocol:

- Minting
- Burning
- $XP$ accrual
- $XP$ boost
- Legend

# Minting

The creation of DYAD is defined by pre-defined minting equations whenever the
underlying collateral (ETH) increases in value.
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

In order to distribute minted DYAD to all active positions, we calculate $\Theta_j$-values for every dNFT and normalise each value by the sync-specific mint norm ($\omega^p_m$)

$$
\omega^p_m = \sum\limits_{j=1}^N\,\Theta_j,
$$

to finally obtain the amount of DYAD that a dNFT obtains within sync $p$.

Let us exemplify the logic with ten dNFTs having random properties.
Here, $XP$ is the actual $XP$-counter and we simplify the example by keeping the fraction between deposit and withdrawn for all dNFTs as one ($\nu_j / \kappa_j = 1.0$).

| Property      | 1     |   2   | 3     | 4     | 5     | 6     |   7   | 8     | 9     | 10    |
| ------------- | ----- | :---: | ----- | ----- | ----- | ----- | :---: | ----- | ----- | ----- |
| $XP$          | 2,161 | 7,588 | 3,892 | 3,350 | 3,012 | 5,496 | 8,048 | 7,333 | 3,435 | 1,079 |
| $XP^{scaled}$ | 0.16  | 0.93  | 0.40  | 0.33  | 0.28  | 0.63  | 1.00  | 0.90  | 0.34  | 0.00  |
| $\Theta$      | 0.50  | 2.23  | 0.50  | 0.50  | 0.50  | 0.52  | 2.43  | 1.98  | 0.50  | 0.50  |

The mint norm within sync $p$ is calculated as

$$
\omega^p_m = 7 \times (0.5 \times 1.0) + (2.2 \times 1.0) + (2.4 \times 1.0) + (2.0 \times 1.0) = 10.2,
$$

and hence we calculate the contribution for dNFT(7), with an $XP$ multiplier of 2.43 and a deposit/minted multiplier of 1.00 as follows.

First, we calculate the product of both multipliers

$$
z_7 = \Theta_7 \times \dfrac{\nu_7}{\kappa_7} = 2.43 \times 1.00 = 2.43,
$$

and then we normalise the multiplier product

$$
\varepsilon_7^p = \dfrac{z}{\omega^p_m} = \dfrac{2.43}{10.2} = 0.24.
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
After describing the minting of DYAD with positive deltas against the collateral, we will subsequently discuss the burning of DYAD with negative deltas.

## Burning

In the case of negative deltas against the collateral, DYAD present in the damping vault must be burned in order to maintain the peg of our stable asset.
Once again, the properties of a dNFT play a decisive role in determining the burning allocation for each participant.

Two criteria determine the respective amount that each dNFT must burn in the event of a negative delta:

- Relative $XP$ position of dNFT with respect to all other dNFTs.
- Amount of minted DYAD relative to average minted amount of DYAD.

We use the $XP$ position of the dNFT as input for the inverse $XP$ function.
This function calculates multipliers of $\approx 2.5$ for a dNFT with a low scaled $XP$ value and $\approx 0.5$ for a dNFT with a high scaled $XP$ value.
We define the inverse $XP$ function as follows for dNFT $j$

$$
\Omega^{-1}_j = \lambda - \Omega_j,
$$

where $\lambda = max(\Omega) + min(\Omega)$ is a constant value.

Furthermore, we calculate the average minted DYAD value $\Zeta$ by dividing the total amount of minted DYAD ($\Gamma$) by the number of dNFTs $N$

$$
\Zeta = \dfrac{\Gamma}{N}.
$$

Again, we calculate allocations using a distribution function consisting of the product of the inverse $XP$ and the fraction of the dNFT's minted DYAD value ($m$) relative to the average minted DYAD, which is for dNFT $j$ defined as

$$
\Epsilon_j = \Omega^{-1}_j \times \delta_j.
$$

where we use the limit $\chi = 2.0$ to define $\delta_j$

$$
\delta_{j} =
    \begin{cases}
        \dfrac{m_j}{\Zeta} & \text{if}\quad  \dfrac{m_j}{\Zeta} < \chi,\\
        \chi & \text{ otherwise.}
    \end{cases}
$$

Note that we limit the burning allocations for the highest $XP$ position to $0.5 \times 2.0 = 1.0$.

We follow the same strategy as for minting and obtain a burn norm for sync $p$

$$
\omega^p_b = \sum\limits_{j=1}^N\,\Epsilon_j.
$$

The distribution of burn allocations follows the same principle as described above and hence we do not repeat the procedure for burning.

## $XP$ accrual

A dNFT receives new $XP$ points with each DYAD that is burned at a negative delta to keep the peg of the stable asset intact.
Again, we take the $XP$ position to determine the accrual in $XP$.
This equation has a direct proportionality to the value of DYAD burnt ($\mu$), and is scaled by the $XP$ function

$$
XP^{accrual}_j = \dfrac{\mu_j}{\Omega_j}.
$$

For a $\mu$-value of 1000, a low-positioned dNFT ($\Omega \approx 0.5$) thus has an $XP$ accrual of 2000, while a high-positioned dNFT ($\Omega \approx 2.5$) only has one of 400.

## $XP$ boost

When a dNFT triggers the Sync method, its $XP$ is incremented at the triggered Sync.
As an example, let's give a dNFT that actually has an $XP$ increment of 2000.
By invoking the sync method, this dNFT can receive an additional $XP$ boost and double its $XP$ accrual to overall 4000 points.
This is especially interesting for dNFTs that already have a high $XP$ position and hence helps to keep a high $XP$ position.

## $XP$ boost

# Legend

| Symbol        | Definition                             |
| ------------- | -------------------------------------- |
| $XP$          | Experience points of a dNFT            |
| $XP_{max}$    | Maximum of experience points           |
| $XP_{min}$    | Minimum of experience points           |
| $XP^{scaled}$ | Min-max scaled experience points       |
| $\Gamma$      | Total amount of minted DYAD            |
| $\nu$         | DYAD in damping vault                  |
| $\kappa$      | DYAD in the wild (withdrawn)           |
| $\Theta$      | Minting distribution function          |
| $\Omega$      | $XP$ function                          |
| $\omega_m$    | Mint norm                              |
| $z$           | Mint multiplier product                |
| $\varepsilon$ | Normalised mint multiplier product     |
| $\zeta$       | Mint allocation                        |
| $\Omega^{-1}$ | Inverse $XP$ function                  |
| $\lambda$     | Sum of $max(\Omega)$ and $min(\Omega)$ |
| $\Zeta$       | Average DYAD minted                    |
| $\Epsilon$    | Burn multiplier product                |
| $\omega_b$    | Burn norm                              |
| $\chi$        | Burn limit                             |
| $\delta$      | Burn limit multiplier                  |
