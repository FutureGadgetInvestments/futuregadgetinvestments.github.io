---
layout: post
title: Cash Flow Analysis
subtitle: Performance tracking for cash flow wallet positions
date: 2026-01-11 18:30:00
cover-img: assets/img/cash_flow/CashFlow Thumbnail.jpg
thumbnail-img: assets/img/cash_flow/CashFlow Thumbnail.jpg
tags: cash-flow-wallet
---

## Key Metrics

<div id="metrics-container" style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 20px; margin: 30px 0;">
  <div style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%); padding: 25px; border-radius: 12px; text-align: center; color: white; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
    <div style="font-size: 14px; opacity: 0.9; margin-bottom: 8px;">Amount Invested</div>
    <div id="invested-value" style="font-size: 42px; font-weight: bold;">Loading...</div>
  </div>

  <div style="background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%); padding: 25px; border-radius: 12px; text-align: center; color: white; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
    <div style="font-size: 14px; opacity: 0.9; margin-bottom: 8px;">Current Value</div>
    <div id="current-value" style="font-size: 42px; font-weight: bold;">Loading...</div>
  </div>

  <div style="background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%); padding: 25px; border-radius: 12px; text-align: center; color: white; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
    <div style="font-size: 14px; opacity: 0.9; margin-bottom: 8px;">Total Withdrawn</div>
    <div id="withdrawn-value" style="font-size: 42px; font-weight: bold;">Loading...</div>
  </div>

  <div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); padding: 25px; border-radius: 12px; text-align: center; color: white; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
    <div style="font-size: 14px; opacity: 0.9; margin-bottom: 8px;">Internal Rate of Return</div>
    <div id="irr-value" style="font-size: 42px; font-weight: bold;">Loading...</div>
  </div>
</div>

---

## Transaction History

<div id="transaction-table">Loading transaction data...</div>

<div id="transaction-summary" style="margin-top: 20px;"></div>

---

## IRR Breakdown

<div id="irr-table">Loading IRR data...</div>

---

## Data Sources

View the live data directly from Google Sheets:

- [Summary Metrics](https://docs.google.com/spreadsheets/d/e/2PACX-1vQdW4--3KMPl6vSJGFY4BdzNxJgbZFMPnfGYSqS7AEox19YzmYQGo5wvKHupYOS1vTO2J6F6oksqzry/pubhtml?gid=706001030&single=true)
- [Transaction History](https://docs.google.com/spreadsheets/d/e/2PACX-1vQdW4--3KMPl6vSJGFY4BdzNxJgbZFMPnfGYSqS7AEox19YzmYQGo5wvKHupYOS1vTO2J6F6oksqzry/pubhtml?gid=0&single=true)
- [IRR Data](https://docs.google.com/spreadsheets/d/e/2PACX-1vQdW4--3KMPl6vSJGFY4BdzNxJgbZFMPnfGYSqS7AEox19YzmYQGo5wvKHupYOS1vTO2J6F6oksqzry/pubhtml?gid=1709105802&single=true)

---

_Last updated: {{ page.date | date: "%B %-d, %Y" }}_

<script>
// CSV URLs
const TRANSACTION_CSV_URL = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vQdW4--3KMPl6vSJGFY4BdzNxJgbZFMPnfGYSqS7AEox19YzmYQGo5wvKHupYOS1vTO2J6F6oksqzry/pub?gid=0&single=true&output=csv';
const IRR_CSV_URL = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vQdW4--3KMPl6vSJGFY4BdzNxJgbZFMPnfGYSqS7AEox19YzmYQGo5wvKHupYOS1vTO2J6F6oksqzry/pub?gid=1709105802&single=true&output=csv';
const SUMMARY_CSV_URL = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vQdW4--3KMPl6vSJGFY4BdzNxJgbZFMPnfGYSqS7AEox19YzmYQGo5wvKHupYOS1vTO2J6F6oksqzry/pub?gid=706001030&single=true&output=csv';

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
    minimumFractionDigits: 2
  }).format(num);
}

