# EAZY PARK - Smart Parking Booking System

A full-featured parking booking system with real-time availability, multiple payment methods (Mpesa, PayPal, Bank Transfer), SMS notifications, and QR code generation.

## Features

### User Features
- **Browse Parking Areas & Lots**: Search and discover parking locations by area and city
- **Real-time Slot Availability**: View available parking slots with color-coded status indicators
- **Smart Booking System**: Select time slots with 10-minute reservation locks to prevent double-booking
- **Multiple Payment Options**:
  - M-Pesa STK Push integration
  - PayPal sandbox payments
  - Bank transfer with admin confirmation
- **QR Code Generation**: Receive instant QR codes for parking entry
- **SMS Notifications**: Multi-number SMS confirmations via Twilio
- **Mobile-First Design**: Responsive UI optimized for all devices

### Admin Features
- **Dashboard Analytics**: View reservations, pending payments, and revenue stats
- **Parking Management**: Add and manage areas, lots, and parking slots
- **Reservation Management**: View all reservations with search and filter
- **Payment Confirmation**: Manually confirm bank transfer payments
- **Real-time Updates**: Monitor booking activity and availability

### Technical Features
- **Concurrency Control**: Slot locking prevents double-booking during checkout
- **Automated Cleanup**: Cron job releases expired pending reservations every 5 minutes
- **Rate Limiting**: Protection against payment spam (10 requests per 15 minutes)
- **JWT Authentication**: Secure admin access with token-based auth
- **Input Validation**: Server-side validation using Zod schemas

## Tech Stack

**Frontend:**
- React with Vite
- TypeScript
- Tailwind CSS
- TanStack Query (React Query)
- Wouter (routing)
- Shadcn UI components

**Backend:**
- Node.js + Express
- TypeScript
- In-memory storage (easily switchable to database)
- JWT for authentication
- bcryptjs for password hashing

**Integrations:**
- Twilio (SMS notifications)
- M-Pesa Daraja API (mobile payments)
- PayPal REST API (online payments)
- QRCode library (entry verification)

## Getting Started on Replit

### 1. Environment Variables

The following secrets are already configured in Replit Secrets:

**Required:**
- `JWT_SECRET` - Secret key for JWT tokens (e.g., a random UUID)
- `SESSION_SECRET` - Session secret for Express

**SMS Notifications (Twilio):**
- `TWILIO_ACCOUNT_SID` - Your Twilio Account SID
- `TWILIO_AUTH_TOKEN` - Your Twilio Auth Token
- `TWILIO_FROM_NUMBER` - Your Twilio phone number (format: +1234567890)

**M-Pesa Payment (Daraja API - Sandbox):**
- `MPESA_CONSUMER_KEY` - Daraja consumer key
- `MPESA_CONSUMER_SECRET` - Daraja consumer secret
- `MPESA_SHORTCODE` - Business shortcode
- `MPESA_PASSKEY` - Lipa Na Mpesa passkey
- `MPESA_CALLBACK_URL` - Webhook URL (your-replit-url/api/payments/webhook/mpesa)

**PayPal (Sandbox):**
- `PAYPAL_CLIENT_ID` - PayPal sandbox client ID
- `PAYPAL_SECRET` - PayPal sandbox secret

### 2. Run the Application

The app is already configured to run on Replit. Just click the **Run** button!

The application will:
1. Seed the database with sample data
2. Start the cleanup cron job
3. Launch both backend and frontend on port 5000

### 3. Access the Application

- **Public Site**: Main Replit URL
- **Admin Login**: Navigate to `/admin/login`
  - Email: `admin@eazypark.test`
  - Password: `password123`

## Seeded Data

The application comes pre-seeded with:

**Areas:**
- Nairobi CBD
- Westlands

**Parking Lots:**
- Central Plaza Parking (CBD) - KES 80/hr
- Kimathi Street Parking (CBD) - KES 100/hr
- Westlands Galleria Parking - KES 120/hr

**Slots:**
- Each lot has 10 slots (A1-A10)
- First 2 slots are VIP (1.5x rate)
- Remaining 8 are regular slots

**Admin User:**
- Email: admin@eazypark.test
- Password: password123

## API Endpoints

### Public Endpoints

```
GET    /api/areas                          - List all areas
GET    /api/areas/:id                      - Get area details
GET    /api/areas/:areaId/lots             - List lots in area
GET    /api/lots/:id                       - Get lot details
GET    /api/lots/:lotId/slots              - List slots (with availability check)
POST   /api/reservations/quote             - Get price quote
POST   /api/reservations                   - Create reservation
GET    /api/reservations/:id               - Get reservation details
POST   /api/payments/initiate              - Initiate payment
POST   /api/payments/webhook/mpesa         - M-Pesa callback
POST   /api/payments/webhook/paypal        - PayPal callback
```

