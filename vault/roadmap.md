# Roadmap

## Phase 1: USDC Vault (Current)

**Status: Active**

The flagship vault establishing Dawn Vault's core infrastructure and track record.

- **Base Layer**: Kamino Multiply (ONyc/USDC ~16%, USDG/PYUSD ~9.5%) + USDC Lending (Kamino, Jupiter, 3-8%)
- **Alpha Layer**: SOL delta-neutral with dawnSOL enhancement (15-20% APY)
- **Target APY**: 9-16%+

### Milestones
- [x] Strategy backtest (821 days, Sharpe Ratio 27.01)
- [x] Manager Bot development
- [x] Live deployment with own capital
- [ ] Public deposits
- [ ] Performance reporting system
- [ ] Documentation site

---

## Phase 2: Strata — Tranched Vault

**Status: Planned**

Split the USDC Vault into Senior and Junior tranches, enabling institutional investors to participate with defined risk/return profiles.

| | Senior Vault | Junior Vault |
|---|---|---|
| **Return** | 8% fixed | Variable (residual upside) |
| **Loss priority** | Protected (last to absorb) | First-loss buffer |
| **Withdrawal** | Instant | 7-day lock |
| **Target** | Institutions, stable-yield seekers | High-yield seekers |

Underlying strategy remains the same as Phase 1. Risk separation is enforced via an on-chain Accounting Waterfall (Anchor program) built on Ranger Finance infrastructure.

> *"Stop losing sleep over hacks. Take the Senior tranche."*

---

## Phase 2 詳細設計: Strata

### なぜ今なのか

Drift ハック（2025年）以降、「全損リスクがある以上、稟議が通らない」という声が機関投資家から相次いでいる。Strata は Senior/Junior のトランチ構造によりリスクを分離し、Senior LP を部分的損失から保護する。Solana 上で CLO（Collateralized Loan Obligation）的な構造を初めて実用実装する。

---

### 2層構造

#### Risk Layer — Senior / Junior Tranche

| | Senior Vault | Junior Vault |
|---|---|---|
| **リターン** | 8% 固定（上限） | 残余全取り（変動・アップサイド大） |
| **損失順序** | 後回し（保護される） | 先に吸収（first-loss buffer） |
| **引き出し** | 即時 | 7日ロック |
| **対象** | 機関投資家・ステーブル志向 | 高利回り志向・リスク許容層 |

**Accounting Waterfall（出口の優先順位）:**

```
Senior payout = user_senior_share × min(total_nav, senior_total_deposits)
Junior payout = user_junior_share × max(total_nav - senior_total_deposits, 0)
```

- 利益が出た場合：Senior に 8% 固定を先払い → 残余を Junior に分配
- 損失が出た場合：Junior NAV が先に削られる → Senior は最後まで保護

#### Strategy Layer — Base / Alpha / DN

| | Base Layer | Alpha Layer | DN Layer |
|---|---|---|---|
| **戦略** | Kamino Lending | Kamino Multiply | SOL Delta-Neutral |
| **利回り** | ~5-7%（安定） | ~14-16%（高め） | ~変動（funding rate 依存） |
| **発動条件** | 常時（overflow） | 常時（primary） | SOL funding rate > 10% が3日継続 |
| **リスク** | 低 | 中（レバレッジあり） | 中（hedging） |

優先順位：Alpha（Multiply）に資金を集中 → 上限超過分を Base（Lending）に overflow。DN は条件付き発動。

---

### アーキテクチャ

```
[Dawn Senior Vault]     [Dawn Junior Vault]
        │                       │
        └───────────┬───────────┘
                    ▼
        Dawn Tranche Program（Anchor）
        ・NAV 追跡
        ・Waterfall 会計
        ・引き出し優先順位の制御
                    │ 全資金を一本化
                    ▼
        Ranger Vault（実行層）
        ├─ Kamino Multiply adaptor（Alpha Layer / primary）
        └─ Kamino Lending adaptor（Base Layer / overflow）
```

