# Recreating IDURAR in Next.js/React - Complete Migration Guide

**Step-by-Step Framework for CTO and Development Team**

---

## 📋 Table of Contents

1. [Introduction](#introduction)
2. [Why Next.js?](#why-nextjs)
3. [Architecture Comparison](#architecture-comparison)
4. [Migration Strategy](#migration-strategy)
5. [Step-by-Step Implementation](#step-by-step-implementation)
6. [Feature-by-Feature Migration](#feature-by-feature-migration)
7. [Testing & Deployment](#testing--deployment)

---

## Introduction

This guide provides a comprehensive framework for recreating the IDURAR ERP/CRM application using Next.js 14+ (App Router) while maintaining the same functionality.

### What We're Building

A modern ERP/CRM system with:
- **Frontend**: Next.js 14 (App Router) + React Server Components
- **Backend**: Next.js API Routes or separate Node.js/Express API
- **Database**: MongoDB with Prisma ORM (or Mongoose)
- **UI**: Tailwind CSS + shadcn/ui (or keep Ant Design)
- **State**: Zustand or React Query (simpler than Redux)
- **Auth**: NextAuth.js or Clerk

---

## Why Next.js?

### Advantages Over Current React + Vite Setup

| Feature | Current (React+Vite) | Next.js 14 |
|---------|---------------------|------------|
| **Rendering** | Client-side only (CSR) | SSR, SSG, ISR, RSC |
| **Routing** | React Router (client) | File-based (built-in) |
| **API** | Separate Express server | Built-in API routes |
| **SEO** | Limited | Excellent |
| **Performance** | Good | Better (automatic optimization) |
| **Code Splitting** | Manual | Automatic |
| **Image Optimization** | Manual | Built-in |
| **TypeScript** | Manual setup | First-class support |

### When to Use Which Approach

**Use Current Stack (React + Express) if:**
- Need complete API control
- Team familiar with current stack
- Microservices architecture

**Use Next.js if:**
- Want better SEO
- Need SSR for performance
- Want unified codebase
- Prefer modern React patterns

---

## Architecture Comparison

### Current Architecture
```
┌─────────────┐    ┌──────────────┐    ┌──────────┐
│  React SPA  │───▶│ Express API  │───▶│ MongoDB  │
│  (Port 3000)│    │  (Port 8888) │    │          │
└─────────────┘    └──────────────┘    └──────────┘
     (Vite)          (Separate Server)
```

### Next.js Architecture (Option 1: Monolith)
```
┌────────────────────────────────┐    ┌──────────┐
│        Next.js App             │───▶│ MongoDB  │
│  ┌─────────────────────────┐  │    │          │
│  │  Pages (RSC + Client)   │  │    └──────────┘
│  └─────────────────────────┘  │
│  ┌─────────────────────────┐  │
│  │  API Routes (/api/*)    │  │
│  └─────────────────────────┘  │
└────────────────────────────────┘
    (Single Server, Port 3000)
```

### Next.js Architecture (Option 2: Hybrid)
```
┌─────────────┐    ┌──────────────┐    ┌──────────┐
│  Next.js    │───▶│ Express API  │───▶│ MongoDB  │
│  Frontend   │    │ (Keep current)│    │          │
│  (Port 3000)│    │  (Port 8888) │    └──────────┘
└─────────────┘    └──────────────┘
   (New Frontend)   (Existing Backend)
```

**Recommended**: Option 2 for migration (less risk), then consolidate to Option 1.

---

## Migration Strategy

### Phase 1: Setup & Foundation (Week 1-2)
1. Initialize Next.js project
2. Setup TypeScript
3. Configure Tailwind CSS
4. Install UI library (shadcn/ui or Ant Design)
5. Setup authentication (NextAuth.js)
6. Configure API client (axios/fetch)

### Phase 2: Core Features (Week 3-6)
1. Authentication & User Management
2. Dashboard
3. Customer Management
4. Invoice Management
5. Quote Management
6. Payment Management

### Phase 3: Advanced Features (Week 7-8)
1. Settings Management
2. File Uploads
3. PDF Generation
4. Email System
5. Analytics

### Phase 4: Testing & Optimization (Week 9-10)
1. Unit tests
2. Integration tests
3. Performance optimization
4. Security audit
5. Documentation

---

## Step-by-Step Implementation

### Step 1: Initialize Next.js Project

```bash
# Create Next.js app with TypeScript and Tailwind
npx create-next-app@latest idurar-nextjs --typescript --tailwind --app

cd idurar-nextjs

# Install dependencies
npm install axios zustand react-query @tanstack/react-query
npm install next-auth bcryptjs jsonwebtoken
npm install @prisma/client
npm install -D prisma
npm install zod react-hook-form @hookform/resolvers

# UI Library (choose one)
# Option A: shadcn/ui (recommended)
npx shadcn-ui@latest init

# Option B: Keep Ant Design
npm install antd @ant-design/nextjs-registry
```

**Project Structure:**
```
idurar-nextjs/
├── app/                    # Next.js App Router
│   ├── (auth)/            # Auth routes group
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/       # Protected routes group
│   │   ├── layout.tsx     # Dashboard layout
│   │   ├── page.tsx       # Dashboard home
│   │   ├── customers/
│   │   ├── invoices/
│   │   ├── quotes/
│   │   └── payments/
│   ├── api/               # API routes
│   │   ├── auth/
│   │   ├── invoices/
│   │   ├── customers/
│   │   └── [...etc]/
│   ├── layout.tsx         # Root layout
│   └── page.tsx           # Home page
├── components/            # React components
│   ├── ui/               # shadcn/ui components
│   ├── forms/
│   └── layouts/
├── lib/                   # Utilities
│   ├── db.ts             # Database client
│   ├── auth.ts           # Auth utilities
│   └── utils.ts
├── hooks/                 # Custom hooks
├── stores/                # Zustand stores
├── types/                 # TypeScript types
└── prisma/                # Database schema
    └── schema.prisma
```

---

### Step 2: Database Setup with Prisma

**Install Prisma:**
```bash
npx prisma init
```

**Create Schema (`prisma/schema.prisma`):**
```prisma
// This is your Prisma schema file

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mongodb"
  url      = env("DATABASE_URL")
}

model User {
  id            String    @id @default(auto()) @map("_id") @db.ObjectId
  email         String    @unique
  name          String?
  surname       String?
  password      String
  role          String    @default("admin")
  photo         String?
  enabled       Boolean   @default(true)
  emailVerified Boolean   @default(false)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  clients       Client[]
  invoices      Invoice[]
  quotes        Quote[]
  payments      Payment[]
}

model Client {
  id        String    @id @default(auto()) @map("_id") @db.ObjectId
  name      String
  email     String?
  phone     String?
  country   String?
  address   String?
  enabled   Boolean   @default(true)
  removed   Boolean   @default(false)
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  
  createdBy String    @db.ObjectId
  user      User      @relation(fields: [createdBy], references: [id])
  
  invoices  Invoice[]
  quotes    Quote[]
  payments  Payment[]
}

model Invoice {
  id            String    @id @default(auto()) @map("_id") @db.ObjectId
  number        Int
  year          Int
  date          DateTime
  expiredDate   DateTime
  content       String?
  taxRate       Float     @default(0)
  subTotal      Float     @default(0)
  taxTotal      Float     @default(0)
  total         Float
  currency      String    @default("USD")
  discount      Float     @default(0)
  credit        Float     @default(0)
  paymentStatus String    @default("unpaid") // unpaid, paid, partially
  status        String    @default("draft")  // draft, pending, sent, paid, cancelled
  notes         String?
  pdf           String?
  removed       Boolean   @default(false)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  clientId      String    @db.ObjectId
  client        Client    @relation(fields: [clientId], references: [id])
  
  createdBy     String    @db.ObjectId
  user          User      @relation(fields: [createdBy], references: [id])
  
  items         InvoiceItem[]
  payments      Payment[]
  
  // Conversion tracking
  convertedFromQuoteId String? @db.ObjectId
  convertedFromQuote   Quote?  @relation(fields: [convertedFromQuoteId], references: [id])
}

model InvoiceItem {
  id          String   @id @default(auto()) @map("_id") @db.ObjectId
  itemName    String
  description String?
  quantity    Int
  price       Float
  total       Float
  
  invoiceId   String   @db.ObjectId
  invoice     Invoice  @relation(fields: [invoiceId], references: [id], onDelete: Cascade)
}

model Quote {
  id            String    @id @default(auto()) @map("_id") @db.ObjectId
  number        Int
  year          Int
  date          DateTime
  expiredDate   DateTime
  content       String?
  taxRate       Float     @default(0)
  subTotal      Float     @default(0)
  taxTotal      Float     @default(0)
  total         Float
  currency      String    @default("USD")
  discount      Float     @default(0)
  status        String    @default("draft")
  notes         String?
  pdf           String?
  removed       Boolean   @default(false)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  clientId      String    @db.ObjectId
  client        Client    @relation(fields: [clientId], references: [id])
  
  createdBy     String    @db.ObjectId
  user          User      @relation(fields: [createdBy], references: [id])
  
  items         QuoteItem[]
  
  // Track if converted to invoice
  convertedToInvoices Invoice[]
}

model QuoteItem {
  id          String   @id @default(auto()) @map("_id") @db.ObjectId
  itemName    String
  description String?
  quantity    Int
  price       Float
  total       Float
  
  quoteId     String   @db.ObjectId
  quote       Quote    @relation(fields: [quoteId], references: [id], onDelete: Cascade)
}

model Payment {
  id          String    @id @default(auto()) @map("_id") @db.ObjectId
  number      Int
  date        DateTime  @default(now())
  amount      Float
  currency    String    @default("USD")
  ref         String?
  description String?
  removed     Boolean   @default(false)
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  clientId    String    @db.ObjectId
  client      Client    @relation(fields: [clientId], references: [id])
  
  invoiceId   String    @db.ObjectId
  invoice     Invoice   @relation(fields: [invoiceId], references: [id])
  
  createdBy   String    @db.ObjectId
  user        User      @relation(fields: [createdBy], references: [id])
  
  paymentModeId String?   @db.ObjectId
  paymentMode   PaymentMode? @relation(fields: [paymentModeId], references: [id])
}

model PaymentMode {
  id        String    @id @default(auto()) @map("_id") @db.ObjectId
  name      String    @unique
  enabled   Boolean   @default(true)
  removed   Boolean   @default(false)
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  
  payments  Payment[]
}

model Tax {
  id        String    @id @default(auto()) @map("_id") @db.ObjectId
  taxName   String
  taxValue  Float
  isDefault Boolean   @default(false)
  enabled   Boolean   @default(true)
  removed   Boolean   @default(false)
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Setting {
  id              String   @id @default(auto()) @map("_id") @db.ObjectId
  settingCategory String
  settingKey      String   @unique
  settingValue    String
  valueType       String   @default("String") // String, Number, Boolean, Date
  isPublic        Boolean  @default(false)
  isSecure        Boolean  @default(false)
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
}
```

**Generate Prisma Client:**
```bash
npx prisma generate
npx prisma db push
```

**Create Database Client (`lib/db.ts`):**
```typescript
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma = globalForPrisma.prisma || new PrismaClient();

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

---

### Step 3: Authentication with NextAuth.js

**Install NextAuth:**
```bash
npm install next-auth
```

**Create Auth Config (`app/api/auth/[...nextauth]/route.ts`):**
```typescript
import NextAuth, { NextAuthOptions } from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';
import bcrypt from 'bcryptjs';
import { prisma } from '@/lib/db';

export const authOptions: NextAuthOptions = {
  providers: [
    CredentialsProvider({
      name: 'Credentials',
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" }
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) {
          throw new Error('Invalid credentials');
        }

        const user = await prisma.user.findUnique({
          where: { email: credentials.email }
        });

        if (!user || !user.password) {
          throw new Error('Invalid credentials');
        }

        const isValid = await bcrypt.compare(credentials.password, user.password);

        if (!isValid) {
          throw new Error('Invalid credentials');
        }

        if (!user.enabled) {
          throw new Error('Account disabled');
        }

        return {
          id: user.id,
          email: user.email,
          name: user.name,
          role: user.role,
        };
      }
    })
  ],
  session: {
    strategy: 'jwt',
  },
  pages: {
    signIn: '/login',
  },
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.id = user.id;
        token.role = user.role;
      }
      return token;
    },
    async session({ session, token }) {
      if (session.user) {
        session.user.id = token.id as string;
        session.user.role = token.role as string;
      }
      return session;
    },
  },
};

const handler = NextAuth(authOptions);
export { handler as GET, handler as POST };
```

**Auth Context Provider (`components/providers/AuthProvider.tsx`):**
```typescript
'use client';

import { SessionProvider } from 'next-auth/react';
import { ReactNode } from 'react';

export default function AuthProvider({ children }: { children: ReactNode }) {
  return <SessionProvider>{children}</SessionProvider>;
}
```

**Update Root Layout (`app/layout.tsx`):**
```typescript
import AuthProvider from '@/components/providers/AuthProvider';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <AuthProvider>
          {children}
        </AuthProvider>
      </body>
    </html>
  );
}
```

---

### Step 4: Create API Routes

**Example: Invoice API (`app/api/invoices/route.ts`):**
```typescript
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/app/api/auth/[...nextauth]/route';
import { prisma } from '@/lib/db';
import { z } from 'zod';