### Admin Endpoints (Require Authentication)

```
POST   /api/admin/login                    - Admin login
GET    /api/admin/reservations             - List all reservations
POST   /api/admin/areas                    - Create area
POST   /api/admin/lots                     - Create parking lot
POST   /api/admin/slots                    - Create slot
PUT    /api/admin/slots/:id                - Update slot
POST   /api/admin/payments/confirm-bank    - Confirm bank payment
```

## Payment Flow

### M-Pesa
1. User selects slot and enters details
2. Creates reservation (status: pending)
3. Initiates M-Pesa STK Push
4. User approves on phone
5. Webhook confirms payment
6. QR code generated, SMS sent
7. Reservation status: paid

### PayPal
1. User selects slot and enters details
2. Creates reservation (status: pending)
3. Redirects to PayPal approval
4. User completes payment
5. Webhook confirms payment
6. QR code generated, SMS sent
7. Reservation status: paid

### Bank Transfer
1. User selects slot and enters details
2. Creates reservation (status: pending)
3. Displays bank account details
4. User completes transfer offline
5. Admin confirms payment manually
6. QR code generated, SMS sent
7. Reservation status: paid

## Testing Payment Webhooks

### M-Pesa Webhook Simulation

```bash
curl -X POST https://your-replit-url/api/payments/webhook/mpesa \
  -H "Content-Type: application/json" \
  -d '{
    "Body": {
      "stkCallback": {
        "CheckoutRequestID": "ws_CO_123456789",
        "ResultCode": 0
      }
    }
  }'
```

### PayPal Webhook Simulation

```bash
curl -X POST https://your-replit-url/api/payments/webhook/paypal \
  -H "Content-Type: application/json" \
  -d '{
    "event_type": "PAYMENT.CAPTURE.COMPLETED",
    "resource": {
      "id": "PAYPAL_ORDER_ID"
    }
  }'
```

## Obtaining API Credentials

### Twilio (SMS)
1. Sign up at https://www.twilio.com
2. Get a phone number
3. Copy Account SID, Auth Token, and Phone Number
4. Add to Replit Secrets

### M-Pesa Daraja (Kenya)
1. Create account at https://developer.safaricom.co.ke
2. Create a sandbox app
3. Get Consumer Key, Consumer Secret, Shortcode, and Passkey
4. Set callback URL to your Replit URL + `/api/payments/webhook/mpesa`
5. Add credentials to Replit Secrets

### PayPal
1. Sign up at https://developer.paypal.com
2. Create a sandbox app
3. Get Client ID and Secret
4. Add to Replit Secrets

## Architecture

### Frontend Structure
```
client/src/
├── components/          # Reusable UI components
│   ├── admin/          # Admin-specific components
│   ├── Header.tsx
│   ├── StatusBadge.tsx
│   ├── LoadingSkeleton.tsx
│   └── ReservationModal.tsx
├── pages/              # Page components
│   ├── Home.tsx
│   ├── AreaPage.tsx
│   ├── LotPage.tsx
│   ├── PaymentPage.tsx
│   ├── ConfirmationPage.tsx
│   ├── AdminLogin.tsx
│   └── AdminDashboard.tsx
└── App.tsx            # Main app with routing
```

### Backend Structure
```
server/
├── services/
│   ├── payments/
│   │   ├── mpesa.ts    # M-Pesa integration
│   │   └── paypal.ts   # PayPal integration
│   ├── sms.ts          # Twilio SMS service
│   └── qr.ts           # QR code generation
├── auth.ts             # JWT authentication
├── routes.ts           # API routes
├── storage.ts          # In-memory storage
├── seed.ts             # Database seeding
├── cron.ts             # Cleanup cron job
└── index.ts            # Server entry point
```

### Data Models
- **User**: Admin and customer accounts
- **Area**: Geographic parking areas
- **ParkingLot**: Individual parking facilities
- **Slot**: Individual parking spaces
- **Reservation**: Booking records with payment info

## Concurrency & Availability Logic

**Slot Availability:**
- Available if no overlapping reservations with status `paid` or `pending` (within lock period)
- 10-minute lock when reservation is pending
- Expired locks are cleaned up by cron job

**Reservation Locking:**
1. User creates reservation (10-minute lock)
2. Payment must complete within 10 minutes
3. If not paid, status changes to `cancelled`
4. Slot becomes available again

## Future Enhancements

- [ ] Multi-day parking passes
- [ ] Recurring reservations
- [ ] Map integration (Leaflet/Google Maps)
- [ ] Real-time WebSocket updates
- [ ] Email notifications
- [ ] Multi-currency support
- [ ] User accounts and booking history
- [ ] PostgreSQL database migration
- [ ] Payment history and receipts
- [ ] Promotional codes and discounts

## License

MIT

## Support

For issues or questions, please contact the development team.