// Load and display summary metrics
async function loadSummaryData() {
  try {
    console.log('Fetching summary CSV from:', SUMMARY_CSV_URL);
    const response = await fetch(SUMMARY_CSV_URL);
    console.log('Response status:', response.status);

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const text = await response.text();
    console.log('Summary CSV text:', text);

    const { headers, rows } = parseCSV(text);
    console.log('Parsed headers:', headers);
    console.log('Parsed rows:', rows);

    if (rows.length > 0) {
      const data = rows[0];
      console.log('First row data:', data);

      // Get values directly from CSV
      const deposited = data['Deposited'] || data['deposited'] || '$0.00';
      const currentValue = data['Current Value'] || data['current value'] || '$0.00';
      const totalWithdrawn = data['Total Withdrawn'] || data['total withdrawn'] || '$0.00';

      console.log('Deposited:', deposited);
      console.log('Current Value:', currentValue);
      console.log('Total Withdrawn:', totalWithdrawn);

      // Update metrics cards
      document.getElementById('invested-value').textContent = deposited;
      document.getElementById('current-value').textContent = currentValue;
      document.getElementById('withdrawn-value').textContent = totalWithdrawn;

      // Calculate gain for summary section only
      const depositedNum = parseFloat(deposited.replace(/[$,]/g, ''));
      const currentNum = parseFloat(currentValue.replace(/[$,]/g, ''));
      const gain = currentNum - depositedNum;
      const gainPercent = depositedNum > 0 ? ((gain / depositedNum) * 100).toFixed(2) : '0.00';

      // Update transaction summary
      document.getElementById('transaction-summary').innerHTML = `
        <p><strong>Total Invested:</strong> ${deposited}</p>
        <p><strong>Current Value:</strong> ${currentValue}</p>
        <p><strong>Total Withdrawn:</strong> ${totalWithdrawn}</p>
        <p><strong>Gain:</strong> ${formatCurrency(gain)} (+${gainPercent}%)</p>
      `;
    } else {
      console.error('No rows found in summary data');
      document.getElementById('invested-value').textContent = 'No Data';
      document.getElementById('current-value').textContent = 'No Data';
      document.getElementById('withdrawn-value').textContent = 'No Data';
    }

  } catch (error) {
    console.error('Error loading summary data:', error);
    console.error('Error message:', error.message);
    console.error('Error stack:', error.stack);
    document.getElementById('invested-value').textContent = 'Error';
    document.getElementById('current-value').textContent = 'Error';
    document.getElementById('withdrawn-value').textContent = 'Error';
    document.getElementById('transaction-summary').innerHTML = `<p style="color: red;">Error loading summary data: ${error.message}</p>`;
  }
}

// Load and display transaction data
async function loadTransactionData() {
  try {
    const response = await fetch(TRANSACTION_CSV_URL);
    const text = await response.text();
    const { headers, rows } = parseCSV(text);

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
    document.getElementById('transaction-table').innerHTML = tableHTML;

  } catch (error) {
    console.error('Error loading transaction data:', error);
    document.getElementById('transaction-table').innerHTML = '<p style="color: red;">Error loading transaction data. Please try again later.</p>';
  }
}

// Load and display IRR data
async function loadIRRData() {
  try {
    const response = await fetch(IRR_CSV_URL);
    const text = await response.text();
    const { headers, rows } = parseCSV(text);

    // Create table HTML
    let tableHTML = '<table style="width: 100%; border-collapse: collapse;">';
    tableHTML += '<thead><tr>';
    headers.forEach(header => {
      tableHTML += `<th style="border: 1px solid #ddd; padding: 12px; background-color: #f2f2f2; text-align: left;">${header}</th>`;
    });
    tableHTML += '</tr></thead><tbody>';

    let irrValue = '';

    rows.forEach(row => {
      tableHTML += '<tr>';
      headers.forEach(header => {
        tableHTML += `<td style="border: 1px solid #ddd; padding: 12px;">${row[header]}</td>`;
      });
      tableHTML += '</tr>';

      // Get IRR value
      if (row['Deposit/Withdrawal']) {
        irrValue = row['Deposit/Withdrawal'];
      }
    });

    tableHTML += '</tbody></table>';
    document.getElementById('irr-table').innerHTML = tableHTML;

    // Update IRR metric
    document.getElementById('irr-value').textContent = irrValue;

  } catch (error) {
    console.error('Error loading IRR data:', error);
    document.getElementById('irr-table').innerHTML = '<p style="color: red;">Error loading IRR data. Please try again later.</p>';
  }
}

// Load all data on page load
document.addEventListener('DOMContentLoaded', function() {
  loadSummaryData();
  loadTransactionData();
  loadIRRData();
});
</script>