// GET /api/invoices - List invoices
export async function GET(request: NextRequest) {
  const session = await getServerSession(authOptions);
  
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const { searchParams } = new URL(request.url);
  const page = parseInt(searchParams.get('page') || '1');
  const limit = parseInt(searchParams.get('limit') || '10');
  const skip = (page - 1) * limit;

  try {
    const [invoices, total] = await Promise.all([
      prisma.invoice.findMany({
        where: {
          removed: false,
          createdBy: session.user.id,
        },
        include: {
          client: true,
          items: true,
        },
        skip,
        take: limit,
        orderBy: { createdAt: 'desc' },
      }),
      prisma.invoice.count({
        where: {
          removed: false,
          createdBy: session.user.id,
        },
      }),
    ]);

    return NextResponse.json({
      success: true,
      data: invoices,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
      },
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to fetch invoices' },
      { status: 500 }
    );
  }
}

// POST /api/invoices - Create invoice
const createInvoiceSchema = z.object({
  clientId: z.string(),
  date: z.string(),
  expiredDate: z.string(),
  currency: z.string().default('USD'),
  taxRate: z.number().default(0),
  discount: z.number().default(0),
  notes: z.string().optional(),
  items: z.array(z.object({
    itemName: z.string(),
    description: z.string().optional(),
    quantity: z.number().min(1),
    price: z.number().min(0),
  })),
});

