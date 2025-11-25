# Smart Battery Charge Blueprint for Home Assistant

Automatically charge your battery at the cheapest times based on electricity price and solar forecast.

## Features
- Dynamically calculates day and night average prices
- Adjusts charging based on battery SOC
- Respects solar forecast to avoid unnecessary grid usage
- Configurable loading windows

## Usage
1. Import the blueprint into Home Assistant using the raw GitHub URL.
2. Configure the following entities:
   - Battery SOC sensor
   - Reserve mode switch
   - Electricity price sensor
   - Solar forecast sensor (optional)
3. Set any configurable parameters such as load start/end hours or minimum solar kWh.
