---
layout: page
title: Portfolio Overview
permalink: /portfolio/
---

## Portfolio Summary

This page provides an overview of all current and past investments across all wallets.

---

## Primary Wallet (Black)

Growth-focused investments with higher risk/reward potential.


---

## Cash Flow Wallet (Green)

Income-generating investments focused on generating cash flow.

### Portfolio Performance

[View Source Spreadsheet](https://docs.google.com/spreadsheets/d/e/2PACX-1vQdW4--3KMPl6vSJGFY4BdzNxJgbZFMPnfGYSqS7AEox19YzmYQGo5wvKHupYOS1vTO2J6F6oksqzry/pub?gid=1440929963&single=true)

<div id="portfolio-table-container">
  <p>Loading portfolio data...</p>
</div>

<script>
  // Fetch and display CSV data from Google Sheets
  const csvUrl = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vQdW4--3KMPl6vSJGFY4BdzNxJgbZFMPnfGYSqS7AEox19YzmYQGo5wvKHupYOS1vTO2J6F6oksqzry/pub?gid=1440929963&single=true&output=csv';

  // Parse CSV handling quoted fields with commas
  function parseCSV(text) {
    const rows = [];
    let currentRow = [];
    let currentCell = '';
    let insideQuotes = false;

    for (let i = 0; i < text.length; i++) {
      const char = text[i];
      const nextChar = text[i + 1];

      if (char === '"') {
        if (insideQuotes && nextChar === '"') {
          currentCell += '"';
          i++;
        } else {
          insideQuotes = !insideQuotes;
        }
      } else if (char === ',' && !insideQuotes) {
        currentRow.push(currentCell.trim());
        currentCell = '';
      } else if ((char === '\n' || char === '\r') && !insideQuotes) {
        if (char === '\r' && nextChar === '\n') {
          i++;
        }
        if (currentCell || currentRow.length > 0) {
          currentRow.push(currentCell.trim());
          rows.push(currentRow);
          currentRow = [];
          currentCell = '';
        }
      } else {
        currentCell += char;
      }
    }

    if (currentCell || currentRow.length > 0) {
      currentRow.push(currentCell.trim());
      rows.push(currentRow);
    }

    return rows;
  }

  fetch(csvUrl)
    .then(response => response.text())
    .then(data => {
      const rows = parseCSV(data);
      const headers = rows[0];

      let tableHTML = '<table style="width: 100%; border-collapse: collapse; margin-top: 20px;">';

      // Add header row
      tableHTML += '<thead><tr>';
      headers.forEach(header => {
        tableHTML += `<th style="border: 1px solid #ddd; padding: 12px; background-color: #f2f2f2; text-align: left;">${header}</th>`;
      });
      tableHTML += '</tr></thead>';

      // Add data rows
      tableHTML += '<tbody>';
      for (let i = 1; i < rows.length; i++) {
        if (rows[i].length > 0 && rows[i].some(cell => cell)) {
          tableHTML += '<tr>';
          rows[i].forEach(cell => {
            tableHTML += `<td style="border: 1px solid #ddd; padding: 12px;">${cell}</td>`;
          });
          tableHTML += '</tr>';
        }
      }
      tableHTML += '</tbody></table>';

      document.getElementById('portfolio-table-container').innerHTML = tableHTML;
    })
    .catch(error => {
      document.getElementById('portfolio-table-container').innerHTML = '<p style="color: red;">Error loading portfolio data. Please refresh the page.</p>';
      console.error('Error fetching CSV:', error);
    });
</script>