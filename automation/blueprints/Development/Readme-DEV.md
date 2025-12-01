# 🔋 Frank Energie - Smart Battery Manager (Dev / v1.4.3)

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fpwnypower%2FSmartcharging-HA%2Fblob%2Fdev%2Fautomation%2Fblueprints%2FSmartcharging%20frankenergie_v1.4.3.yaml)

**The "Pro" version: Strategic Profit Optimization & Weighted Peak Analysis.**

While v1.4 focuses on simplicity ("Just-in-Time"), this **v1.4.3** version introduces a strategic algorithm. It doesn't just look at the next hour; it analyzes the entire day to find the "Weighted Peak" prices and calculates buffers based on relative market value.

It is designed for users who want to maximize **financial return (ROI)** while strictly protecting their hardware (EEPROM).

---

## ✨ Key Features (v1.4.3 Specifics)

* **🧠 Weighted Peak Analysis:** The system splits the day into AM/PM blocks and calculates the average of the **Top 3 most expensive hours**. It uses this "Peak Reference" to determine if a current price is truly "cheap" or just "average".
* **📉 Dynamic Percentage Logic:**
    * **Charging:** Only charges if the price is `X%` lower than the Daily Peak.
    * **Discharging:** Only discharges if the price is `Y%` higher than your purchase price.
* **⚓ Smart Hold (The Ratchet):** Instead of switching the battery OFF when prices are moderate, it anchors the `Target SoC` to the `Current SoC`. This prevents grid usage but allows **free solar energy** to fill the battery further without adjusting settings (Zero EEPROM writes).
* **☕ Strict Morning Guarantee:** Forces a base charge (e.g., 50%) for the morning, but is strictly bound to the **Night Slot** to prevent accidental peak charging during the day.
* **🛡️ Switch-Only Discharge:** To save hardware write-cycles, the system only toggles the switch OFF to allow discharging, without rewriting the Target SoC.

---

## ⚙️ Prerequisites

Before importing, ensure you have the standard requirements plus the Helper:

### 1. Integrations
* **Frank Energie** (or any sensor with `prices` attribute).
* **Forecast.Solar** integration.

### 2. Helpers (Required!)
Go to **Settings > Devices & Services > Helpers > Create Helper**:

1.  **Input Number** (Required):
    * **Name:** `Last Purchase Price`
    * **Icon:** `mdi:currency-eur`
    * **Min:** `0`, **Max:** `5`
    * **Step:** `0.001`
    * *Purpose: Stores the exact kWh price paid during charging to calculate profit later.*

2.  **Input Boolean** (Optional):
    * **Name:** `Battery Boost`
    * *Purpose: Manual override to force charge to 100%.*

---

## 🚀 Configuration Guide

### 🔧 Strategy Settings

#### 📉 Buffer Discount (%)
* **The "Bargain Hunter" setting.**
* *How it works:* It compares the current price to today's **Weighted Peak**.
* *Formula:* `Max Charge Price = Peak Price * (100% - Discount%)`.
* **Example:** Peak is €0.40. Discount is **40%**. The system will charge if price drops below **€0.24**.
* **Recommended:** `40%`. (Higher = Stricter/Less charging).

#### 💰 Min. Discharge Profit (%)
* **The "Trader" setting.**
* *How it works:* It compares the current price to your **Last Purchase Price**.
* *Formula:* `Min Sell Price = Bought Price * (100% + Profit%)`.
* **Example:** Bought at €0.20. Profit is **20%**. The system allows discharging above **€0.24**.
* **Recommended:** `15% - 20%` (Covers ~15% battery conversion loss).

#### ☕ Morning Guarantee (%)
* **The "Comfort" setting.**
* *How it works:* Ensures you reach this SoC before the morning starts.
* *Constraint:* Only active during the configured **Night Slot** (e.g., 00:00 - 06:00). It will ignore high prices (up to the safety cap) to ensure this level is reached.
* **Recommended:** `40% - 60%`.

#### 📊 Peak Hours Count
* *How it works:* How many "expensive hours" should be averaged to define the "Peak"?
* **Recommended:** `3`. (Setting it to 1 makes the system too sensitive to a single price spike).

---

## 🧠 Logic Flow (Decision Matrix)

Every 5 minutes, the system evaluates the following priority list:

1.  **🚀 BOOST:** Is the manual Boost switch ON? -> **Charge to 100%.**
2.  **⚠️ EMERGENCY:** Is SoC < Absolute Minimum (15%)? -> **Charge.**
3.  **🟢 CHARGE:**
    * Is it Night AND SoC < Morning Guarantee? -> **Charge.**
    * Is Price < (Peak - Buffer Discount)? -> **Charge.**
    * Is Price one of the absolute cheapest hours needed for the target? -> **Charge.**
    *(Action: Switch ON, Target MAX, Save Price).*
4.  **🔴 DISCHARGE:** Is Price > (Bought Price + Profit %)? -> **Release.**
    *(Action: Switch OFF).*
5.  **🟡 HOLD:** Price is "Meh" (Between Charge & Discharge). -> **Anchor.**
    *(Action: Switch ON, Target = Current SoC).*

---

## ❓ FAQ (Dev Branch)

**Q: My battery is at 100% but the "Target SoC" in the inverter says 80%. Why?**
A: This is the **Solar Ratchet**. The system anchored the grid-charge at 80%. Solar energy pushed it to 100%. The system purposely *didn't* update the setting to 100% to save a write-cycle on your inverter's memory chip. This is normal behavior.

**Q: Why isn't it charging? The price is low!**
A: Check the **Buffer Discount**. The system might see that although the price is low, the *Peak* for today isn't very high either, meaning the relative discount isn't big enough to justify the wear on the battery.

**Q: Why does the Morning Guarantee not work in the afternoon?**
A: By design. The Morning Guarantee is strictly bound to the `Night Slot` to prevent the battery from force-charging during expensive afternoon peaks if it happens to dip below the threshold.

---

## ☕ Support development

Building complex algorithms takes time. If this blueprint optimizes your return on investment, consider buying a coffee!

<a href="https://www.buymeacoffee.com/pwnypower" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

---

## 📄 License

* **Version:** 1.4.3 (Development / Test)
* **Author:** PwnyPower