export async function POST(request: NextRequest) {
  const session = await getServerSession(authOptions);
  
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  try {
    const body = await request.json();
    const validatedData = createInvoiceSchema.parse(body);

    // Calculate totals
    const items = validatedData.items.map(item => ({
      ...item,
      total: item.quantity * item.price,
    }));

    const subTotal = items.reduce((sum, item) => sum + item.total, 0);
    const taxTotal = (subTotal * validatedData.taxRate) / 100;
    const total = subTotal + taxTotal - validatedData.discount;

    // Generate invoice number
    const currentYear = new Date().getFullYear();
    const lastInvoice = await prisma.invoice.findFirst({
      where: { year: currentYear },
      orderBy: { number: 'desc' },
    });
    const number = (lastInvoice?.number || 0) + 1;

    // Create invoice
    const invoice = await prisma.invoice.create({
      data: {
        number,
        year: currentYear,
        date: new Date(validatedData.date),
        expiredDate: new Date(validatedData.expiredDate),
        clientId: validatedData.clientId,
        currency: validatedData.currency,
        taxRate: validatedData.taxRate,
        discount: validatedData.discount,
        subTotal,
        taxTotal,
        total,
        notes: validatedData.notes,
        createdBy: session.user.id,
        items: {
          create: items,
        },
      },
      include: {
        client: true,
        items: true,
      },
    });

    return NextResponse.json({
      success: true,
      data: invoice,
    });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', details: error.errors },
        { status: 400 }
      );
    }
    return NextResponse.json(
      { error: 'Failed to create invoice' },
      { status: 500 }
    );
  }
}
```

**Single Invoice API (`app/api/invoices/[id]/route.ts`):**
```typescript
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/app/api/auth/[...nextauth]/route';
import { prisma } from '@/lib/db';

