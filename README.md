# Order Monitoring Dashboard - User Guide

A web-based dashboard for monitoring store orders and tracking submission timeliness. This guide explains how to use the dashboard and understand the order statuses.

## Tab Overview

### 1. Orders Tab

**Purpose**: Reactive monitoring of all submitted orders.

**Description**: Displays all orders that have been submitted to the system. Each order is automatically evaluated to determine if it was received on time or late based on the display start date.

**Columns**:
- `order_id`: Unique identifier for the order
- `store_code`: Store identifier
- `store_name`: Name of the store
- `urls`: Array of image URLs
- `metadata`: JSON object containing order details including `start_day`
- `created_at`: Timestamp when the order was received
- `week_folder`: Week identifier (format: `YYYY-Wnn`, e.g., "2025-W51")
- `order_status`: Computed status (see Status Definitions below)

**Hidden by Default**: `id`, `card_id`, `store_id`, `card_title`, `status`

**Status Computation**:

The order status is calculated based on whether the order was received by the deadline:

1. **Extract Display Start Day**: From the order's `metadata.start_day` field (e.g., "Monday", "Tuesday")
2. **Calculate Deadline**: 3 days before the display start day
3. **Compare Submission Time**: Check if `created_at` is before or after the deadline
4. **Assign Status**:
   - **Received | On Time** (green): Order submitted on or before the deadline
   - **Received | Late** (yellow): Order submitted after the deadline
   - **Unknown** (gray): Missing required data (start_day, created_at, or week_folder)

**Example**:
- Display starts on Friday
- Deadline is Tuesday 11:59:59 PM (3 days before Friday)
- Order submitted on Monday → **Received | On Time** ✓
- Order submitted on Wednesday → **Received | Late** ⚠️

**Filters**: All Status, On Time, Late, Unknown

---

### 2. Expected Orders Tab

**Purpose**: Proactive monitoring of stores that should submit orders for the current week.

**Description**: Automatically calculates which stores are expected to submit orders based on their configuration in `store_branches`. Shows whether each expected order has been received and if it was on time.

**Columns**:
- `store_code`: Store identifier
- `store_name`: Name of the store
- `expected_receive_day`: Day of the week when the order should be received
- `display_duration`: How often the store submits (7 days = weekly, 14 days = bi-weekly)
- `week_folder`: Current week identifier
- `order_received_date`: Timestamp when the order was received (null if not received)
- `expectation_status`: Computed status (see Status Definitions below)

**Hidden by Default**: `id`

**Status Computation**:

The expectation status is calculated for the current week based on store configuration:

1. **Identify Active Stores**: Filter stores where `is_active = true`
2. **Handle Bi-weekly Stores**: Skip stores with 14-day duration if they submitted last week
3. **Check Current Week Submissions**: Look for orders matching `store_code` and current `week_folder`
4. **Calculate Deadline**: Based on the store's `expected_order_receive_day`
5. **Assign Status**:
   - **Received | On Time** (green): Order received on or before the expected day
   - **Received | Late** (yellow): Order received after the expected day
   - **Did Not Receive** (red): No order received for the current week

**Bi-weekly Logic**:
- Stores with `display_duration_days = 14` submit every other week
- If a bi-weekly store submitted last week (previous week_folder), they are excluded from current week expectations
- This prevents false "Did Not Receive" alerts for bi-weekly stores in their off-week

**Example**:
- Store A (weekly): Expected every week
  - Current week: 2025-W51
  - Expected receive day: Tuesday
  - Order received Monday → **Received | On Time** ✓
  
- Store B (bi-weekly): Expected every other week
  - Current week: 2025-W51
  - Previous submission: 2025-W50
  - Status: Not expected this week (excluded from list)
  
- Store C (weekly): Expected every week
  - Current week: 2025-W51
  - Expected receive day: Monday
  - No order received → **Did Not Receive** ✗

**Filters**: All Status, Did Not Receive, Received | On Time, Received | Late

---

### 3. Store Branches Tab

**Purpose**: Configuration reference for all store locations.

