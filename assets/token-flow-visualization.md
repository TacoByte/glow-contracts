# Glow Protocol — Token Flow Visualization

## Full System Overview

```mermaid
flowchart TB
  subgraph Inflation["GLOW Inflation (230k/week)"]
    MINT[GLOW Mint]
  end

  subgraph Actors
    USER([User])
    FARMS([Solar Farms])
    GCAS([GCA Agents])
  end

  subgraph Core["Core Contracts"]
    EL[Early Liquidity\nBonding Curve]
    MP[Miner Pool & GCA]
    SAFETY[Safety Delay\n7-day lockup]
    UNLOCK[Glow Unlocker\n96M vesting]
  end

  subgraph Market["Market & Auctions"]
    AUCTION[GCC Auction\nDescending Price]
    IMPACT[Impact Catalyst]
    UNI[Uniswap V2\nGCC / USDC LP]
  end

  subgraph Gov["Governance"]
    GOV[Governance]
    VETO[Veto Council]
    GRANTS[Grants Treasury]
  end

  subgraph Tokens
    GCC_T([GCC Token])
    USDG_T([USDG Wrapper])
  end

  %% ── GLOW Inflation ──
  MINT -->|"175k/wk"| MP
  MINT -->|"10k/wk via MP"| GCAS
  MINT -->|"5k/wk"| VETO
  MINT -->|"40k/wk"| GRANTS
  MINT -->|"12M initial"| EL
  UNLOCK -->|"96M vest"| MINT

  %% ── Early Liquidity ──
  USER -->|"USDC buy"| EL
  EL -->|"GLOW sold"| USER
  EL -->|"USDC donated"| MP

  %% ── Miner Pool Rewards ──
  MP -->|"GLOW rewards"| FARMS
  MP -->|"USDC rewards"| SAFETY
  SAFETY -->|"USDC after 7d"| FARMS
  MP -->|"GCC minted"| FARMS
  GCAS -->|"submit reports"| MP

  %% ── GCC Auction ──
  MP -->|"GCC to auction"| AUCTION
  AUCTION -->|"GCC sold"| USER
  USER -->|"GLOW pay → burn"| AUCTION

  %% ── GCC Commitment ──
  USER -->|"commit GCC"| IMPACT
  IMPACT -->|"GCC + USDC LP"| UNI
  IMPACT -.->|"Nominations\n√(gcc × usdc)"| GOV

  %% ── Governance ──
  USER -->|"stake GLOW"| GOV
  GOV -->|"allocate grants"| GRANTS
  GRANTS -->|"claim GLOW"| USER

  %% ── Veto Council ──
  VETO -.->|"delay +90d"| SAFETY

  %% ── USDG ──
  USER -->|"wrap USDC 1:1"| USDG_T

  %% ── Styles ──
  classDef glow fill:#f0883e22,stroke:#f0883e,color:#f0883e
  classDef gcc fill:#3fb95022,stroke:#3fb950,color:#3fb950
  classDef usdc fill:#58a6ff22,stroke:#58a6ff,color:#58a6ff
  classDef contract fill:#161b22,stroke:#30363d,color:#c9d1d9
  classDef actor fill:#1a1e24,stroke:#58a6ff,color:#58a6ff
  classDef gov fill:#161b22,stroke:#bc8cff,color:#bc8cff

  class MINT,EL,UNLOCK glow
  class GCC_T,AUCTION gcc
  class USDG_T,SAFETY usdc
  class MP,IMPACT,UNI contract
  class USER,FARMS,GCAS actor
  class GOV,VETO,GRANTS gov
```

## GLOW Token Flows

```mermaid
flowchart LR
  MINT[GLOW Mint\n230k/week] -->|"175k"| MP[Miner Pool]
  MINT -->|"40k"| GRANTS[Grants Treasury]
  MINT -->|"10k"| GCAS[GCA Agents]
  MINT -->|"5k"| VETO[Veto Council]
  MINT -->|"12M init"| EL[Early Liquidity]

  EL -->|"bonding curve"| USER([User])
  MP -->|"farm rewards"| FARMS([Solar Farms])
  GRANTS -->|"claim"| USER
  USER -->|"pay at auction → burned"| AUCTION[GCC Auction]
  USER -->|"stake for voting"| GOV[Governance]

  classDef glow fill:#f0883e22,stroke:#f0883e,color:#f0883e
  class MINT,EL,GRANTS,MP,GCAS,VETO glow
```

## GCC Token Flows

```mermaid
flowchart LR
  MP[Miner Pool] -->|"mint from\nfarm reports"| AUCTION[GCC Auction]
  MP -->|"direct to farms"| FARMS([Solar Farms])
  AUCTION -->|"descending price\nhalf-life 1 week"| USER([User])
  USER -->|"commit"| IMPACT[Impact Catalyst]
  IMPACT -->|"add liquidity"| UNI[Uniswap V2\nGCC/USDC LP]
  IMPACT -.->|"Nominations\n√(gcc × usdc)"| GOV[Governance]
  USER -->|"burn = retire\n1 GCC = 1 ton CO₂"| BURN[Burned]

  classDef gcc fill:#3fb95022,stroke:#3fb950,color:#3fb950
  class MP,AUCTION,IMPACT,UNI,BURN gcc
```

## USDC Flows

```mermaid
flowchart LR
  USER([User]) -->|"buy GLOW"| EL[Early Liquidity]
  EL -->|"all proceeds\ndonated"| MP[Miner Pool]
  MP -->|"farm USDC"| SAFETY[Safety Delay\n7 days]
  SAFETY -->|"claim after\n7-day lock"| FARMS([Solar Farms])
  VETO[Veto Council] -.->|"can extend\n+90 days"| SAFETY
  USER -->|"commit USDC"| IMPACT[Impact Catalyst]
  IMPACT -->|"LP"| UNI[Uniswap V2]
  USER -->|"wrap 1:1"| USDG[USDG]

  classDef usdc fill:#58a6ff22,stroke:#58a6ff,color:#58a6ff
  class EL,MP,SAFETY,IMPACT,UNI,USDG usdc
```

## Key Mechanics

| Mechanism | Details |
|-----------|---------|
| **GLOW Inflation** | 230k/week split: 175k Miners, 40k Grants, 10k GCAs, 5k Veto |
| **Early Liquidity** | Price = $0.003 × 2^(sold/1M), 12M supply |
| **GCC Minting** | 1 GCC = 1 ton CO₂, minted from validated farm reports |
| **GCC Auction** | Reverse Dutch auction, 0.1 GLOW start, 1-week half-life decay |
| **Nominations** | √(gcc_committed × usdc_received), used for proposals & votes |
| **Safety Delay** | 7-day USDC lockup, Veto can extend to 97 days |
| **GLOW Staking** | Stake for governance votes, 5-year unstake cooldown |
| **GCC Burning** | Permanently retire carbon credits, tracked on-chain |