// GET /api/invoices/[id]
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const session = await getServerSession(authOptions);
  
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  try {
    const invoice = await prisma.invoice.findFirst({
      where: {
        id: params.id,
        removed: false,
        createdBy: session.user.id,
      },
      include: {
        client: true,
        items: true,
        payments: true,
      },
    });

    if (!invoice) {
      return NextResponse.json({ error: 'Invoice not found' }, { status: 404 });
    }

    return NextResponse.json({
      success: true,
      data: invoice,
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to fetch invoice' },
      { status: 500 }
    );
  }
}

// PATCH /api/invoices/[id]
export async function PATCH(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const session = await getServerSession(authOptions);
  
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  try {
    const body = await request.json();
    // Similar validation and update logic
    // ...

    const invoice = await prisma.invoice.update({
      where: { id: params.id },
      data: body,
      include: {
        client: true,
        items: true,
      },
    });

    return NextResponse.json({
      success: true,
      data: invoice,
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to update invoice' },
      { status: 500 }
    );
  }
}

// DELETE /api/invoices/[id] (soft delete)
export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const session = await getServerSession(authOptions);
  
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  try {
    await prisma.invoice.update({
      where: { id: params.id },
      data: { removed: true },
    });

    return NextResponse.json({
      success: true,
      message: 'Invoice deleted',
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to delete invoice' },
      { status: 500 }
    );
  }
}
```

---

### Step 5: Create Frontend Pages

**Dashboard Layout (`app/(dashboard)/layout.tsx`):**
```typescript
import { getServerSession } from 'next-auth';
import { redirect } from 'next/navigation';
import { authOptions } from '@/app/api/auth/[...nextauth]/route';
import Sidebar from '@/components/layouts/Sidebar';
import Header from '@/components/layouts/Header';

export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const session = await getServerSession(authOptions);

  if (!session) {
    redirect('/login');
  }

  return (
    <div className="flex h-screen">
      <Sidebar />
      <div className="flex-1 flex flex-col overflow-hidden">
        <Header user={session.user} />
        <main className="flex-1 overflow-y-auto bg-gray-100 p-6">
          {children}
        </main>
      </div>
    </div>
  );
}
```

**Dashboard Page (`app/(dashboard)/page.tsx`):**
```typescript
import { getServerSession } from 'next-auth';
import { authOptions } from '@/app/api/auth/[...nextauth]/route';
import { prisma } from '@/lib/db';
import DashboardStats from '@/components/dashboard/DashboardStats';
import RecentInvoices from '@/components/dashboard/RecentInvoices';
import RevenueChart from '@/components/dashboard/RevenueChart';

