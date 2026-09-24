# Baithul Madeena

Baithul Madeena is a multi-branch real-estate ERP with AI capabilities.

## Repositories

- `backend/` — Laravel REST API
- `frontend/` — Angular application

The backend and frontend are maintained as Git submodules.

## Change Log

- **2026-09-24**: Fixed the Super Admin Users API crash caused by uncast login timestamps.
- **2026-09-24**: Fixed deployment triggers for backend and frontend submodule pointer updates.
- **2026-09-24**: Hardened SSH deployment connections with keepalive, connection retries, and bounded job timeouts.
- **2026-09-24**: Updated the application header and navigation layout.
- **2026-09-24**:
  - **macOS Spotlight Command Palette (`Ctrl + K`)**: Redesigned top navbar search into a centered floating Spotlight command palette with keyboard navigation (`↑↓`, `↵`, `ESC`), category badges, route shortcuts, mobile drawer integration, and dynamic record lookup fallbacks.
  - **About Application Page (`/app/about`)**: Implemented system overview page with Fujairah & Ajman UAE branch context, credits for engineering firm ([Desertwhales Technology](https://dwtech.vercel.app/)), lead architect ([Sanu Khan](https://www.sanukhan.dev/)), and AI platform ([ZaakiyV3RSE](https://www.zaakiy.io/)).
  - **Help Navigation Group & Dropdown**: Added dedicated Help menu in top navbar containing links for About Application, FAQ & User Guide, Terms of Use, Privacy Policy, and Legal Suite.
  - **Responsive Layout & Window Overflow Fix**: Resolved `w-screen` scrollbar overflow issue, refined responsive navigation scaling across standard screen breakpoints (`640px` to `1536px`), and elevated dropdown z-index stacking above dashboard hero cards.
  - **Zaakiy Chat UI & Searchable Comboboxes**: Modernized Zaakiy AI interface to match ERP design system, and upgraded owner/tenant agreement selection to searchable comboboxes.
- **2026-09-24**: Added the Zaakiy ERP FAQ skill and published the business user guide.
- **2026-09-24**:
  - **Datatables & UI Interactivity Audit**: Audited and upgraded all datatables across Accounts (Inward/Outward Receipts & Petty Cash), Billing (Quotations & Invoices), Maintenance (Work Orders, Vendors, & Inventory), and Administration (Branches, Users, & Audit Trail).
  - **Filters & Pagination Standardization**: Added missing search inputs, status dropdowns (draft, posted, voided, active, inactive, archived, open, in_progress, completed), priority filters, clear filters buttons, and standardized `<bm-pagination>` controls with seamless transitions and interactive row hover states.
