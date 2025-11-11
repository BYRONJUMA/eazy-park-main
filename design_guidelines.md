# EAZY PARK Design Guidelines

## Design Approach
**Hybrid Approach:** Combining Material Design's clarity for data-dense interfaces with inspiration from modern booking platforms (Airbnb, Booking.com) for the reservation flow. Focus on efficiency, instant comprehension of availability status, and frictionless booking experience.

## Core Design Principles
1. **Status-First Design:** Availability status must be instantly recognizable through visual hierarchy
2. **Progressive Disclosure:** Show essential info first, details on demand
3. **Mobile-First Responsiveness:** Optimized for on-the-go parking searches
4. **Trust & Confirmation:** Clear feedback at every transaction step

---

## Typography System

**Font Stack:** Inter (via Google Fonts CDN)
- **Hero/Headers (H1):** text-4xl md:text-5xl, font-bold, tracking-tight
- **Section Headers (H2):** text-2xl md:text-3xl, font-semibold
- **Card Titles (H3):** text-xl font-semibold
- **Body Text:** text-base, font-normal, leading-relaxed
- **Small Text/Labels:** text-sm, font-medium
- **Micro Text (prices, metadata):** text-xs, font-normal

---

## Spacing & Layout System

**Tailwind Units:** Consistently use 4, 6, 8, 12, 16, 20, 24 for spacing
- **Component padding:** p-4 (mobile), p-6 (tablet), p-8 (desktop)
- **Section spacing:** py-12 (mobile), py-16 (tablet), py-24 (desktop)
- **Card gaps:** gap-4 (mobile), gap-6 (desktop)
- **Container max-widths:** max-w-7xl for main content, max-w-md for forms/modals

**Grid Systems:**
- **Area/Lot Cards:** grid-cols-1 md:grid-cols-2 lg:grid-cols-3, gap-6
- **Slot Grid:** grid-cols-2 sm:grid-cols-4 md:grid-cols-5 lg:grid-cols-10, gap-2
- **Admin Dashboard:** grid-cols-1 lg:grid-cols-3 for stats/overview

---

## Component Specifications

### Landing Page
- **Hero Section (60vh):** Clean centered search with city/area dropdown + prominent CTA, subtle gradient overlay on background image of organized parking lot
- **Area Cards:** Horizontal layout with small thumbnail, area name, city, "X lots available" count, right arrow
- **Search Bar:** Large input with magnifying glass icon, rounded-xl, shadow-lg

### Slot Availability Grid
- **Slot Cells:** Square aspect-ratio buttons, rounded-lg, centered slot identifier (e.g., "A1")
- **Status Indicators:**
  - Available: Border style, checkmark icon top-right
  - Reserved: Slightly muted with lock icon
  - Out of Service: Diagonal stripe pattern (CSS), "X" icon
- **Hover State (available only):** Subtle scale transform (scale-105), shadow increase

### Reservation Flow (Sidebar/Modal)
- **Slide-in Panel (Desktop):** w-96, fixed right, full-height, shadow-2xl, z-50
- **Full-screen Modal (Mobile):** Slide up from bottom
- **Content Structure:**
  1. Slot info header with lot name, slot ID, hourly rate
  2. DateTime pickers (start time, duration dropdown in 0.25hr increments)
  3. Price summary box with breakdown (hours × rate = total), prominent border
  4. Phone numbers input (allow multiple with "+ Add Number" button)
  5. Payment method radio cards with icons (Mpesa, PayPal, Bank)
  6. Sticky footer with "Proceed to Payment" CTA (w-full, py-4)

### Payment Checkout
- **Mpesa Flow:** Center card showing phone prompt, animated loading spinner, step indicators (1. Initiate → 2. Approve on phone → 3. Confirm)
- **PayPal:** Embed button, maintain brand guidelines
- **Bank Transfer:** Display account details in copyable fields, upload receipt button, "Pending Admin Approval" badge

### Confirmation Page
- **QR Code:** Large centered display (256x256px), border, subtle shadow
- **Reservation Details Card:** Organized list with icons (calendar, clock, location, money), clear label-value pairs
- **Success Animation:** Checkmark icon with subtle bounce-in
- **SMS Status:** "Confirmation sent to X numbers" with phone list

### Admin Dashboard
- **Top Stats Bar:** 4-column grid showing total reservations today, pending payments, available slots, revenue (card layout with icon, number, label)
- **Management Forms:** Single-column, clear field labels above inputs, sections separated by horizontal dividers, "Add" buttons with plus icon
- **Reservations Table:** Striped rows, status badges (pending/paid/failed), sortable columns, search filter at top
- **Manual Confirmation Modal:** Show bank reference, user details, amount, "Confirm Payment" danger-style button

### Navigation
- **Public Header:** Logo left, area/login links right, mobile hamburger
- **Admin Sidebar (Desktop):** Fixed left, w-64, vertical nav with icons
- **Admin Top Bar (Mobile):** Sticky header with menu toggle

---

## Status & Feedback Components

**Status Badges:**
- Paid: Green badge with checkmark
- Pending: Yellow/amber with clock icon
- Failed: Red with X icon
- Cancelled: Gray with dash

**Toasts:** Fixed top-right, slide-in animation, auto-dismiss in 5s, dismiss button, icon + message + optional action button

**Loading States:** Skeleton screens for slot grids, spinner for payment processing, progress bars for multi-step flows

---

## Form Design

**Input Fields:** 
- Border style, rounded-md, focus ring, px-4 py-3
- Labels: text-sm font-medium, mb-2
- Error states: red border, error message text-sm text-red-600 below
- Required fields: asterisk after label

**Buttons:**
- **Primary CTA:** w-full or px-8, py-3, rounded-lg, font-semibold, with appropriate hover/active states
- **Secondary:** Border style with transparent background
- **Icon Buttons:** Square, p-2, for actions like delete/edit

**Dropdowns/Selects:** Match input styling, chevron icon

---

## Images

**Hero Image:** Full-width background image of modern, organized parking facility (aerial view showing clear slot markings), 60vh height, subtle gradient overlay for text readability

**Lot Card Thumbnails:** Small rectangular images (aspect-ratio-16/9) showing lot exterior or unique features, rounded-lg

**Empty States:** Simple illustrations for "No lots available" or "No reservations yet" scenarios

---

## Responsive Breakpoints

- **Mobile First:** All base styles optimized for 375px+
- **Tablet (md:):** 768px - expand grids to 2 columns, show more info
- **Desktop (lg:):** 1024px - full multi-column layouts, sidebars
- **Slot Grid:** Maintain touch-friendly sizing on mobile (min 44px tap targets)