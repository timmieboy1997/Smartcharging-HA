# 🔋 Frank Energie - Smart Battery Manager v1.7 (BETA)

> **STRATEGY:** Profit & Hardware Protection.

This blueprint completely automates your home battery based on dynamic energy prices (Frank Energie/Zonneplan) and solar forecasts. It aims to charge at the absolute lowest moments, hold charge during neutral prices, and discharge only when profitable or needed for home consumption.

---

## 🚀 New in v1.7
* **Fix v1.6.1:** Solved a syntax error in the 'default' action sequence.
* **Dayround Logic:** The logic is now active **24/7**, ensuring optimal decisions throughout the day and night (removed previous time restrictions).
* **Rolling Peak Analysis:** The system now looks ahead 12 hours. It calculates a rolling average of peak prices to prevent charging now if prices are expected to drop significantly tomorrow.
* **Backup SoC Logic:** Removed the necessity for the system to adjust the hardware backup SoC when disabling the reserve switch.

---

## ✨ Key Features & Functions

1.  **Smart Price Analysis (Rolling Window)**
    Instead of hardcoded times, the blueprint calculates a rolling average of the most expensive hours in the upcoming 12-hour window. It only charges when the current price is significantly lower (e.g., 40% discount) than this average.

2.  **Solar Forecast Integration**
    * **Winter/Cloudy Mode:** Charges to a high target (e.g., 90%) if the solar forecast is low.
    * **Summer/Sunny Mode:** Charges to a low target (e.g., 30%) to leave room for excess solar energy during the day.

3.  **Profit Discharging**
    Prevents the battery from dumping energy into the grid unless the selling price offers a guaranteed profit margin (e.g., +20%) over the price you paid for charging.

4.  **Smart Hold (Price Neutral)**
    If the price is not low enough to charge, but not high enough to sell, the battery is put in "Hold" mode. This preserves the cheap energy currently in the battery for your own house consumption later.

5.  **Emergency Guard**
    Always maintains a minimum SoC (e.g., 15%). If the battery drops below this, it will force charge (unless prices are above your absolute maximum cap).

6.  **Reporting**
    Sends optional notifications (Morning & Evening) via the mobile app about the current status, planned charging slots, and financial thresholds.

---

## ⚙️ Prerequisites & Entities

To use this blueprint, you need to provide the following entities.

### Required Entities
| Input Field | Domain | Description |
| :--- | :--- | :--- |
| **Switch: Force Charge** | `switch` | The hardware switch that forces the battery to charge from the grid (or holds the charge). |
| **Sensor: Battery SoC** | `sensor` | Current State of Charge (%) of your battery. |
| **Entity: Target SoC** | `number` | The entity that sets the maximum charge limit of your battery/inverter. |
| **Sensor: Frank Energy Prices** | `sensor` | The sensor containing the price attributes (prices list). Compatible with Zonneplan/Frank integrations. |
| **Helper: Last Purchase Price** | `input_number` | **Create this helper manually.** Used to store the price at which the battery was last charged to calculate profit margins later. |

### Optional / System Entities
* **Solar Forecasts:** `sensor.solar_forecast_today` and `sensor.solar_forecast_tomorrow` (or similar).
* **Switch: Manual Boost:** An `input_boolean` helper to manually trigger a full charge immediately.
* **Notification Device:** Your mobile phone (Mobile App integration) for reports.

---

## 🧠 How it works (The Logic Flow)

Every 5 minutes, the automation runs through this priority list. It executes **only the first** matching condition:

1.  **Reporting:** Checks if it's time to send the morning or evening summary.
2.  **Boost Mode:** Is the manual boost switch ON?
    * *Action:* Charge to 100% immediately.
3.  **Emergency Charge:** Is battery < `Min Base SoC`?
    * *Action:* Force charge to prevent deep discharge (ignoring price, unless > Max Price Cap).
4.  **Smart Charging:**
    * Is the price below the **Strict Limit**?
    * OR is the price below the **Buffer Limit** (e.g., 40% cheaper than the 12h peak)?
    * *Action:* Set Target SoC & Start Charging.
5.  **Discharging:** Is the current price high enough to guarantee profit (vs. the `Last Purchase Price`)?
    * *Action:* Turn off the switch (allow discharge to grid).
6.  **Smart Hold:** Is the price "neutral" (too expensive to charge, too cheap to sell)?
    * *Action:* Turn on the switch (Hold battery) to save cheap energy for later house usage.
7.  **Default (Eco Mode):** If none of the above apply.
    * *Action:* Turn off forced charging/holding. The battery covers house consumption (Self-consumption).

---

## 🛠 Configuration Tips

* **Peak Analysis Window:** Default is `3`. This means the system looks at the 3 most expensive hours in the next 12 hours to determine the "expensive reference price".
* **Buffer Discount (%):** Default `40%`. The system will only charge if the current price is 40% lower than the calculated peak average.
* **Min. Discharge Profit (%):** Default `20%`. You will only export to the grid if you make at least 20% profit over your stored energy price.

---

*Disclaimer: This is a BETA version. Always monitor your system after installation. Use at your own risk.*