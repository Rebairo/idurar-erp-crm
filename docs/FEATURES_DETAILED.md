# IDURAR ERP/CRM - Detailed Feature Documentation

**Feature-by-Feature Technical Breakdown for Development Team**

---

## Table of Contents

1. [Customer Management (CRM)](#customer-management-crm)
2. [Invoice Management](#invoice-management)
3. [Quote Management](#quote-management)
4. [Payment Management](#payment-management)
5. [Settings & Configuration](#settings--configuration)
6. [Authentication & Security](#authentication--security)
7. [PDF Generation System](#pdf-generation-system)
8. [Email System](#email-system)
9. [Dashboard & Analytics](#dashboard--analytics)
10. [File Management](#file-management)

---

## Customer Management (CRM)

### Overview
Complete customer relationship management system for tracking clients and their business relationships.

### Technical Stack
- **Backend**: `backend/src/controllers/appControllers/clientController/`
- **Frontend**: `frontend/src/pages/Customer/`
- **Model**: `backend/src/models/appModels/Client.js`
- **API Base**: `/api/client/`

### Database Schema
```javascript
{
  _id: ObjectId,
  name: String (required),
  email: String,
  phone: String,
  country: String,
  address: String,
  enabled: Boolean (default: true),
  removed: Boolean (default: false),  // Soft delete
  createdBy: ObjectId → Admin,
  assigned: ObjectId → Admin,         // Sales rep assignment
  created: Date,
  updated: Date
}
```

### API Endpoints

```
GET    /api/client/list              # Paginated list
GET    /api/client/listAll           # All clients (non-paginated)
POST   /api/client/create            # Create new client
GET    /api/client/read/:id          # Get single client
PATCH  /api/client/update/:id        # Update client
DELETE /api/client/delete/:id        # Soft delete
GET    /api/client/search?q=term     # Search clients
GET    /api/client/filter?field=val  # Filter clients
GET    /api/client/summary           # Statistics
```

### Business Logic

**Client Creation Flow:**
```
1. User fills form in frontend
2. Validation (Joi schema)
3. Check for duplicate email
4. Create client record
5. Link to current user (createdBy)
6. Return client data
7. Redirect to client list
```

**Client Summary Statistics:**
```javascript
{
  totalClients: 150,
  activeClients: 140,
  inactiveClients: 10,
  newThisMonth: 12,
  topClients: [
    {
      client: {...},
      totalInvoices: 45,
      totalRevenue: 125000,
      outstandingAmount: 15000
    }
  ]
}
```

### Frontend Implementation

**Component Structure:**
```
pages/Customer/
  └── index.jsx              # Main page using CrudModule

Uses CrudModule with config:
- entity: 'client'
- dataTableColumns: [name, email, phone, country, actions]
- searchConfig: { name, email, phone }
- createFields: [name, email, phone, country, address]
```

**Redux Integration:**
```javascript
// State structure
{
  crud: {
    client: {
      list: { result: [], isLoading, pagination },
      current: { result: {}, isLoading },
      create: { isSuccess, isLoading },
      update: { isSuccess, isLoading },
      delete: { isSuccess, isLoading }
    }
  }
}

// Actions
dispatch(crud.list({ entity: 'client', options: { page: 1 } }));
dispatch(crud.create({ entity: 'client', jsonData: {...} }));
dispatch(crud.update({ entity: 'client', id, jsonData: {...} }));
dispatch(crud.delete({ entity: 'client', id }));
```

### Key Features

1. **Client Assignment**: Assign clients to sales representatives
2. **Enable/Disable**: Toggle client active status
3. **Soft Delete**: Clients marked as removed, not deleted
4. **Search**: Full-text search across name, email, phone
5. **Filter**: Filter by country, status, assigned rep
6. **Relationship Tracking**: View all invoices, quotes, payments per client

---

## Invoice Management

### Overview
Complete invoicing system with line items, tax calculations, PDF generation, and payment tracking.

### Technical Stack
- **Backend**: `backend/src/controllers/appControllers/invoiceController/`
- **Frontend**: `frontend/src/modules/InvoiceModule/`, `frontend/src/pages/Invoice/`
- **Model**: `backend/src/models/appModels/Invoice.js`
- **API Base**: `/api/invoice/`
- **PDF**: `backend/src/pdf/`

### Database Schema
```javascript
{
  _id: ObjectId,
  number: Number (sequential per year),
  year: Number,
  date: Date (invoice date),
  expiredDate: Date (due date),
  content: String (description),
  recurring: Enum ['daily', 'weekly', 'monthly', 'annually', 'quarter'],
  
  client: ObjectId → Client (autopopulate),
  
  items: [
    {
      itemName: String (required),
      description: String,
      quantity: Number (required),
      price: Number (required),
      total: Number (quantity * price)
    }
  ],
  
  taxRate: Number (percentage),
  subTotal: Number,
  taxTotal: Number,
  total: Number,
  currency: String (default: 'USD'),
  discount: Number,
  credit: Number,
  
  payment: [ObjectId → Payment],
  paymentStatus: Enum ['unpaid', 'paid', 'partially'],
  isOverdue: Boolean,
  approved: Boolean,
  
  status: Enum ['draft', 'pending', 'sent', 'refunded', 'cancelled', 'on hold'],
  
  notes: String,
  pdf: String (file path),
  files: [{id, name, path, description, isPublic}],
  
  converted: {
    from: Enum ['quote', 'offer'],
    quote: ObjectId → Quote
  },
  
  createdBy: ObjectId → Admin,
  removed: Boolean (default: false),
  created: Date,
  updated: Date
}
```

### API Endpoints
```
POST   /api/invoice/create           # Create invoice
GET    /api/invoice/read/:id         # Get single invoice
PATCH  /api/invoice/update/:id       # Update invoice
DELETE /api/invoice/delete/:id       # Soft delete
GET    /api/invoice/list             # Paginated list
GET    /api/invoice/listAll          # All invoices
GET    /api/invoice/search           # Search
GET    /api/invoice/filter           # Filter
GET    /api/invoice/summary          # Statistics
POST   /api/invoice/mail             # Email invoice to client
```

### Business Logic

**Invoice Creation Flow:**
```
1. User fills invoice form with client and items
2. Client-side validation
3. POST /api/invoice/create
4. Server-side Joi validation
5. Generate invoice number:
   - Get current year
   - Find last invoice of current year
   - Increment number (or start at 1)
6. Calculate totals:
   - subTotal = Σ(items.quantity * items.price)
   - taxTotal = (subTotal * taxRate) / 100
   - total = subTotal + taxTotal - discount
7. Save invoice to database
8. Generate PDF (optional, can be deferred)
9. Return invoice data
10. Redirect to invoice detail page
```

**Invoice Number Generation:**
```javascript
// Example: INV-2024-001, INV-2024-002, etc.
const currentYear = new Date().getFullYear();
const lastInvoice = await Invoice.findOne({ year: currentYear })
  .sort({ number: -1 });
const number = (lastInvoice?.number || 0) + 1;

// Result: { number: 1, year: 2024 }
// Display as: INV-2024-001
```

**Payment Status Calculation:**
```javascript
// Automatic calculation based on payments
const totalPaid = invoice.payment.reduce((sum, payment) => sum + payment.amount, 0);

if (totalPaid === 0) {
  invoice.paymentStatus = 'unpaid';
} else if (totalPaid >= invoice.total) {
  invoice.paymentStatus = 'paid';
} else {
  invoice.paymentStatus = 'partially';
}

// Overdue detection
if (new Date() > invoice.expiredDate && invoice.paymentStatus !== 'paid') {
  invoice.isOverdue = true;
}
```

**Invoice Summary Statistics:**
```javascript
{
  total: 350,
  draft: 20,
  pending: 45,
  sent: 180,
  paid: 105,
  
  totalRevenue: 525000,
  paidRevenue: 350000,
  outstandingRevenue: 175000,
  overdueRevenue: 45000,
  
  averageInvoiceValue: 1500,
  
  monthlyRevenue: {
    '2024-01': 42000,
    '2024-02': 38000,
    '2024-03': 45000,
    // ...
  },
  
  topClients: [
    { client: {...}, totalInvoices: 25, totalRevenue: 75000 }
  ]
}
```

### Frontend Implementation

**Pages:**
```
frontend/src/pages/Invoice/
├── index.jsx             # Invoice list (data table)
├── InvoiceCreate.jsx     # Create new invoice
├── InvoiceRead.jsx       # Invoice detail view
├── InvoiceUpdate.jsx     # Edit invoice
└── InvoiceRecordPayment.jsx  # Record payment modal
```

**Module Components:**
```
frontend/src/modules/InvoiceModule/
├── InvoiceDataTable.jsx       # List view with filters
├── InvoiceForm.jsx            # Create/edit form
├── InvoiceItemsTable.jsx      # Line items table
├── RecordPayment/             # Payment recording
│   └── index.jsx
├── SendInvoiceEmail.jsx       # Email invoice modal
└── InvoicePDFViewer.jsx       # PDF preview
```

**Invoice Form Features:**
1. **Client Selection**: Dropdown with search
2. **Date Pickers**: Invoice date and due date
3. **Line Items**:
   - Dynamic array (add/remove rows)
   - Auto-calculation of item totals
   - Item name, description, quantity, price
4. **Tax Configuration**: Percentage input with live calculation
5. **Discount**: Fixed amount or percentage
6. **Currency Selection**: Multi-currency support
7. **Notes**: Rich text editor for additional information
8. **Status Selection**: Draft, pending, sent, etc.
9. **Save Actions**: Save as draft, save and send, save and view

**Real-time Calculations:**
```javascript
// React component with live totals
const InvoiceForm = () => {
  const [items, setItems] = useState([]);
  const [taxRate, setTaxRate] = useState(0);
  const [discount, setDiscount] = useState(0);
  
  const subTotal = items.reduce((sum, item) => 
    sum + (item.quantity * item.price), 0
  );
  
  const taxTotal = (subTotal * taxRate) / 100;
  const total = subTotal + taxTotal - discount;
  
  return (
    <div>
      {/* Form fields */}
      <div className="totals">
        <div>Subtotal: ${subTotal.toFixed(2)}</div>
        <div>Tax ({taxRate}%): ${taxTotal.toFixed(2)}</div>
        <div>Discount: -${discount.toFixed(2)}</div>
        <div>Total: ${total.toFixed(2)}</div>
      </div>
    </div>
  );
};
```

### PDF Generation

**Template**: `backend/src/pdf/invoice.pug`
**Library**: html-pdf

**PDF Generation Flow:**
```
1. User clicks "Download PDF" or invoice is created
2. GET /api/invoice/read/:id?generatePdf=true
3. Backend fetches invoice with populated client
4. Fetch company settings (logo, name, address)
5. Render Pug template with data:
   - Company information
   - Invoice details
   - Client information
   - Line items table
   - Totals
   - Notes
6. html-pdf converts HTML to PDF
7. Save PDF to storage (local or S3)
8. Update invoice.pdf = file path
9. Return PDF download URL
10. Browser initiates download
```

**PDF Template Structure:**
```pug
doctype html
html
  head
    style
      // CSS for professional invoice layout
  body
    .invoice-container
      .header
        .company-logo
          img(src=company.logo)
        .company-info
          h1= company.name
          p= company.address
      
      .invoice-info
        h2 Invoice ##{invoice.number}/#{invoice.year}
        p Date: #{invoice.date}
        p Due Date: #{invoice.expiredDate}
      
      .client-info
        h3 Bill To:
        p= client.name
        p= client.address
        p= client.email
      
      table.items
        thead
          tr
            th Item
            th Description
            th Quantity
            th Price
            th Total
        tbody
          each item in invoice.items
            tr
              td= item.itemName
              td= item.description
              td= item.quantity
              td= item.price
              td= item.total
      
      .totals
        p Subtotal: #{invoice.subTotal}
        p Tax (#{invoice.taxRate}%): #{invoice.taxTotal}
        p Discount: #{invoice.discount}
        p.total Total: #{invoice.total}
      
      .notes
        p= invoice.notes
      
      .footer
        p Thank you for your business!
```

### Email System

**Send Invoice via Email:**
```
1. User clicks "Send Invoice" button
2. Modal opens: Enter recipient email (pre-filled with client email)
3. Option to add custom message
4. POST /api/invoice/mail
   {
     id: invoiceId,
     email: recipient@email.com,
     message: "Custom message"
   }
5. Backend:
   - Fetch invoice data
   - Generate PDF if not exists
   - Render email template (Pug)
   - Send email via Resend API with PDF attachment
   - Log email sent in database
6. Return success/failure
7. Show notification to user
```

**Email Template:** `backend/src/emailTemplate/invoice.pug`

---

## Quote Management

### Overview
Sales quotation system that can be converted to invoices when accepted by clients.

### Technical Stack
- **Backend**: `backend/src/controllers/appControllers/quoteController/`
- **Frontend**: `frontend/src/modules/QuoteModule/`, `frontend/src/pages/Quote/`
- **Model**: `backend/src/models/appModels/Quote.js`
- **API Base**: `/api/quote/`

### Database Schema
Very similar to Invoice, but without payment tracking:
```javascript
{
  _id: ObjectId,
  number: Number (sequential),
  year: Number,
  date: Date,
  expiredDate: Date,
  content: String,
  
  client: ObjectId → Client,
  items: [...],  // Same as invoice items
  
  taxRate: Number,
  subTotal: Number,
  taxTotal: Number,
  total: Number,
  currency: String,
  discount: Number,
  
  status: Enum ['draft', 'pending', 'sent', 'accepted', 'rejected', 'expired'],
  notes: String,
  pdf: String,
  
  createdBy: ObjectId → Admin,
  removed: Boolean,
  created: Date,
  updated: Date
}
```

### API Endpoints
```
POST   /api/quote/create            # Create quote
GET    /api/quote/read/:id          # Get single quote
PATCH  /api/quote/update/:id        # Update quote
DELETE /api/quote/delete/:id        # Soft delete
GET    /api/quote/list              # Paginated list
GET    /api/quote/search            # Search
GET    /api/quote/filter            # Filter
GET    /api/quote/summary           # Statistics
POST   /api/quote/mail              # Email quote
GET    /api/quote/convert/:id       # Convert to invoice ⭐
```

### Quote to Invoice Conversion

**API Endpoint:** `GET /api/quote/convert/:id`

**Conversion Flow:**
```javascript
// Backend: quoteController/convertQuoteToInvoice.js

1. Fetch quote by ID (with populated client and items)
2. Verify quote exists and not already converted
3. Create new invoice with quote data:
   {
     client: quote.client,
     items: quote.items.map(item => ({
       itemName: item.itemName,
       description: item.description,
       quantity: item.quantity,
       price: item.price,
       total: item.total
     })),
     taxRate: quote.taxRate,
     subTotal: quote.subTotal,
     taxTotal: quote.taxTotal,
     total: quote.total,
     currency: quote.currency,
     discount: quote.discount,
     notes: quote.notes,
     
     // Generate new invoice number
     number: getNextInvoiceNumber(),
     year: currentYear,
     date: new Date(),
     expiredDate: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000),
     
     // Link back to quote
     converted: {
       from: 'quote',
       quote: quote._id
     },
     
     status: 'draft',
     paymentStatus: 'unpaid',
     createdBy: currentUser._id
   }
4. Update quote status to 'accepted'
5. Save both documents
6. Return new invoice data
7. Frontend redirects to new invoice detail page
```

**Frontend Implementation:**
```javascript
// Quote detail page
const handleConvertToInvoice = async (quoteId) => {
  try {
    const response = await axios.get(`/api/quote/convert/${quoteId}`);
    const newInvoice = response.data.result;
    
    // Show success notification
    notification.success({
      message: 'Quote Converted',
      description: `Quote #${quote.number} converted to Invoice #${newInvoice.number}`
    });
    
    // Navigate to new invoice
    navigate(`/invoice/read/${newInvoice._id}`);
  } catch (error) {
    notification.error({
      message: 'Conversion Failed',
      description: error.message
    });
  }
};
```

---

## Payment Management

### Overview
Track payments received from clients against invoices.

### Technical Stack
- **Backend**: `backend/src/controllers/appControllers/paymentController/`
- **Frontend**: `frontend/src/modules/PaymentModule/`, `frontend/src/pages/Payment/`
- **Model**: `backend/src/models/appModels/Payment.js`
- **API Base**: `/api/payment/`

### Database Schema
```javascript
{
  _id: ObjectId,
  number: Number (sequential),
  date: Date (default: now),
  amount: Number (required),
  currency: String (default: 'USD'),
  ref: String (transaction reference/check number),
  description: String,
  
  client: ObjectId → Client (autopopulate),
  invoice: ObjectId → Invoice (autopopulate),
  paymentMode: ObjectId → PaymentMode (autopopulate),
  
  createdBy: ObjectId → Admin,
  removed: Boolean,
  created: Date,
  updated: Date
}
```

### API Endpoints
```
POST   /api/payment/create          # Record payment
GET    /api/payment/read/:id        # Get single payment
PATCH  /api/payment/update/:id      # Update payment
DELETE /api/payment/delete/:id      # Delete payment
GET    /api/payment/list            # Paginated list
GET    /api/payment/search          # Search
GET    /api/payment/filter          # Filter
POST   /api/payment/mail            # Email payment receipt
```

### Payment Recording Flow

**From Invoice Detail Page:**
```
1. View invoice detail
2. Click "Record Payment" button
3. Modal/page opens with form:
   - Amount (pre-filled with remaining balance)
   - Payment date (default: today)
   - Payment method (dropdown)
   - Reference number (optional)
   - Description/notes (optional)
4. Submit payment
5. POST /api/payment/create
   {
     invoice: invoiceId,
     client: invoice.client,
     amount: 1500,
     date: '2024-03-15',
     paymentMode: paymentModeId,
     ref: 'CHK-12345',
     description: 'Payment via check'
   }
6. Backend:
   - Create payment record
   - Add payment to invoice.payment array
   - Recalculate invoice.paymentStatus:
     * totalPaid = sum of all payments
     * If totalPaid >= invoice.total → 'paid'
     * Else if totalPaid > 0 → 'partially'
     * Else → 'unpaid'
   - Update invoice.isOverdue = false if paid
   - Save changes
7. Return updated invoice
8. Redux updates state
9. Invoice detail page re-renders with updated status
10. Show success notification
```

**Payment Calculation Logic:**
```javascript
// Backend: paymentController/create.js
const payment = await Payment.create({
  number: await getNextPaymentNumber(),
  invoice: invoiceId,
  client: invoice.client,
  amount,
  date,
  paymentMode,
  ref,
  description,
  createdBy: currentUser._id
});

// Update invoice
invoice.payment.push(payment._id);

const totalPaid = await Payment.aggregate([
  { $match: { invoice: invoiceId, removed: false } },
  { $group: { _id: null, total: { $sum: '$amount' } } }
]);

const paidAmount = totalPaid[0]?.total || 0;

if (paidAmount >= invoice.total) {
  invoice.paymentStatus = 'paid';
  invoice.isOverdue = false;
} else if (paidAmount > 0) {
  invoice.paymentStatus = 'partially';
} else {
  invoice.paymentStatus = 'unpaid';
}

await invoice.save();
```

### Payment Modes

**Model:** `PaymentMode.js`
```javascript
{
  _id: ObjectId,
  name: String (unique, e.g., 'Credit Card', 'Bank Transfer'),
  enabled: Boolean (default: true),
  removed: Boolean,
  created: Date,
  updated: Date
}
```

**Pre-configured Payment Modes:**
- Cash
- Credit Card
- Debit Card
- Bank Transfer
- PayPal
- Stripe
- Check
- Wire Transfer
- Online Payment

**Usage:**
Used in dropdown when recording payments. Admins can add/edit payment modes in settings.

---

## Settings & Configuration

### Overview
Dynamic application settings stored in database for easy configuration without code changes.

### Technical Stack
- **Backend**: `backend/src/controllers/coreControllers/settingController/`
- **Frontend**: `frontend/src/modules/SettingModule/`, `frontend/src/pages/Settings/`
- **Model**: `backend/src/models/coreModels/Setting.js`
- **API Base**: `/api/setting/`

### Database Schema
```javascript
{
  _id: ObjectId,
  settingCategory: String (e.g., 'company', 'invoice', 'email'),
  settingKey: String (unique, e.g., 'company_name', 'invoice_prefix'),
  settingValue: Mixed (can be string, number, boolean, object, array),
  valueType: Enum ['String', 'Number', 'Boolean', 'Date', 'Object', 'Array'],
  isPublic: Boolean (can be accessed without auth),
  isSecure: Boolean (sensitive data, encrypt if needed),
  created: Date,
  updated: Date
}
```

### API Endpoints
```
POST   /api/setting/create                         # Create setting
GET    /api/setting/read/:id                       # Get by ID
PATCH  /api/setting/update/:id                     # Update by ID
GET    /api/setting/list                           # All settings
GET    /api/setting/search                         # Search
GET    /api/setting/readBySettingKey/:key          # Get by key ⭐
GET    /api/setting/listBySettingKey?category=x    # Get group ⭐
PATCH  /api/setting/updateBySettingKey/:key        # Update by key ⭐
PATCH  /api/setting/upload/:key                    # Upload file setting
PATCH  /api/setting/updateManySetting              # Bulk update ⭐
```

### Common Settings

**Company Settings:**
```javascript
{
  settingKey: 'company_name',
  settingValue: 'Acme Corporation',
  settingCategory: 'company',
  valueType: 'String',
  isPublic: true
}

{
  settingKey: 'company_email',
  settingValue: 'info@acme.com',
  settingCategory: 'company',
  valueType: 'String',
  isPublic: true
}

{
  settingKey: 'company_phone',
  settingValue: '+1-555-1234',
  settingCategory: 'company',
  valueType: 'String',
  isPublic: true
}

{
  settingKey: 'company_address',
  settingValue: '123 Business St, City, State, ZIP',
  settingCategory: 'company',
  valueType: 'String',
  isPublic: true
}

{
  settingKey: 'company_logo',
  settingValue: '/uploads/logo.png',
  settingCategory: 'company',
  valueType: 'String',
  isPublic: true
}
```

**Invoice Settings:**
```javascript
{
  settingKey: 'invoice_prefix',
  settingValue: 'INV-',
  settingCategory: 'invoice',
  valueType: 'String'
}

{
  settingKey: 'invoice_starting_number',
  settingValue: 1,
  settingCategory: 'invoice',
  valueType: 'Number'
}

{
  settingKey: 'default_tax_rate',
  settingValue: 10,
  settingCategory: 'invoice',
  valueType: 'Number'
}

{
  settingKey: 'invoice_due_days',
  settingValue: 30,
  settingCategory: 'invoice',
  valueType: 'Number'
}

{
  settingKey: 'invoice_terms',
  settingValue: 'Payment due within 30 days...',
  settingCategory: 'invoice',
  valueType: 'String'
}
```

**Currency Settings:**
```javascript
{
  settingKey: 'default_currency',
  settingValue: 'USD',
  settingCategory: 'currency',
  valueType: 'String'
}

{
  settingKey: 'supported_currencies',
  settingValue: ['USD', 'EUR', 'GBP', 'CAD'],
  settingCategory: 'currency',
  valueType: 'Array'
}
```

**Email Settings:**
```javascript
{
  settingKey: 'email_from_name',
  settingValue: 'Acme Corporation',
  settingCategory: 'email',
  valueType: 'String'
}

{
  settingKey: 'email_from_address',
  settingValue: 'noreply@acme.com',
  settingCategory: 'email',
  valueType: 'String'
}

{
  settingKey: 'smtp_host',
  settingValue: 'smtp.resend.com',
  settingCategory: 'email',
  valueType: 'String',
  isSecure: true
}

{
  settingKey: 'smtp_api_key',
  settingValue: 're_xxxxxx',
  settingCategory: 'email',
  valueType: 'String',
  isSecure: true
}
```

### Settings Management UI

**Frontend Implementation:**
```
frontend/src/pages/Settings/
└── Settings.jsx

frontend/src/modules/SettingModule/
├── CompanySettings.jsx      # Company info form
├── InvoiceSettings.jsx      # Invoice configuration
├── EmailSettings.jsx        # Email configuration
├── CurrencySettings.jsx     # Currency settings
└── AppearanceSettings.jsx   # Branding, logo
```

**Settings Page Structure:**
```javascript
const SettingsPage = () => {
  const [activeTab, setActiveTab] = useState('company');
  
  return (
    <div className="settings-page">
      <Tabs activeKey={activeTab} onChange={setActiveTab}>
        <TabPane tab="Company" key="company">
          <CompanySettings />
        </TabPane>
        <TabPane tab="Invoice" key="invoice">
          <InvoiceSettings />
        </TabPane>
        <TabPane tab="Email" key="email">
          <EmailSettings />
        </TabPane>
        <TabPane tab="Currency" key="currency">
          <CurrencySettings />
        </TabPane>
      </Tabs>
    </div>
  );
};
```

**Bulk Update Settings:**
```javascript
// Frontend
const handleSaveSettings = async (category) => {
  const settings = {
    company_name: formValues.name,
    company_email: formValues.email,
    company_phone: formValues.phone,
    company_address: formValues.address
  };
  
  await axios.patch('/api/setting/updateManySetting', {
    settings: Object.entries(settings).map(([key, value]) => ({
      settingKey: key,
      settingValue: value
    }))
  });
};

// Backend
for (const { settingKey, settingValue } of settings) {
  await Setting.findOneAndUpdate(
    { settingKey },
    { settingValue },
    { upsert: true, new: true }
  );
}
```

---

## Authentication & Security

### Overview
JWT-based authentication with secure password storage and session management.

### Technical Stack
- **Backend**: `backend/src/controllers/coreControllers/adminAuth/`
- **Frontend**: `frontend/src/modules/AuthModule/`
- **Models**: `Admin.js`, `AdminPassword.js`
- **API Base**: `/api/`

### Authentication Architecture

**Two-Model Approach:**
```
Admin (Profile)          AdminPassword (Credentials)
- email                  - user → Admin
- name                   - password (hashed)
- surname                - salt
- role                   - emailToken
- photo                  - resetToken
- enabled                - emailVerified
```

**Why Separate?**
1. **Security**: Password never queried with profile data
2. **Performance**: Profile queries don't load password data
3. **Audit**: Separate password change history
4. **Privacy**: Password operations isolated

### API Endpoints
```
POST   /api/login              # User login
POST   /api/register           # New user registration
POST   /api/logout             # Sign out
POST   /api/forgetpassword     # Request password reset
POST   /api/resetpassword      # Reset with token
GET    /api/admin/read/:id     # Get user profile
PATCH  /api/admin/profile/update           # Update profile
PATCH  /api/admin/profile/password         # Change password
POST   /api/verify-email       # Email verification
```

### Authentication Flow

**Registration:**
```
1. User fills registration form
2. POST /api/register
   {
     email: 'user@example.com',
     password: 'SecurePass123!',
     name: 'John',
     surname: 'Doe'
   }
3. Backend:
   - Validate input (Joi schema)
   - Check email not already registered
   - Hash password (bcrypt, 12 rounds)
   - Create Admin record
   - Create AdminPassword record
   - Generate email verification token
   - Send verification email
4. Return success (without sensitive data)
5. Redirect to login
```

**Login:**
```
1. User enters email and password
2. POST /api/login
   {
     email: 'user@example.com',
     password: 'SecurePass123!'
   }
3. Backend:
   - Find Admin by email
   - Find linked AdminPassword
   - bcrypt.compare(password, hashedPassword)
   - If match:
     * Check user.enabled === true
     * Generate JWT token
       payload: { id: user._id, email, role }
       secret: process.env.JWT_SECRET
       expiresIn: '7d'
     * Set HTTP-only cookie
     * Return: { token, user: {id, email, name, role} }
   - If no match:
     * Return 401 error
4. Frontend:
   - Store token in localStorage
   - Store user in Redux
   - Redirect to dashboard
```

**JWT Token Structure:**
```javascript
// Payload
{
  id: '507f1f77bcf86cd799439011',
  email: 'user@example.com',
  role: 'admin',
  iat: 1677721200,  // Issued at
  exp: 1678326000   // Expires
}

// Token
'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjUwN2YxZjc3YmNmODZjZDc5OTQzOTAxMSIsImVtYWlsIjoidXNlckBleGFtcGxlLmNvbSIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTY3NzcyMTIwMCwiZXhwIjoxNjc4MzI2MDAwfQ.xxxxxxxxxxxxxxxxxxxxx'
```

**Authorization Middleware:**
```javascript
// backend/src/middlewares/createAuthMiddleware.js
const isValidAuthToken = async (req, res, next) => {
  try {
    // 1. Extract token from cookie or header
    const token = req.cookies.token || 
                  req.headers.authorization?.replace('Bearer ', '');
    
    if (!token) {
      return res.status(401).json({ message: 'No token provided' });
    }
    
    // 2. Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // 3. Find user
    const user = await Admin.findById(decoded.id);
    
    if (!user) {
      return res.status(401).json({ message: 'User not found' });
    }
    
    // 4. Check if user is enabled
    if (!user.enabled) {
      return res.status(403).json({ message: 'Account disabled' });
    }
    
    // 5. Attach user to request
    req.admin = user;
    req.adminId = user._id;
    
    // 6. Continue
    next();
  } catch (error) {
    return res.status(401).json({ message: 'Invalid token' });
  }
};
```

**Password Security:**
```javascript
// Hash password on registration
const salt = await bcrypt.genSalt(12);
const hashedPassword = await bcrypt.hash(password, salt);

await AdminPassword.create({
  user: adminId,
  password: hashedPassword,
  salt: salt
});

// Verify password on login
const isValid = await bcrypt.compare(password, adminPassword.password);

// Strength requirements (frontend validation)
- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one number
- At least one special character
```

**Password Reset Flow:**
```
1. User clicks "Forgot Password"
2. POST /api/forgetpassword { email }
3. Backend:
   - Find user by email
   - Generate reset token (crypto.randomBytes)
   - Save token to AdminPassword.resetToken
   - Set token expiry (24 hours)
   - Send email with reset link
     Link: http://app.com/reset-password?token=xxxxx
4. User clicks link
5. Frontend: Reset password page with token
6. User enters new password
7. POST /api/resetpassword { token, newPassword }
8. Backend:
   - Find user by resetToken
   - Check token not expired
   - Hash new password
   - Update AdminPassword
   - Clear resetToken
9. Redirect to login
10. User logs in with new password
```

### Frontend Auth Implementation

**Login Component:**
```javascript
const LoginForm = () => {
  const dispatch = useDispatch();
  const navigate = useNavigate();
  
  const handleLogin = async (values) => {
    try {
      const response = await axios.post('/api/login', values);
      const { token, user } = response.data;
      
      // Store token
      localStorage.setItem('token', token);
      
      // Update Redux
      dispatch(authActions.login({ user, token }));
      
      // Redirect
      navigate('/');
    } catch (error) {
      message.error('Invalid credentials');
    }
  };
  
  return (
    <Form onFinish={handleLogin}>
      <Form.Item name="email" rules={[{ required: true, type: 'email' }]}>
        <Input placeholder="Email" />
      </Form.Item>
      <Form.Item name="password" rules={[{ required: true }]}>
        <Input.Password placeholder="Password" />
      </Form.Item>
      <Button type="primary" htmlType="submit">Login</Button>
    </Form>
  );
};
```

**Protected Route:**
```javascript
// router/ProtectedRoute.jsx
const ProtectedRoute = ({ children }) => {
  const { isLoggedIn } = useSelector(state => state.auth);
  
  if (!isLoggedIn) {
    return <Navigate to="/login" />;
  }
  
  return children;
};

// Usage
<Route path="/" element={<ProtectedRoute><Dashboard /></ProtectedRoute>} />
```

**Axios Interceptor:**
```javascript
// request/request.js
import axios from 'axios';

const request = axios.create({
  baseURL: '/api',
  withCredentials: true
});

// Add token to requests
request.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Handle auth errors
request.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Token expired or invalid
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default request;
```

---

## Dashboard & Analytics

### Overview
Real-time business metrics and visualizations for quick insights.

### Technical Stack
- **Backend**: Summary endpoints in each controller
- **Frontend**: `frontend/src/modules/DashboardModule/`, `frontend/src/pages/Dashboard.jsx`
- **API**: Various `/api/{entity}/summary` endpoints

### Dashboard Metrics

**Key Performance Indicators:**
1. **Total Revenue**: Sum of all paid invoices
2. **Outstanding Amount**: Sum of unpaid/partially paid invoices
3. **Overdue Amount**: Sum of overdue invoices
4. **Total Invoices**: Count of all invoices
5. **Paid Invoices**: Count of paid invoices
6. **Unpaid Invoices**: Count of unpaid invoices
7. **Total Clients**: Count of active clients
8. **New Clients This Month**: Count of clients created this month
9. **Total Quotes**: Count of all quotes
10. **Pending Quotes**: Count of pending/sent quotes

**API Endpoints:**
```
GET /api/invoice/summary
GET /api/client/summary
GET /api/quote/summary
GET /api/payment/summary
```

**Invoice Summary Response:**
```javascript
{
  success: true,
  result: {
    // Counts
    total: 350,
    byStatus: {
      draft: 20,
      pending: 45,
      sent: 180,
      paid: 105
    },
    byPaymentStatus: {
      unpaid: 150,
      partially: 95,
      paid: 105
    },
    
    // Revenue
    totalRevenue: 525000,
    paidRevenue: 350000,
    unpaidRevenue: 125000,
    overdueRevenue: 50000,
    
    // Averages
    averageInvoiceValue: 1500,
    averagePaymentTime: 25, // days
    
    // Trends
    monthlyRevenue: {
      '2024-01': 42000,
      '2024-02': 38000,
      '2024-03': 45000,
      '2024-04': 48000,
      '2024-05': 52000,
      '2024-06': 50000
    },
    
    // Top performers
    topClients: [
      { id, name, totalInvoices: 25, totalRevenue: 75000 },
      { id, name, totalInvoices: 20, totalRevenue: 65000 },
      { id, name, totalInvoices: 18, totalRevenue: 55000 }
    ],
    
    // Recent activity
    recentInvoices: [...last 5 invoices],
    recentPayments: [...last 5 payments]
  }
}
```

### Frontend Implementation

**Dashboard Layout:**
```
Dashboard
├── Stats Cards Row
│   ├── Total Revenue Card
│   ├── Outstanding Card
│   ├── Total Invoices Card
│   └── Paid Invoices Card
├── Charts Row
│   ├── Monthly Revenue Chart (Line/Bar)
│   └── Invoice Status Pie Chart
└── Tables Row
    ├── Recent Invoices Table
    └── Recent Payments Table
```

**Component Structure:**
```javascript
const Dashboard = () => {
  const { data: stats, isLoading } = useQuery({
    queryKey: ['dashboard-stats'],
    queryFn: async () => {
      const [invoices, clients, payments] = await Promise.all([
        axios.get('/api/invoice/summary'),
        axios.get('/api/client/summary'),
        axios.get('/api/payment/summary')
      ]);
      return { invoices: invoices.data, clients: clients.data, payments: payments.data };
    }
  });
  
  return (
    <div className="dashboard">
      <h1>Dashboard</h1>
      
      {/* Stats Cards */}
      <Row gutter={16}>
        <Col span={6}>
          <Card>
            <Statistic
              title="Total Revenue"
              value={stats?.invoices.totalRevenue}
              prefix="$"
              valueStyle={{ color: '#3f8600' }}
            />
          </Card>
        </Col>
        {/* More stat cards... */}
      </Row>
      
      {/* Charts */}
      <Row gutter={16} style={{ marginTop: 24 }}>
        <Col span={16}>
          <Card title="Monthly Revenue">
            <RevenueChart data={stats?.invoices.monthlyRevenue} />
          </Card>
        </Col>
        <Col span={8}>
          <Card title="Invoice Status">
            <PieChart data={stats?.invoices.byStatus} />
          </Card>
        </Col>
      </Row>
      
      {/* Recent Activity */}
      <Row gutter={16} style={{ marginTop: 24 }}>
        <Col span={12}>
          <Card title="Recent Invoices">
            <RecentInvoicesTable data={stats?.invoices.recentInvoices} />
          </Card>
        </Col>
        <Col span={12}>
          <Card title="Recent Payments">
            <RecentPaymentsTable data={stats?.payments.recent} />
          </Card>
        </Col>
      </Row>
    </div>
  );
};
```

**Chart Implementation (using Recharts):**
```javascript
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Legend } from 'recharts';

const RevenueChart = ({ data }) => {
  const chartData = Object.entries(data).map(([month, revenue]) => ({
    month,
    revenue
  }));
  
  return (
    <LineChart width={600} height={300} data={chartData}>
      <CartesianGrid strokeDasharray="3 3" />
      <XAxis dataKey="month" />
      <YAxis />
      <Tooltip />
      <Legend />
      <Line type="monotone" dataKey="revenue" stroke="#8884d8" />
    </LineChart>
  );
};
```

---

## Summary

This documentation provides a complete technical breakdown of every major feature in the IDURAR ERP/CRM application. Each section includes:

✅ Database schema and relationships
✅ API endpoints and request/response formats
✅ Business logic and workflows
✅ Frontend implementation details
✅ Code examples and patterns
✅ Security considerations

**Next Steps:**
1. Review the [CODEBASE_OVERVIEW.md](./CODEBASE_OVERVIEW.md) for high-level architecture
2. Follow the [NEXTJS_MIGRATION.md](./NEXTJS_MIGRATION.md) to recreate in Next.js
3. Reference this document when implementing specific features

**Remember**: This application follows industry-standard patterns and best practices. Understanding these patterns will help you build better applications and work effectively with similar codebases.
