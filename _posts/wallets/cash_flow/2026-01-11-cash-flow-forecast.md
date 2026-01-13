---
layout: post
title: Cash Flow Forecast
subtitle: Projected returns and cash flow analysis
date: 2026-01-11 20:30:00
cover-img: assets/img/cash_flow/CashFlow Thumbnail.jpg
thumbnail-img: assets/img/cash_flow/CashFlow Thumbnail.jpg
tags: cash-flow-wallet
---

## Summary Metrics

<div id="metrics-container" style="display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; margin: 30px 0;">
  <div style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%); padding: 25px; border-radius: 12px; text-align: center; color: white; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
    <div style="font-size: 14px; opacity: 0.9; margin-bottom: 8px;">Amount Invested</div>
    <div id="invested-value" style="font-size: 42px; font-weight: bold;">Loading...</div>
  </div>

  <div style="background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%); padding: 25px; border-radius: 12px; text-align: center; color: white; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
    <div style="font-size: 14px; opacity: 0.9; margin-bottom: 8px;">Anticipated Cash Flow (Low)</div>
    <div id="cashflow-low-value" style="font-size: 42px; font-weight: bold;">Loading...</div>
  </div>

  <div style="background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%); padding: 25px; border-radius: 12px; text-align: center; color: white; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
    <div style="font-size: 14px; opacity: 0.9; margin-bottom: 8px;">Anticipated Cash Flow (High)</div>
    <div id="cashflow-high-value" style="font-size: 42px; font-weight: bold;">Loading...</div>
  </div>
</div>

---

## Forecast Details

<div id="forecast-table">Loading forecast data...</div>

---

## Data Sources

View the live data directly from Google Sheets:

- [Cash Flow Forecast](https://docs.google.com/spreadsheets/d/e/2PACX-1vQdW4--3KMPl6vSJGFY4BdzNxJgbZFMPnfGYSqS7AEox19YzmYQGo5wvKHupYOS1vTO2J6F6oksqzry/pubhtml?gid=36108301&single=true)

---

_Last updated: {{ page.date | date: "%B %-d, %Y" }}_

<script>
// CSV URL
const FORECAST_CSV_URL = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vQdW4--3KMPl6vSJGFY4BdzNxJgbZFMPnfGYSqS7AEox19YzmYQGo5wvKHupYOS1vTO2J6F6oksqzry/pub?gid=36108301&single=true&output=csv';

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

// Format currency
function formatCurrency(value) {
  // Handle both numbers and strings
  let num;
  if (typeof value === 'number') {
    num = value;
  } else if (typeof value === 'string') {
    num = parseFloat(value.replace(/[$,]/g, ''));
  } else {
    num = 0;
  }

  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 0,
    maximumFractionDigits: 0
  }).format(num);
}

// Load and display forecast data
async function loadForecastData() {
  try {
    console.log('Fetching forecast CSV from:', FORECAST_CSV_URL);
    const response = await fetch(FORECAST_CSV_URL);
    console.log('Response status:', response.status);

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const text = await response.text();
    console.log('Forecast CSV text:', text);

    const { headers, rows } = parseCSV(text);
    console.log('Parsed headers:', headers);
    console.log('Parsed rows:', rows);

    // Create table HTML
    let tableHTML = '<div style="overflow-x: auto;"><table style="width: 100%; border-collapse: collapse; font-size: 14px;">';
    tableHTML += '<thead><tr>';
    headers.forEach(header => {
      tableHTML += `<th style="border: 1px solid #ddd; padding: 8px; background-color: #f2f2f2; text-align: left; white-space: nowrap;">${header}</th>`;
    });
    tableHTML += '</tr></thead><tbody>';

    let totalInvested = 0;
    let totalCashFlowLow = 0;
    let totalCashFlowHigh = 0;

    rows.forEach(row => {
      tableHTML += '<tr>';
      headers.forEach(header => {
        tableHTML += `<td style="border: 1px solid #ddd; padding: 8px;">${row[header]}</td>`;
      });
      tableHTML += '</tr>';

      // Calculate totals
      const depositAmount = parseFloat((row['Deposit Amount'] || '').replace(/[$,]/g, ''));
      const cashFlowLow = parseFloat((row['Annualized Anticipated Cash Flow (Low)'] || '').replace(/[$,]/g, ''));
      const cashFlowHigh = parseFloat((row['Annualized Anticipated Cash Flow (High)'] || '').replace(/[$,]/g, ''));

      if (!isNaN(depositAmount)) {
        totalInvested += depositAmount;
      }
      if (!isNaN(cashFlowLow)) {
        totalCashFlowLow += cashFlowLow;
      }
      if (!isNaN(cashFlowHigh)) {
        totalCashFlowHigh += cashFlowHigh;
      }
    });

    tableHTML += '</tbody></table></div>';
    document.getElementById('forecast-table').innerHTML = tableHTML;

    // Update summary metrics
    document.getElementById('invested-value').textContent = formatCurrency(totalInvested);
    document.getElementById('cashflow-low-value').textContent = formatCurrency(totalCashFlowLow);
    document.getElementById('cashflow-high-value').textContent = formatCurrency(totalCashFlowHigh);

    console.log('Total Invested:', totalInvested);
    console.log('Total Cash Flow (Low):', totalCashFlowLow);
    console.log('Total Cash Flow (High):', totalCashFlowHigh);

  } catch (error) {
    console.error('Error loading forecast data:', error);
    console.error('Error message:', error.message);
    console.error('Error stack:', error.stack);
    document.getElementById('forecast-table').innerHTML = `<p style="color: red;">Error loading forecast data: ${error.message}</p>`;
    document.getElementById('invested-value').textContent = 'Error';
    document.getElementById('cashflow-low-value').textContent = 'Error';
    document.getElementById('cashflow-high-value').textContent = 'Error';
  }
}

// Load data on page load
document.addEventListener('DOMContentLoaded', function() {
  loadForecastData();
});
</script>
