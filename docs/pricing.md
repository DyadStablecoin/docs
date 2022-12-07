---
sidebar_position: 6
---

# Price modeling

Let $F$ be the theoretical "Base price" of a dNFT - the amount a buyer is willing to pay to enter the Dyad protocol and the current inherent value of the dNFT.
We then add it to the Net Present Value (NPV), which is the sum of a dNFT's potential future cash flows discounted to the present time.

$$
P = \beta + NPV
$$

## Upside

Net Present Value is a valuation methodology which examines an asset's future cash flows and discounting them to the present.
The formula used in modern financial services is

$$
NPV = \sum\limits_{t=1}^N\frac{R_t}{(1+i)^t},
$$

where $R$ is defined as the cash flow at time $t$ and $i$ is defined as the discount rate (typically rate of inflation) summed over $N$ snapshots.
The cash flow will ultimately depend on two factors: net upside and ether implied volatility ($\sigma$).

Assume $\sigma$ is equal to 0.2 for the next year.
This means that one standard deviation in the distribution of possible ETH prices a year from now will be 20% of the current price.
Now assume that a given dNFT's net upside exposure is 1.0.
This means that for every $n\%$ change in ETH price, the dNFT will see an $n\%$ gain in its balance

$$
NPV = \sum\limits_{t=1}^N\frac{\rho^{\sigma \upsilon t}}{(1+i)^t},
$$

where $\rho$ equalts the current DYAD Balance, $\sigma$ the ETH implied volatility, $\upsilon$ the net upside, and $t$ the time in years.

However, dNFT cash flow happens continuously as DYAD is minted and burned to ensure that the value of the Eth collateral vault is equal to the number of DYAD tokens in circulation.
Therefore, a dNFT's NPV is calculated as follows:

$$
NPV = \rho + \lim_{n\to\infty}\sum_{t=1}^{ny}\dfrac{\rho(\frac{\sigma \upsilon}{n})^{nt}}{(1+\dfrac{i}{n})^{nt}},
$$

with $\rho$ as current DYAD Balance, $\sigma$ as ETH implied volatility, $\upsilon$ as net upside, $t$ as time in years, $i$ as discount rate, and $y$ as desired NPV forecast range in years.

This simplifies to:

$$
NPV = \rho + \int_{0}^{y}\dfrac{\dfrac{d}{dt}\rho(e^{\sigma \upsilon t})}{e^{it}}dt
$$

Think of $Pe^{rt}$ for continuous compound interest while the denominator fits the definition of e.
In the numerator, like $Pe^{rt}$, $\rho e^{\sigma\upsilon t}$ gives a dNFT's actual DYAD balance over time.
The derivative of this function is used to find the instantaneous cash flow (rate of change of the principal).

## Downside

Additionally, we must consider the effect that XP accrual from burned DYAD has on the $XP$ rank of the dNFT, hence increasing its valuation.
We can approximate future $XP$ ranking growth and thus net upside in relation to time.

Let $\delta$ be the dNFT relative $XP$ rank at time of purchase.
$\xi(t)$ returns a future relative $XP$ rank given any time input:

$$
\xi(t) = \dfrac{2}{\pi}{\tan}^{-1}{\left(\dfrac{\pi}{2}\sigma (t + f(\delta))\right)}
$$

Here, $f(\delta)$ is a modifier which shifts this function so that the initial $XP$ rank at $t=0$ is maintained.
It is defined as follows:

$$
f(\delta) = \dfrac{2}{\pi \sigma}\tan\left(\dfrac{\pi}{2}\delta\right)
$$

Subsequently, $\upsilon(\xi(t))$ is a function of the of dNFT net upside with respect to its relative $XP$ rank $(\xi(t))$

$$
\upsilon(\xi(t)) = \Omega*\Psi - \Gamma,
$$

where

$$
\Omega = 1.5 + \tanh(11.0 (\xi(t) - 0.85))
$$

and

$$
\Gamma = 3.0 - \Omega(\xi(t)).
$$

If a dNFT has a negative $\upsilon(\xi(t))$ at time of purchase, its **break-even time** $(t_b)$ can be found by letting $\upsilon(\xi(t)) = 0$ and solving for $t$.

Now, we can use $\upsilon(\xi(t))$ instead of a constant $\upsilon$ to model the dNFT's net present value.
Additionally, we need to consider a dNFT's resale value, as its future cash flow and inherent value increases as it climbs the leaderboard.
At $y$, the value of the dNFT can be calculated from the same NPV with time bounds $y$ to $2y$ and then added to the current valuation.
This means that the integral should be from $0$ to $2y$, with $y$ being the period intended for the dNFT. This simplifies to:

$$
NPV = \int_{0}^{2y}\dfrac{\dfrac{d}{dt}\rho(e^{\sigma t (\upsilon(\xi(t)))})}{e^{it}}dt
$$

## Base value

The base value of a dNFT should be affected by several factors:

- DYAD market cap in relation to ETH market cap.
- Current dNFT balance.
- Current dNFT reserve ratio.
- Floor price, or what investors are willing to pay to enter the protocol.

Now, let us consider the amount of ETH in the DYAD collateral vault as a percentage of total ETH market cap.
We define the following properties: $C_D$ = ETH in DYAD collateral vault and $C_E$ = ETH market cap, and $\frac{C_D}{C_E}$ = $C_P$ = Percentage of DYAD ETH making up total market cap to account for DYAD total value locked.
Given this, the base value should be defined as

$$
B=(\alpha*C_P*\sigma+\rho)*\frac{R}{1.5},
$$

where $\rho$ = current DYAD balance, $\sigma$ = ETH implied volatility, $R$ = reserve ratio, or DYAD in a dNFT's balance over DYAD in circulation from the dNFT.
A "proper" dNFT should have a reserve ratio of approx. 1.5, and dNFTs which have a smaller reserve ratio will be worth less.
The floor equilibrium price $F = \alpha C_P \sigma$, and $\alpha$ is an unknown value which needs to be gathered empirically.
Theoretically, it represents the floor price if the price of ETH were extremely volatile, and the DYAD protocol was as important as the entire Ethereum system.
