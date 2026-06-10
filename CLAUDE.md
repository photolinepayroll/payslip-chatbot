# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-file HTML chatbot (`index.html`) for Photoline HR payslip inquiries. Employees enter their 8-digit account number and last name to retrieve their payslip. An admin can upload payroll data via Excel file.

## Architecture

Everything is in `index.html` — no build step, no dependencies to install. It is deployed via GitHub Pages (main branch).

**Data flow:**
- Payroll data is stored in a Google Sheet with two columns: `Account No` and `Response`
- A Google Apps Script (`SHEET_URL` in the HTML) acts as the backend API
- Admin uploads Excel → HTML parses it with xlsx.js → POST to Apps Script → appended to Sheet
- Employee queries → GET to Apps Script with `?acct=xxx&lastname=yyy` → Apps Script filters and returns only matching rows

**Two-step employee auth:**
1. Employee enters account number
2. Bot asks for last name → sent to Apps Script for server-side verification against name embedded in the `Response` text

**Apps Script endpoints:**
- `GET ?acct=xxx&lastname=yyy` — returns matching payslip rows (filtered server-side)
- `GET ?action=checkPassword&password=xxx` — verifies admin password (password stored only in Apps Script)
- `POST` with `{ rows: [...] }` — saves uploaded records to the Sheet

## Deployment

Push to `main` branch → GitHub Pages auto-deploys. No build process needed.

The Google Apps Script must be redeployed (new version) whenever its code changes — saving alone is not enough.

## Key Implementation Notes

- `SHEET_URL` is publicly visible in HTML source — the Apps Script is the security boundary
- Date sorting uses `toInputDate()` to convert `MM/DD/YYYY` → `YYYY-MM-DD` before comparing
- Excel upload skips rows where column A has no digit (header detection by content, not row index)
- Payslip text in `Response` column is a structured string parsed client-side via regex (see `parsePayslip` and `parsePayslipNew`)
- `_pendingAcct` variable holds the account number between the two-step auth flow
