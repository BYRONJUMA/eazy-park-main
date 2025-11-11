# EAZY PARK - Smart Parking Booking System

## Overview

EAZY PARK is a full-stack parking booking platform that enables users to discover parking areas, view real-time slot availability, and complete reservations through multiple payment methods. The system features concurrency control to prevent double-booking, automated cleanup of expired reservations, SMS notifications, and QR code generation for parking entry. Administrators can manage parking infrastructure (areas, lots, slots) and monitor all reservations through a dedicated dashboard.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend Architecture

**Framework**: React with Vite for fast development and optimized production builds.

**Routing**: Wouter (lightweight client-side routing) with key routes:
- Public: Home, AreaPage, LotPage, PaymentPage, ConfirmationPage
- Admin: AdminLogin, AdminDashboard

**State Management**: TanStack Query (React Query) for server state management with aggressive caching strategies (staleTime: Infinity) to reduce redundant API calls.

**UI Components**: Shadcn UI component library built on Radix UI primitives, providing accessible and customizable components with Tailwind CSS styling.

**Design System**: 
- Typography: Inter font family via Google Fonts
- Spacing: Tailwind's 4px-based scale (4, 6, 8, 12, 16, 20, 24)
- Layout: Mobile-first responsive design with CSS Grid for card layouts
- Theme: Custom HSL-based color system with CSS variables for light/dark mode support

**Rationale**: React Query eliminates boilerplate for data fetching, caching, and synchronization. Wouter keeps bundle size small compared to React Router. Shadcn UI provides production-ready accessible components without the weight of a full component library.

### Backend Architecture

**Runtime**: Node.js with Express framework.

**Language**: TypeScript for type safety across the full stack.

**Data Storage**: In-memory storage (MemStorage class implementing IStorage interface). This design allows easy swapping to a database-backed implementation by creating a new class that implements the same IStorage interface.

**Authentication**: 
- JWT tokens with 7-day expiration
- bcryptjs for password hashing
- Middleware-based auth (authMiddleware for authentication, adminMiddleware for role checking)

**API Design**: RESTful endpoints organized by resource:
- `/api/areas` - Parking areas
- `/api/lots` - Parking lots
- `/api/slots` - Individual parking slots
- `/api/reservations` - User bookings
- `/api/payments` - Payment processing
- `/api/admin/*` - Admin-only endpoints

**Concurrency Control**: 
- 10-minute slot locking during checkout prevents simultaneous bookings
- Transaction-like availability checks (checkSlotAvailability) before reservation creation
- Optimistic locking pattern where reservations are created as "pending" and updated to "paid" after payment

**Background Jobs**: Node-cron runs every 5 minutes to clean up expired pending reservations that have exceeded their 10-minute lock period.

**Rate Limiting**: Express-rate-limit middleware on payment endpoints (10 requests per 15 minutes) to prevent spam and abuse.

**Validation**: Zod schemas defined in shared/schema.ts for runtime validation of all API inputs, ensuring type safety and preventing invalid data from entering the system.

**Rationale**: In-memory storage enables rapid prototyping and easy Replit deployment without database provisioning. The IStorage interface abstraction means switching to PostgreSQL/Drizzle ORM later requires only implementing the interface, not refactoring business logic. JWT provides stateless authentication suitable for serverless/edge deployments.

### Database Design (Schema)

**ORM**: Drizzle ORM configured (drizzle.config.ts) for PostgreSQL migrations, though currently using in-memory storage.

**Core Entities**:
- **User**: id, name, email, phoneNumbers (array), role, password
- **Area**: id, name, city
- **ParkingLot**: id, name, areaId, address, description, ratePerHour, imageUrl
- **Slot**: id, lotId, identifier, type, ratePerHour (optional override), status
- **Reservation**: id, slotId, userId, startAt, endAt, totalAmount, paymentMethod, paymentStatus, qrCodeUrl, createdAt

**Relationships**:
- Area → Many ParkingLots
- ParkingLot → Many Slots
- Slot → Many Reservations (time-series data)
- User → Many Reservations

