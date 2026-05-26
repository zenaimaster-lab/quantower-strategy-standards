---
name: quantower-strategy-standards
description: Global development, metadata documentation, compilation, and safety standards for Quantower C# trading strategies. Use when writing, refactoring, or reviewing any Quantower C# strategies or indicator systems.
---

# Quantower C# Strategy Standards

A set of premium development guidelines and automation procedures to ensure robust, self-documenting, and error-free C# strategy development for the Quantower Trading Platform.

## Quick Start (Pre-Flight checklist)

Whenever you edit or create a Quantower C# strategy, you must:
1. **Always** supplement a `"0. METADATA & SYSTEM INFO"` settings category with standard parameters.
2. **Always** bump the strategy version inside both the `STRATEGY_VERSION` constant and settings parameter.
3. **Always** run the **atomic build-deploy-sync** workflow immediately after changes.

---

## 1. Mandatory Settings Metadata

Every strategy must feature a dedicated `"0. METADATA & SYSTEM INFO"` category inside its input parameters. This allows traders to immediately identify the active bot's capabilities, version, time of compilation, and compatibility requirements inside the Quantower Settings panel.

```csharp
[Category("0. METADATA & SYSTEM INFO")]
[InputParameter("Bot Description", 0)]
public string InpBotDescription = "Automated [Strategy Name] strategy featuring [Core Mechanism].";

[Category("0. METADATA & SYSTEM INFO")]
[InputParameter("Strategy Version", 1)]
public string InpStrategyVersion = "1.1";

[Category("0. METADATA & SYSTEM INFO")]
[InputParameter("Adapter Version", 2)]
public string InpAdapterVersion = "v1.145.17"; // Matches referenced TradingPlatform.BusinessLayer.dll version

[Category("0. METADATA & SYSTEM INFO")]
[InputParameter("Last Updated (UTC)", 3)]
public string InpLastUpdated = "YYYY-MM-DD HH:mm:SS";
```

### Metadata Best Practices:
- **Version Bumping**: Increment the strategy version (e.g. `1.0` -> `1.1` -> `1.2`) on every logical update or feature injection.
- **Adapter Reference**: Match the exact version code of the referenced Quantower TradingPlatform DLL (e.g. `v1.145.17`).
- **Timestamp**: Keep the `InpLastUpdated` parameter aligned with the local system time of your latest codebase modification.

---

## 2. Mandatory Build-Deploy-Sync Workflow

To ensure strategies run with the latest local changes, compile the project in **Release** mode, deploy the compiled binaries (`.dll` and `.pdb` for debugging symbols) to the Quantower runtime folder, and sync all source changes to Git as a single, atomic operation:

```powershell
# 1. Compile in Release mode
dotnet build -c Release

# 2. Deploy compiled DLL & PDB to Quantower runtime Scripts folder (Force overwrite)
Copy-Item -Path "bin\Release\net8.0-windows\<AssemblyName>.dll", "bin\Release\net8.0-windows\<AssemblyName>.pdb" -Destination "C:\Quantower\Settings\Scripts\Strategies\" -Force

# 3. Commit and push strategy source code to Remote Repository
git add .
git commit -m "feat: [brief description of strategy updates]"
git push origin master
```

---

## 3. Order Safety & Stop-Level Wrapping

When placing pending orders (such as `OrderType.Stop` or `OrderType.Limit` orders), ensure you validate the price distance against the current market quote and broker stops levels to avoid `[Invalid price]` rejections:

- If the breakout entry price is **slightly past** the market quote but within the `InpMaxChaseTicks` threshold, immediately fall back to placing a **Market Order** to avoid missing the breakout.
- If the breakout entry price is **too far past** the quote, place a **Limit Order** at the breakout level to await a pullback.
- Always use `Math.Round(price / tickSize) * tickSize` to safely round prices to the symbol's exact tick format before order submission.

---

## 4. Coding Style and Project Standards

- **Group Settings**: Always group inputs using descriptive `[Category("...")]` headers next to logical parameter order indices.
- **Error Handling**: Implement structured `try-catch` blocks for all trading and historical subscription logic. Log details using the strategy's native `Log(...)` system.
- **Resource Management**: Dispose of all `HistoricalData` subscriptions inside `OnStop()` to prevent memory leaks in the Quantower desktop runtime.