export default async function DashboardPage() {
  const session = await getServerSession(authOptions);

  // Fetch dashboard data (Server Component)
  const [invoiceStats, recentInvoices] = await Promise.all([
    prisma.invoice.groupBy({
      by: ['paymentStatus'],
      where: {
        removed: false,
        createdBy: session!.user.id,
      },
      _count: true,
      _sum: {
        total: true,
      },
    }),
    prisma.invoice.findMany({
      where: {
        removed: false,
        createdBy: session!.user.id,
      },
      include: {
        client: true,
      },
      take: 5,
      orderBy: { createdAt: 'desc' },
    }),
  ]);

  const stats = {
    totalRevenue: invoiceStats.reduce((sum, stat) => sum + (stat._sum.total || 0), 0),
    totalInvoices: invoiceStats.reduce((sum, stat) => sum + stat._count, 0),
    paidInvoices: invoiceStats.find(s => s.paymentStatus === 'paid')?._count || 0,
    unpaidInvoices: invoiceStats.find(s => s.paymentStatus === 'unpaid')?._count || 0,
  };

  return (
    <div>
      <h1 className="text-3xl font-bold mb-6">Dashboard</h1>
      
      <DashboardStats stats={stats} />
      
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 mt-6">
        <RevenueChart userId={session!.user.id} />
        <RecentInvoices invoices={recentInvoices} />
      </div>
    </div>
  );
}
```

**Invoice List Page (`app/(dashboard)/invoices/page.tsx`):**
```typescript
'use client';

import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';
import Link from 'next/link';
import axios from 'axios';
import { Button } from '@/components/ui/button';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Badge } from '@/components/ui/badge';

