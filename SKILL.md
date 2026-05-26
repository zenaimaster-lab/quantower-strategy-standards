---
name: quantower-strategy-standards
description: Global development, metadata documentation, compilation, and safety standards for Quantower C# trading strategies. Trigger whenever writing, refactoring, or reviewing any C# code, strategies, indicators, or system parameters related to the Quantower (QT) trading platform.
---

# Quantower C# Strategy & Indicator Standards

You are an expert C# Algorithmic Trading Developer specializing in the Quantower platform. Your task is to write clean, efficient, and error-free trading strategies (bots) and indicators using the Quantower API, adhering strictly to the official QuantowerKB and api-examples.

---

## 1. Core Namespaces

All Quantower C# scripts **MUST** include these namespaces:
```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using TradingPlatform.BusinessLayer;
```

---

## 2. Rules for Strategies (Bots)

1. **Class Inheritance**: Always inherit from the `Strategy` class.
2. **Settings**: Use the `[InputParameter]` attribute for user-defined properties (e.g., Symbol, Account, SL, TP).
3. **Strategy Lifecycle Methods**: The following methods **MUST** be overridden:
   - `OnCreated()`: Called once when the strategy is added. Use for initialization.
   - `OnRun()`: Called when the 'Run' button is pressed. Always validate inputs here (e.g., check if Symbol and Account are matching connection IDs). Subscribe to market events here (e.g., `this.currentSymbol.NewQuote += SymbolOnNewQuote`).
   - `OnStop()`: Called when stopped. **ALWAYS** unsubscribe from events here to prevent memory leaks (e.g., `this.currentSymbol.NewQuote -= SymbolOnNewQuote`).
   - `OnRemove()`: Final cleanup when the strategy is completely removed from the panel.

---

## 3. Rules for Indicators

1. **Class Inheritance**: Always inherit from the `Indicator` class.
2. **Indicator Lifecycle Methods**:
   - `OnInit()`: Setup lines, buffers, and colors (e.g., `AddLineSeries()`).
   - `OnUpdate(UpdateArgs args)`: Core calculation logic executed on every new bar/quote. Use `GetPrice(PriceType.Close, 0)` to get current bar data, and `SetValue()` to push data to the buffer.

---

## 4. Standard C# Strategy Template

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

        [Category("1. GENERAL & SCHEDULE")]
        [InputParameter("Trade Quantity", 30)]
        public double quantity = 1.0;

        public CustomQuantowerBot() : base()
        {
            this.Name = "Custom QW Bot";
            this.Description = "Automated strategy for Quantower";
        }

        protected override void OnCreated()
        {
            // One-time initialization logic
        }

        protected override void OnRun()
        {
            // Input validation is critical in Quantower API
            if (currentSymbol == null || currentAccount == null || currentSymbol.ConnectionId != currentAccount.ConnectionId)
            {
                Log("Error: Symbol or Account not specified correctly.", StrategyLoggingLevel.Error);
                return;
            }

            // Ensure symbol is populated with full data context
            this.currentSymbol = Core.GetSymbol(this.currentSymbol?.CreateInfo());
            
            if (this.currentSymbol != null)
            {
                // Subscribe to tick data
                this.currentSymbol.NewLast += SymbolOnNewLast;
            }
        }

        protected override void OnStop()
        {
            if (this.currentSymbol != null)
            {
                // Mandatory unsubscription to avoid memory leaks
                this.currentSymbol.NewLast -= SymbolOnNewLast;
            }
        }

        private void SymbolOnNewLast(Symbol symbol, Last last)
        {
            // Core execution logic goes here
            // Example Order Placement Syntax:
            /*
            Core.Orders.PlaceOrder(new PlaceOrderRequestParameters()
            {
                Account = this.currentAccount,
                Symbol = this.currentSymbol,
                OrderTypeId = OrderType.Market,
                Side = Side.Buy,
                Quantity = this.quantity,
                TimeInForce = TimeInForce.Day
            });
            */
        }
    }
}
```

---

## 5. Mandatory Settings Metadata Category

Every strategy must feature a dedicated `"0. METADATA & SYSTEM INFO"` category inside its input parameters:
- **Bot Description**: Summarizes the bot's features and target mechanisms.
- **Strategy Version**: Incremented on every logical update or feature injection (e.g. `1.0` -> `1.1` -> `1.2`).
- **Adapter Version**: Matches the exact version code of the referenced Quantower DLL (e.g. `v1.145.17`).
- **Last Updated (UTC)**: Aligned with the local system time of your latest codebase modification.

---

## 6. Mandatory Build-Deploy-Sync Workflow

To ensure strategies run with the latest local changes, execute the following commands sequentially after any modifications:

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

## 7. Order Safety & Stop-Level Wrapping

- When placing pending orders (such as `OrderType.Stop` or `OrderType.Limit` orders), ensure you validate the price distance against the current market quote and broker stops levels to avoid `[Invalid price]` rejections.
- If the breakout entry price is **slightly past** the market quote but within the `InpMaxChaseTicks` threshold, immediately fall back to placing a **Market Order** to avoid missing the breakout.
- If the breakout entry price is **too far past** the quote, place a **Limit Order** at the breakout level to await a pullback.
- Always use `Math.Round(price / tickSize) * tickSize` to safely round prices to the symbol's exact tick format before order submission.

---

## 8. Strict Constraints & Best Practices

- 🔒 **Syntax Lock-in**: **DO NOT** use NinjaScript (NinjaTrader) or MQL (MetaTrader) syntax. Quantower uses purely `Core.Orders.PlaceOrder`, `Core.Positions`, `Core.GetSymbol()`, and `TradingPlatform.BusinessLayer`.
- 📢 **Logging**: Output messages using `Log("message", StrategyLoggingLevel.Info);`.
- 🪟 **WPF & WinForms Support**: Enabled by default inside `.csproj` via `<UseWPF>true</UseWPF>` and `<UseWindowsForms>true</UseWindowsForms>`.
- 🧹 **Garbage Collection**: Dispose of all `HistoricalData` and unsubscribe from all events inside `OnStop()` or `OnRemove()`.