**Design Pattern**: Rich entity types with computed/joined data (AreaWithLots, ParkingLotWithSlots, SlotWithAvailability, ReservationWithDetails) provide denormalized views optimized for specific UI needs without complex client-side joins.

**Rationale**: The schema separates concerns (geographic areas, physical lots, individual slots) allowing flexible pricing (per-lot or per-slot rates). Array-based phoneNumbers supports multi-number SMS notifications without a separate junction table.

## External Dependencies

### Payment Integrations

**M-Pesa (Safaricom Daraja API)**:
- STK Push for mobile money collection
- Sandbox environment: `https://sandbox.safaricom.co.ke`
- Requires: MPESA_CONSUMER_KEY, MPESA_CONSUMER_SECRET, MPESA_SHORTCODE, MPESA_PASSKEY, MPESA_CALLBACK_URL
- Fallback: Mock implementation logs payment attempts when credentials not configured

**PayPal REST API**:
- Sandbox environment: `https://api-m.sandbox.paypal.com`
- Requires: PAYPAL_CLIENT_ID, PAYPAL_SECRET
- Fallback: Mock implementation returns simulated order IDs

**Bank Transfer**:
- Manual confirmation flow via admin dashboard
- Admin can mark pending bank transfers as "paid"

**Design Decision**: All payment integrations include graceful degradation (mock implementations) allowing development/testing without API credentials. The PaymentInitiation schema enforces validation before hitting external APIs.

### SMS Notifications

**Provider**: Twilio
- Requires: TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN, TWILIO_FROM_NUMBER
- Bulk SMS support: Iterates phoneNumbers array on User/Reservation
- Fallback: Console logging when Twilio credentials unavailable

**Message Format**: Reservation confirmations include lot name, slot identifier, time range, QR code URL.

**Rationale**: Twilio provides reliable global delivery with simple REST API. The sendBulkSMS abstraction allows future swap to alternative providers (Africa's Talking, etc.) by implementing the same interface.

### QR Code Generation

**Library**: qrcode npm package
- Generates PNG images stored in `/public/qrcodes/`
- QR data: JSON with reservationId, token (verification), timestamp
- Returns public URL path for embedding in SMS/email

**Security**: Reservation tokens generated with nanoid-style randomness to prevent guessing/enumeration attacks.

**Rationale**: File-based QR storage is simple for development. Production deployments should migrate to cloud storage (S3, Cloudinary) to avoid filesystem dependencies in containerized environments.

### Third-Party UI Libraries

**Radix UI**: Unstyled accessible component primitives (Dialog, Dropdown, Tabs, etc.)
**Tailwind CSS**: Utility-first styling with custom theme configuration
**Lucide React**: Icon library with tree-shaking support
**date-fns**: Date formatting and manipulation
**Axios**: HTTP client for external API calls (M-Pesa, PayPal)

### Development Tools

**Vite**: Build tool with HMR, optimized production builds
**TypeScript**: Type checking across client, server, and shared code
**Replit Plugins**: Runtime error overlay, cartographer, dev banner for Replit environment integration

### Environment Variables Required

```
# JWT Authentication
JWT_SECRET=your-secret-key

# Database (for Drizzle migrations)
DATABASE_URL=postgresql://...

# M-Pesa
MPESA_CONSUMER_KEY=your-key
MPESA_CONSUMER_SECRET=your-secret
MPESA_SHORTCODE=your-shortcode
MPESA_PASSKEY=your-passkey
MPESA_CALLBACK_URL=https://yourdomain.com/api/payments/mpesa/callback

# PayPal
PAYPAL_CLIENT_ID=your-client-id
PAYPAL_SECRET=your-secret

# Twilio
TWILIO_ACCOUNT_SID=your-sid
TWILIO_AUTH_TOKEN=your-token
TWILIO_FROM_NUMBER=+1234567890
```

**Note**: All external service integrations have mock fallbacks, so the app runs without credentials for local development.