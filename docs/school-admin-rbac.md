---
sidebar_position: 5
title: Permissions & Multi-Tenancy
---

# Permissions, Multi-Tenancy & School Administration

NeuroSense Africa is built from the ground up as a multi-tenant platform. All student data, progress telemetry logs, and caregiver assignments are isolated by school boundaries.

---

## User Roles & Permissions Matrix

The platform enforces strict role-based access controls (RBAC) at the routing, page, and API query levels.

| Feature / Page | Caregiver | Classroom Teacher | School Administrator |
| :--- | :---: | :---: | :---: |
| **Speech Practice (Basic)** | Yes | Yes | Yes |
| **Focus Skills (Basic)** | Yes | Yes | Yes |
| **Social Lessons (Basic)** | Yes | Yes | Yes |
| **Personalized Onboarding** | Yes | No | Yes |
| **Child Progress Dashboard** | Own Child | Assigned Children | Redirect to Admin |
| **Admin Console (/admin)** | No | No | Yes |
| **Staff & Teacher Management** | No | No | Yes |
| **Roster / Student Management** | No | No | Yes |
| **Stripe Billing Panel** | Personal | No | Institutional (School) |

---

## Multi-Tenant Data Isolation

To prevent cross-tenant data leakage (where users of one school could accidentally view data from another), queries are scoped at the database level using `schoolId`.

### Admin Dashboard Scoping Example (`/admin/page.tsx`)
```typescript
const schoolId = dbUser.schoolId;

// Total children in the admin's school only
const childrenCount = await prisma.child.count({
  where: { schoolId }
});

// Total sessions completed by children in this school only
const totalSessionsCount = await prisma.activitySession.count({
  where: { child: { schoolId } }
});
```

### API Endpoint Scoping Example (`/api/admin/children/route.ts`)
```typescript
const adminUser = await prisma.user.findUnique({ where: { id: session.user.id } });
if (!adminUser || adminUser.role !== "ADMIN" || !adminUser.schoolId) {
  return NextResponse.json({ error: "Forbidden" }, { status: 403 });
}

// Fetch children belonging exclusively to the admin's school
const children = await prisma.child.findMany({
  where: { schoolId: adminUser.schoolId },
  include: { goals: true }
});
```

---

## Teacher-Student Assignments

A school administrator assigns classroom teachers to specific students. These assignments are maintained in the `TeacherChildAssignment` join table.
- **Teacher View Limits**: On the teacher dashboard, the dropdown switcher is populated only with student profiles where a join assignment is active.
- **Logging Verification**: When `/api/sessions` receives a progress log from a teacher, the backend validates that a mapping exists for that specific `teacherId` and `childId` before saving.

---

## Print-to-PDF Layout Customizations

Both the Caregiver Dashboard and the School Admin Report feature a "Generate Report" print button. We use custom CSS inside `@media print` rules to optimize the layout for physical printing or PDF downloads.

```css
@media print {
  /* 1. Hide interactive elements */
  aside, nav, header, button, .no-print {
    display: none !important;
  }
  
  /* 2. Break layout restrictions */
  body, html, main, .flex.h-screen {
    height: auto !important;
    overflow: visible !important;
    background: white !important;
    color: black !important;
  }

  /* 3. Page break optimizations */
  .page-break {
    page-break-before: always;
  }
}
```
This guarantees that PDF generation (such as Chrome's *Save as PDF*) correctly splits long student rosters and charts across standard A4 pages without truncation or scrolling cutoff artifacts.
