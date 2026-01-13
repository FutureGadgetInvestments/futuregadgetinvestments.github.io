---
layout: post
title: "Position 1 - Pendle yoUSD"
subtitle: "Zero-coupon bond position generating yield from optimized USD"
date: 2026-01-11 21:00:00
cover-img: assets/img/cash_flow/pendle-yousd-cover.jpg
thumbnail-img: assets/img/cash_flow/pendle-yousd-thumb.jpg
tags: cash-flow-wallet
---

<div style="text-align: center; margin: 30px 0;">
  <img src="/assets/img/cash_flow/pendle-yousd-position.jpg" alt="Pendle yoUSD Position" style="max-width: 100%; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
</div>

---

## Current Position

<div id="position-table">Loading position data...</div>

---

## Market Research

### What is yoUSD?

TODO
### How It Generates Yield

TODO

---

## Risks

TODO

---

## Links & Resources

### Position Management
- [Pendle Market (Base Chain)](https://app.pendle.finance/trade/markets/0xa679ce6d07cbe579252f0f9742fc73884b1c611c/swap?view=pt&chain=base&tab=info) - Trade and manage position
- [Position Data (Live)](https://docs.google.com/spreadsheets/d/e/2PACX-1vQdW4--3KMPl6vSJGFY4BdzNxJgbZFMPnfGYSqS7AEox19YzmYQGo5wvKHupYOS1vTO2J6F6oksqzry/pubhtml?gid=2134666800&single=true) - View live position data

### Token Information
- [yoUSD on CoinGecko](https://www.coingecko.com/en/coins/yield-optimizer-usd) - Price, charts, and market data
- [yoUSD Documentation](https://www.yo.xyz/whitepaper) - Whitepapers

---

_Last updated: {{ page.date | date: "%B %-d, %Y" }}_

<script>
// CSV URL
const POSITION_CSV_URL = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vQdW4--3KMPl6vSJGFY4BdzNxJgbZFMPnfGYSqS7AEox19YzmYQGo5wvKHupYOS1vTO2J6F6oksqzry/pub?gid=2134666800&single=true&output=csv';

// Parse CSV text to array (handles quoted fields with commas)
function parseCSV(text) {
  const lines = text.trim().split('\n');

  // Parse a single CSV line respecting quotes
  function parseLine(line) {
    const values = [];
    let current = '';
    let inQuotes = false;

    for (let i = 0; i < line.length; i++) {
      const char = line[i];
      const nextChar = line[i + 1];

      if (char === '"') {
        if (inQuotes && nextChar === '"') {
          // Escaped quote
          current += '"';
          i++; // Skip next quote
        } else {
          // Toggle quote state
          inQuotes = !inQuotes;
        }
      } else if (char === ',' && !inQuotes) {
        // Field separator
        values.push(current.trim());
        current = '';
      } else {
        current += char;
      }
    }

    // Add last field
    values.push(current.trim());

    // Remove surrounding quotes from all values
    return values.map(v => v.replace(/^["']|["']$/g, ''));
  }

  const headers = parseLine(lines[0]);
  const rows = [];

  for (let i = 1; i < lines.length; i++) {
    const values = parseLine(lines[i]);
    const row = {};
    headers.forEach((header, index) => {
      row[header] = values[index] || '';
    });
    rows.push(row);
  }

  return { headers, rows };
}

// Load and display position data
async function loadPositionData() {
  try {
    console.log('Fetching position CSV from:', POSITION_CSV_URL);
    const response = await fetch(POSITION_CSV_URL);
    console.log('Response status:', response.status);

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const text = await response.text();
    console.log('Position CSV text:', text);

    const { headers, rows } = parseCSV(text);
    console.log('Parsed headers:', headers);
    console.log('Parsed rows:', rows);

    // Create table HTML
    let tableHTML = '<table style="width: 100%; border-collapse: collapse;">';
    tableHTML += '<thead><tr>';
    headers.forEach(header => {
      tableHTML += `<th style="border: 1px solid #ddd; padding: 12px; background-color: #f2f2f2; text-align: left;">${header}</th>`;
    });
    tableHTML += '</tr></thead><tbody>';

    rows.forEach(row => {
      tableHTML += '<tr>';
      headers.forEach(header => {
        tableHTML += `<td style="border: 1px solid #ddd; padding: 12px;">${row[header]}</td>`;
      });
      tableHTML += '</tr>';
    });

    tableHTML += '</tbody></table>';
    document.getElementById('position-table').innerHTML = tableHTML;

  } catch (error) {
    console.error('Error loading position data:', error);
    console.error('Error message:', error.message);
    console.error('Error stack:', error.stack);
    document.getElementById('position-table').innerHTML = `<p style="color: red;">Error loading position data: ${error.message}</p>`;
  }
}

// Load data on page load
document.addEventListener('DOMContentLoaded', function() {
  loadPositionData();
});
</script>
