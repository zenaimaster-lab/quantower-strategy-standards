# Quantower C# Strategy Standards Skill

An automated agent skill and standard operating procedure for developing high-performance, self-documenting, and error-free C# trading strategies on the **Quantower Trading Platform**.

## Features & Specifications

- 🛡️ **Mandatory Metadata Settings**: Ensures every compiled strategy is self-documenting directly inside Quantower's settings panel.
- ⚙️ **Atomic Dev-Ops Pipeline**: Zero-friction local compilation, binary hot-deployment, and remote GitHub synchronization.
- 🎯 **Stop-Level Safety Wrappers**: Auto-recovering order routing to convert pending breakout entries into Market or Limit executions based on real-time chase-tick criteria.
- 📈 **Memory & Resource Leak Safety**: Enforces complete lifecycle disposal of historical candles and indicators to prevent desktop client lag.

---

## 📂 Repository Structure

- `SKILL.md`: Main instructions and formatting directives for AI coding agents.
- `README.md`: Public-facing project overview and configuration guides.

---

## 🛠️ Global Installation (Antigravity AI Platform)

This skill is designed to run globally inside the **Antigravity AI agent config**. To install or load it manually:

1. Clone or download this repository to your global agent skills folder:
   ```bash
   git clone https://github.com/zenaimaster-lab/quantower-strategy-standards.git ~/.gemini/config/skills/quantower-strategy-standards
   ```
2. The AI agent will automatically detect and load `quantower-strategy-standards` on startup.

---

## 💻 Standard C# Metadata Template

Apply this setting template to the top of your strategy class definition:

```csharp
[Category("0. METADATA & SYSTEM INFO")]
[InputParameter("Bot Description", 0)]
public string InpBotDescription = "Automated strategy description.";

[Category("0. METADATA & SYSTEM INFO")]
[InputParameter("Strategy Version", 1)]
public string InpStrategyVersion = "1.1";

[Category("0. METADATA & SYSTEM INFO")]
[InputParameter("Adapter Version", 2)]
public string InpAdapterVersion = "v1.145.17";

[Category("0. METADATA & SYSTEM INFO")]
[InputParameter("Last Updated (UTC)", 3)]
public string InpLastUpdated = "2026-05-27 09:17:06";
```

---

## 🤝 Contribution & Support

Maintained with ❤️ by the **ZenaiMaster Lab** team. Feel free to open issues or submit pull requests for additional Quantower development guidelines!