export default function InvoicesPage() {
  const [page, setPage] = useState(1);

  const { data, isLoading } = useQuery({
    queryKey: ['invoices', page],
    queryFn: async () => {
      const response = await axios.get(`/api/invoices?page=${page}`);
      return response.data;
    },
  });

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">Invoices</h1>
        <Link href="/invoices/create">
          <Button>Create Invoice</Button>
        </Link>
      </div>

      <div className="bg-white rounded-lg shadow">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Invoice #</TableHead>
              <TableHead>Client</TableHead>
              <TableHead>Date</TableHead>
              <TableHead>Total</TableHead>
              <TableHead>Status</TableHead>
              <TableHead>Actions</TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {data?.data.map((invoice: any) => (
              <TableRow key={invoice.id}>
                <TableCell>
                  {invoice.number}/{invoice.year}
                </TableCell>
                <TableCell>{invoice.client.name}</TableCell>
                <TableCell>
                  {new Date(invoice.date).toLocaleDateString()}
                </TableCell>
                <TableCell>
                  {invoice.currency} {invoice.total.toFixed(2)}
                </TableCell>
                <TableCell>
                  <Badge variant={invoice.paymentStatus === 'paid' ? 'success' : 'warning'}>
                    {invoice.paymentStatus}
                  </Badge>
                </TableCell>
                <TableCell>
                  <Link href={`/invoices/${invoice.id}`}>
                    <Button variant="ghost" size="sm">View</Button>
                  </Link>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>

      {/* Pagination */}
      <div className="mt-4 flex justify-center gap-2">
        <Button
          onClick={() => setPage(p => Math.max(1, p - 1))}
          disabled={page === 1}
        >
          Previous
        </Button>
        <span className="py-2 px-4">
          Page {page} of {data?.pagination.totalPages}
        </span>
        <Button
          onClick={() => setPage(p => p + 1)}
          disabled={page >= data?.pagination.totalPages}
        >
          Next
        </Button>
      </div>
    </div>
  );
}
```

**Create Invoice Page (`app/(dashboard)/invoices/create/page.tsx`):**
```typescript
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { useQuery, useMutation } from '@tanstack/react-query';
import axios from 'axios';
import { useForm, useFieldArray } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';

const invoiceSchema = z.object({
  clientId: z.string().min(1, 'Client is required'),
  date: z.string(),
  expiredDate: z.string(),
  currency: z.string().default('USD'),
  taxRate: z.number().default(0),
  discount: z.number().default(0),
  notes: z.string().optional(),
  items: z.array(z.object({
    itemName: z.string().min(1, 'Item name required'),
    description: z.string().optional(),
    quantity: z.number().min(1),
    price: z.number().min(0),
  })).min(1, 'At least one item required'),
});

type InvoiceFormData = z.infer<typeof invoiceSchema>;

export default function CreateInvoicePage() {
  const router = useRouter();
  
  const { data: clients } = useQuery({
    queryKey: ['clients'],
    queryFn: async () => {
      const response = await axios.get('/api/clients?limit=100');
      return response.data.data;
    },
  });

  const form = useForm<InvoiceFormData>({
    resolver: zodResolver(invoiceSchema),
    defaultValues: {
      date: new Date().toISOString().split('T')[0],
      expiredDate: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000)
        .toISOString().split('T')[0],
      currency: 'USD',
      taxRate: 0,
      discount: 0,
      items: [{ itemName: '', description: '', quantity: 1, price: 0 }],
    },
  });

  const { fields, append, remove } = useFieldArray({
    control: form.control,
    name: 'items',
  });

  const createMutation = useMutation({
    mutationFn: async (data: InvoiceFormData) => {
      const response = await axios.post('/api/invoices', data);
      return response.data;
    },
    onSuccess: (data) => {
      router.push(`/invoices/${data.data.id}`);
    },
  });

  const onSubmit = (data: InvoiceFormData) => {
    createMutation.mutate(data);
  };

  // Calculate totals
  const items = form.watch('items');
  const taxRate = form.watch('taxRate');
  const discount = form.watch('discount');
  
  const subTotal = items.reduce((sum, item) => sum + (item.quantity * item.price), 0);
  const taxTotal = (subTotal * taxRate) / 100;
  const total = subTotal + taxTotal - discount;

  return (
    <div>
      <h1 className="text-3xl font-bold mb-6">Create Invoice</h1>

      <div className="bg-white rounded-lg shadow p-6">
        <Form {...form}>
          <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
            
            {/* Client Selection */}
            <FormField
              control={form.control}
              name="clientId"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Client</FormLabel>
                  <Select onValueChange={field.onChange} defaultValue={field.value}>
                    <FormControl>
                      <SelectTrigger>
                        <SelectValue placeholder="Select a client" />
                      </SelectTrigger>
                    </FormControl>
                    <SelectContent>
                      {clients?.map((client: any) => (
                        <SelectItem key={client.id} value={client.id}>
                          {client.name}
                        </SelectItem>
                      ))}
                    </SelectContent>
                  </Select>
                  <FormMessage />
                </FormItem>
              )}
            />

            {/* Date fields */}
            <div className="grid grid-cols-2 gap-4">
              <FormField
                control={form.control}
                name="date"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Invoice Date</FormLabel>
                    <FormControl>
                      <Input type="date" {...field} />
                    </FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />

              <FormField
                control={form.control}
                name="expiredDate"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Due Date</FormLabel>
                    <FormControl>
                      <Input type="date" {...field} />
                    </FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />
            </div>

            {/* Line Items */}
            <div>
              <div className="flex justify-between items-center mb-4">
                <h3 className="text-lg font-semibold">Invoice Items</h3>
                <Button
                  type="button"
                  variant="outline"
                  onClick={() => append({ itemName: '', description: '', quantity: 1, price: 0 })}
                >
                  Add Item
                </Button>
              </div>

              {fields.map((field, index) => (
                <div key={field.id} className="border rounded p-4 mb-4">
                  <div className="grid grid-cols-12 gap-4">
                    <div className="col-span-4">
                      <FormField
                        control={form.control}
                        name={`items.${index}.itemName`}
                        render={({ field }) => (
                          <FormItem>
                            <FormLabel>Item Name</FormLabel>
                            <FormControl>
                              <Input {...field} />
                            </FormControl>
                            <FormMessage />
                          </FormItem>
                        )}
                      />
                    </div>

                    <div className="col-span-3">
                      <FormField
                        control={form.control}
                        name={`items.${index}.description`}
                        render={({ field }) => (
                          <FormItem>
                            <FormLabel>Description</FormLabel>
                            <FormControl>
                              <Input {...field} />
                            </FormControl>
                            <FormMessage />
                          </FormItem>
                        )}
                      />
                    </div>

                    <div className="col-span-2">
                      <FormField
                        control={form.control}
                        name={`items.${index}.quantity`}
                        render={({ field }) => (
                          <FormItem>
                            <FormLabel>Quantity</FormLabel>
                            <FormControl>
                              <Input
                                type="number"
                                {...field}
                                onChange={e => field.onChange(parseFloat(e.target.value))}
                              />
                            </FormControl>
                            <FormMessage />
                          </FormItem>
                        )}
                      />
                    </div>

                    <div className="col-span-2">
                      <FormField
                        control={form.control}
                        name={`items.${index}.price`}
                        render={({ field }) => (
                          <FormItem>
                            <FormLabel>Price</FormLabel>
                            <FormControl>
                              <Input
                                type="number"
                                step="0.01"
                                {...field}
                                onChange={e => field.onChange(parseFloat(e.target.value))}
                              />
                            </FormControl>
                            <FormMessage />
                          </FormItem>
                        )}
                      />
                    </div>

                    <div className="col-span-1 flex items-end">
                      <Button
                        type="button"
                        variant="destructive"
                        onClick={() => remove(index)}
                        disabled={fields.length === 1}
                      >
                        Remove
                      </Button>
                    </div>
                  </div>
                </div>
              ))}
            </div>

            {/* Tax and Discount */}
            <div className="grid grid-cols-3 gap-4">
              <FormField
                control={form.control}
                name="taxRate"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Tax Rate (%)</FormLabel>
                    <FormControl>
                      <Input
                        type="number"
                        step="0.01"
                        {...field}
                        onChange={e => field.onChange(parseFloat(e.target.value))}
                      />
                    </FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />

              <FormField
                control={form.control}
                name="discount"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Discount</FormLabel>
                    <FormControl>
                      <Input
                        type="number"
                        step="0.01"
                        {...field}
                        onChange={e => field.onChange(parseFloat(e.target.value))}
                      />
                    </FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />

              <FormField
                control={form.control}
                name="currency"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Currency</FormLabel>
                    <Select onValueChange={field.onChange} defaultValue={field.value}>
                      <FormControl>
                        <SelectTrigger>
                          <SelectValue />
                        </SelectTrigger>
                      </FormControl>
                      <SelectContent>
                        <SelectItem value="USD">USD</SelectItem>
                        <SelectItem value="EUR">EUR</SelectItem>
                        <SelectItem value="GBP">GBP</SelectItem>
                      </SelectContent>
                    </Select>
                    <FormMessage />
                  </FormItem>
                )}
              />
            </div>

            {/* Totals Display */}
            <div className="bg-gray-50 p-4 rounded">
              <div className="flex justify-between mb-2">
                <span>Subtotal:</span>
                <span className="font-semibold">{subTotal.toFixed(2)}</span>
              </div>
              <div className="flex justify-between mb-2">
                <span>Tax ({taxRate}%):</span>
                <span className="font-semibold">{taxTotal.toFixed(2)}</span>
              </div>
              <div className="flex justify-between mb-2">
                <span>Discount:</span>
                <span className="font-semibold">-{discount.toFixed(2)}</span>
              </div>
              <div className="flex justify-between text-lg font-bold border-t pt-2">
                <span>Total:</span>
                <span>{total.toFixed(2)}</span>
              </div>
            </div>

            {/* Notes */}
            <FormField
              control={form.control}
              name="notes"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Notes</FormLabel>
                  <FormControl>
                    <textarea
                      className="w-full border rounded p-2"
                      rows={4}
                      {...field}
                    />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />

            {/* Submit Button */}
            <div className="flex gap-4">
              <Button
                type="submit"
                disabled={createMutation.isPending}
              >
                {createMutation.isPending ? 'Creating...' : 'Create Invoice'}
              </Button>
              <Button
                type="button"
                variant="outline"
                onClick={() => router.back()}
              >
                Cancel
              </Button>
            </div>
          </form>
        </Form>
      </div>
    </div>
  );
}
```

---

### Step 6: State Management with Zustand

For simpler state management than Redux, use Zustand:

```typescript
// stores/useInvoiceStore.ts
import { create } from 'zustand';
import axios from 'axios';