**技術スタック:**
- Anchor（Dawn Tranche Program）: Senior-LP / Junior-LP トークン発行、Waterfall 会計
- [Ranger Finance](https://docs.ranger.finance/)（Vault-as-a-Service インフラ）
  - `withdrawal_waiting_period` で Junior の 7日ロックをネイティブ実装
  - CPI integration: `deposit_vault` / `request_withdraw_vault` / `withdraw_vault` / `instant_withdraw`
- Kamino Finance（Multiply + Lending adaptors）

**設計の肝:**
Ranger は全 LP に対して単一の asset-per-share 比率で管理するため、Tranche Program が Ranger LP トークンを中間保有し、出金時に独自の Waterfall ロジックを適用することで Senior/Junior の不均等分配を実現する。

---

### Junior Bootstrap 戦略

初期の Junior 資金は **Dawn Labs 自身が張る**。

- Dawn Labs が first-loss を取ることで Senior LP に対して「skin in the game」のコミットを示す
- 実績が積み上がれば高利回り目的の外部 Junior LP を誘引可能
- ハッカソンデモ段階では Dawn Labs 資金で完結

---

### $1M ARR への道筋

手数料体系（Phase 1 と共通）：
- Performance Fee: 20%（High Water Mark）
- Management Fee: 1% / 年
- Withdrawal Fee: 0.1%

```
$10M Senior AUM × 8% yield × 20% perf fee  = $160K / 年
$5M Junior AUM  × 20% yield × 20% perf fee = $200K / 年
→ 合計 ~$360K / 年 @ $15M AUM

$1M ARR 達成には ~$40-50M AUM が必要
```

機関クライアントへの Senior Vault 提案（既存パイプライン: Mobcast, Pacific Meta, KEY3 等）から積み上げる。

---

### 先行事例・競合分析

#### TradFi の先行事例
- **CDO / CLO**（伝統金融）: 住宅ローン・レバレッジドローンをトランチ分け。AAA（Senior）→ Equity（Junior）。Strata は CLO のオンチェーン版に相当
- **不動産ファンドの優先劣後構造**（日本含む）: 優先出資（元本保護）/ 劣後出資（損失先吸収・アップサイド大）。同一構造

#### Crypto の先行事例（失敗事例）
- **Barnbridge**（Ethereum、2021）: SMART Yield で Compound 等の利回りを Senior/Junior に分離。$178M TVL → 低金利環境で固定 Senior 利率を維持できず失敗
- **Saffron Finance**（Ethereum、同時期）: 同様の構造。同じ理由で衰退

#### Solana 上の直接競合
- **Kormos**（Cypherpunk 2025年9月、DeFi 2位、C4アクセラレーター採択）
  - 構造：Liquid depositor（Junior 的）/ Locked depositor（Senior 的）
  - ナラティブ：「銀行の部分準備制度を DeFi で」
  - 差異：Locked = first-loss という **逆方向のトランチ**。機関投資家・全損リスク排除は明示していない

#### Strata の差別化
1. **ナラティブ**: Drift ハック後の「全損リスク忌避」という具体的な痛点に応答
2. **ターゲット**: 機関投資家（Senior）が主たる顧客。稟議が通る構造設計
3. **利回り源泉**: Kamino Multiply（~16%）という高い原資があって初めて Senior 8% 固定が成立
4. **実運用バックグラウンド**: Dawn Labs 自身が Phase 1 Vault を稼働させており、戦略・リスク管理の実績あり

---

### 残課題

- [ ] Anchor program の詳細設計（NAV 計算のオンチェーン化度合い）
- [ ] Ranger Vault Owner 登録・adaptor 設定の具体的手順確認（Ranger チームと壁打ち済み）
- [ ] CPI ルーティングの実現可能性検証（1つの Anchor プログラムが2つのデポジタープールの LP トークンを保持できるか）
- [ ] locked profit degradation / HWM フィー計算と Tranche 会計の非同期問題の解決策確定
- [ ] Junior の外部募集タイミング・条件設計
- [ ] ピッチ資料作成

---

## Phase 3: Japan Institutional Access

**Status: Planned**

Expand access to Japanese institutional investors and qualified investors (適格投資家) through regulatory alignment and JPY-native on-ramps.

- **Regulatory & audit compliance**: Structured reporting and audit trails meeting Japanese financial regulations
- **JPY stablecoin integration**: Enable vault participation via JPY stablecoins (e.g., JPYC, progmat coin) to remove FX friction for domestic investors
- **KYC/AML layer**: Permissioned deposit flow for compliance with Japanese fund regulations
- **Localized reporting**: Performance reports and risk disclosures in Japanese, aligned with domestic institutional requirements

---

*Timelines are indicative and subject to change based on market conditions and development progress.*
