# Quantower C# Strategy & Indicator Standards Skill

An automated agent skill and standard operating procedure for developing high-performance, self-documenting, and error-free C# trading strategies and indicators on the **Quantower Trading Platform**. 

Adheres strictly to official Quantower standards, preventing syntax hallucinations (e.g. from NinjaTrader or MetaTrader).

---

## ✨ Features & Specifications

- 🛡️ **Mandatory Metadata Settings**: Ensures every compiled strategy is self-documenting directly inside Quantower's settings panel.
- 🔄 **Strategy & Indicator Lifecycle Guards**: Standardizes lifecycle hooks (`OnCreated`, `OnRun`, `OnStop`, `OnRemove` for bots; `OnInit`, `OnUpdate` for indicators) to prevent memory leaks and thread locks.
- ⚙️ **Atomic Dev-Ops Pipeline**: Zero-friction local compilation, binary hot-deployment, and remote GitHub synchronization.
- 🔒 **Strict Syntax Lock-in**: Locks LLM code generators to pure `TradingPlatform.BusinessLayer` assemblies, eliminating NinjaScript/MQL confusion.

---

## 📂 Repository Structure

- `SKILL.md`: Detailed instructions and code generation templates for AI coding agents.
- `README.md`: Public-facing documentation and installation guide.

---

## 💻 Standard C# Strategy Template

```csharp
using System;
using TradingPlatform.BusinessLayer;

namespace CustomQuantowerBot
{
    public class CustomQuantowerBot : Strategy
    {
        [Category("0. METADATA & SYSTEM INFO")]
        [InputParameter("Bot Description", 0)]
        public string InpBotDescription = "Automated strategy for Quantower.";

        [Category("0. METADATA & SYSTEM INFO")]
        [InputParameter("Strategy Version", 1)]
        public string InpStrategyVersion = "1.0";

        [Category("0. METADATA & SYSTEM INFO")]
        [InputParameter("Adapter Version", 2)]
        public string InpAdapterVersion = "v1.145.17";

        [Category("0. METADATA & SYSTEM INFO")]
        [InputParameter("Last Updated (UTC)", 3)]
        public string InpLastUpdated = "2026-05-27 09:23:28";

        [Category("1. GENERAL & SCHEDULE")]
        [InputParameter("Trading Symbol", 10)]
        public Symbol currentSymbol;

        [Category("1. GENERAL & SCHEDULE")]
        [InputParameter("Trading Account", 20)]
        public Account currentAccount;

        public CustomQuantowerBot() : base()
        {
            this.Name = "Custom QW Bot";
            this.Description = "Automated strategy for Quantower";
        }

        protected override void OnCreated() { }

        protected override void OnRun()
        {
            if (currentSymbol == null || currentAccount == null || currentSymbol.ConnectionId != currentAccount.ConnectionId)
            {
                Log("Error: Symbol or Account not specified correctly.", StrategyLoggingLevel.Error);
                return;
            }

            this.currentSymbol = Core.GetSymbol(this.currentSymbol?.CreateInfo());
            if (this.currentSymbol != null)
                this.currentSymbol.NewLast += SymbolOnNewLast;
        }

        protected override void OnStop()
        {
            if (this.currentSymbol != null)
                this.currentSymbol.NewLast -= SymbolOnNewLast;
        }

        private void SymbolOnNewLast(Symbol symbol, Last last)
        {
            // Execution logic goes here
        }
    }
}
```

---

## 🛠️ Global Installation (Antigravity AI Platform)

This skill is designed to run globally inside the **Antigravity AI agent config**. To install or load it manually:

1. Clone this repository to your global agent skills folder:
   ```bash
   git clone https://github.com/zenaimaster-lab/quantower-strategy-standards.git ~/.gemini/config/skills/quantower-strategy-standards
   ```
2. The AI agent will automatically detect and load `quantower-strategy-standards` on startup.

---

## 🤝 Contribution & Support

Maintained with ❤️ by the **ZenaiMaster Lab** team. Feel free to open issues or submit pull requests for additional Quantower development guidelines!