interface Invoice {
  id: string;
  number: number;
  // ... other fields
}

interface InvoiceStore {
  invoices: Invoice[];
  currentInvoice: Invoice | null;
  isLoading: boolean;
  error: string | null;
  
  fetchInvoices: (page?: number) => Promise<void>;
  fetchInvoice: (id: string) => Promise<void>;
  createInvoice: (data: any) => Promise<Invoice>;
  updateInvoice: (id: string, data: any) => Promise<Invoice>;
  deleteInvoice: (id: string) => Promise<void>;
}

export const useInvoiceStore = create<InvoiceStore>((set) => ({
  invoices: [],
  currentInvoice: null,
  isLoading: false,
  error: null,

  fetchInvoices: async (page = 1) => {
    set({ isLoading: true, error: null });
    try {
      const response = await axios.get(`/api/invoices?page=${page}`);
      set({ invoices: response.data.data, isLoading: false });
    } catch (error) {
      set({ error: 'Failed to fetch invoices', isLoading: false });
    }
  },

  fetchInvoice: async (id) => {
    set({ isLoading: true, error: null });
    try {
      const response = await axios.get(`/api/invoices/${id}`);
      set({ currentInvoice: response.data.data, isLoading: false });
    } catch (error) {
      set({ error: 'Failed to fetch invoice', isLoading: false });
    }
  },

  createInvoice: async (data) => {
    set({ isLoading: true, error: null });
    try {
      const response = await axios.post('/api/invoices', data);
      set({ isLoading: false });
      return response.data.data;
    } catch (error) {
      set({ error: 'Failed to create invoice', isLoading: false });
      throw error;
    }
  },

  updateInvoice: async (id, data) => {
    set({ isLoading: true, error: null });
    try {
      const response = await axios.patch(`/api/invoices/${id}`, data);
      set({ currentInvoice: response.data.data, isLoading: false });
      return response.data.data;
    } catch (error) {
      set({ error: 'Failed to update invoice', isLoading: false });
      throw error;
    }
  },

  deleteInvoice: async (id) => {
    set({ isLoading: true, error: null });
    try {
      await axios.delete(`/api/invoices/${id}`);
      set((state) => ({
        invoices: state.invoices.filter(inv => inv.id !== id),
        isLoading: false,
      }));
    } catch (error) {
      set({ error: 'Failed to delete invoice', isLoading: false });
      throw error;
    }
  },
}));
```

---

## Feature-by-Feature Migration

### Customer Management
1. **API Routes**: `/api/customers/*`
2. **Pages**: `/customers`, `/customers/[id]`
3. **Components**: `CustomerList`, `CustomerForm`, `CustomerDetail`

### Quote Management
1. **API Routes**: `/api/quotes/*`, `/api/quotes/[id]/convert`
2. **Pages**: `/quotes`, `/quotes/create`, `/quotes/[id]`
3. **Components**: `QuoteList`, `QuoteForm`, `ConvertToInvoice`

### Payment Management
1. **API Routes**: `/api/payments/*`
2. **Pages**: `/payments`, `/invoices/[id]/record-payment`
3. **Components**: `PaymentList`, `PaymentForm`, `RecordPaymentModal`

### Settings Management
1. **API Routes**: `/api/settings/*`
2. **Pages**: `/settings`
3. **Components**: `CompanySettings`, `InvoiceSettings`, `EmailSettings`

### PDF Generation
- Use `@react-pdf/renderer` or server-side `puppeteer`
- API route: `/api/invoices/[id]/pdf`
- Generate PDF on demand

### Email System
- Use Resend or SendGrid
- API routes: `/api/invoices/[id]/send`, `/api/quotes/[id]/send`
- Email templates with React Email

---

## Testing & Deployment

### Testing
```bash
# Unit tests
npm install -D vitest @testing-library/react @testing-library/jest-dom

# E2E tests
npm install -D @playwright/test
npx playwright install
```

### Environment Variables
```env
# .env.local
DATABASE_URL="mongodb://..."
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="generate-a-secret"
RESEND_API_KEY="your-key"
AWS_ACCESS_KEY_ID="your-key"
AWS_SECRET_ACCESS_KEY="your-secret"
```

### Deployment (Vercel)
```bash
npm run build
vercel --prod
```

---

## Summary

**Key Migration Steps:**
1. ✅ Setup Next.js 14 with App Router
2. ✅ Configure Prisma with MongoDB
3. ✅ Implement NextAuth.js authentication
4. ✅ Create API routes for all entities
5. ✅ Build frontend pages with Server Components
6. ✅ Use React Query or Zustand for state
7. ✅ Implement PDF generation
8. ✅ Setup email system
9. ✅ Test thoroughly
10. ✅ Deploy

**Advantages of Next.js Version:**
- Better performance (SSR)
- Improved SEO
- Simpler deployment
- Built-in optimization
- Modern React patterns
- Type safety with TypeScript

**This guide provides a complete framework for recreating IDURAR in Next.js. Follow the steps sequentially, and you'll have a modern, performant ERP/CRM system!**