**Description**: Displays the master list of all store branches with their configuration settings for order submissions and display schedules.

**Columns**:
- `store_code`: Unique store identifier
- `store_name`: Name of the store
- `display_start_day`: Day of the week when display period begins
- `display_end_day`: Day of the week when display period ends
- `display_duration_days`: Number of days between submissions (7 = weekly, 14 = bi-weekly)
- `expected_order_receive_day`: Day of the week when orders should be submitted
- `resolution`: Display resolution requirements
- `min_items_qty`: Minimum number of items required

**Hidden by Default**: `id`, `store_id`, `metadata`, `is_active`, `created_at`, `updated_at`, `share_group_id`

**No Status Computation**: This is a reference table showing configuration only.

---

## Status Definitions

### Orders Tab Statuses

| Status | Color | Meaning | Computation |
|--------|-------|---------|-------------|
| **Received \| On Time** | Green | Order submitted before deadline | `created_at <= (start_day - 3 days)` |
| **Received \| Late** | Yellow | Order submitted after deadline | `created_at > (start_day - 3 days)` |
| **Unknown** | Gray | Missing data to calculate status | Missing `start_day`, `created_at`, or `week_folder` |

### Expected Orders Tab Statuses

| Status | Color | Meaning | Computation |
|--------|-------|---------|-------------|
| **Received \| On Time** | Green | Order received on or before expected day | Order exists AND `created_at <= expected_receive_day` |
| **Received \| Late** | Yellow | Order received after expected day | Order exists AND `created_at > expected_receive_day` |
| **Did Not Receive** | Red | No order received for current week | No matching order for current `week_folder` |

---

## Key Concepts

### Week Folder Format

Orders and expectations are organized by week using the format: `YYYY-Wnn`

- `YYYY`: Four-digit year
- `W`: Week indicator
- `nn`: Two-digit week number (01-52)

**Examples**: `2025-W51`, `2025-W52`, `2026-W01`

### Display Duration

Stores can be configured with different submission frequencies:
- **7 days (Weekly)**: Store submits orders every week
- **14 days (Bi-weekly)**: Store submits orders every other week

### Deadline Calculation

For the **Orders tab**, the deadline is calculated as:
```
deadline = start_day - 3 days at 23:59:59
```

For the **Expected Orders tab**, the deadline is:
```
deadline = expected_receive_day at 23:59:59
```

Both use the Monday of the week specified in `week_folder` as the reference point for calculating specific dates.

---

## How to Use the Dashboard

### Navigation

The dashboard has three tabs at the top:
1. **Orders** - View all submitted orders (default view)
2. **Expected Orders** - See which stores should submit this week
3. **Store Branches** - Reference list of all stores and their settings

Click any tab to switch between views.

### Search and Filter

- **Search Box**: Type to search across all visible columns (store names, codes, dates, etc.)
- **Status Filter Dropdown**: Filter by order status (On Time, Late, Did Not Receive, etc.)
- **Results Update**: Search and filters update the table in real-time

### Sorting

- Click any column header to sort by that column
- Click again to reverse the sort order (ascending ↑ / descending ↓)
- The arrow indicator shows current sort direction

### Column Management

- Click the **"Columns"** button (top right) to show/hide specific columns
- Eye icon (👁) next to each column header also hides that column
- Checkbox list shows all available columns and which are currently visible
- Customize your view by hiding columns you don't need

### Viewing Cell Details

- Hover over any cell to see the **"..."** menu button
- Click the menu to access:
  - **Copy Cell**: Copy the cell content to clipboard
  - **View Cell**: Open a modal to see the full content (useful for long text or JSON data)

### Pagination

- The table shows 25 items per page
- Use **Previous** and **Next** buttons at the bottom to navigate pages
- Current page number and total pages are displayed between the buttons

### Refreshing Data

- Reload the browser page to fetch the latest data from the database
- The dashboard shows a loading spinner while data is being fetched

---

## Need Help?

For questions or issues with the dashboard, contact the N-Compass TV development team.
