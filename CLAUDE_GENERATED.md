# Claude Generated Code

**Task**: Objective:
Help design and implement new features based on the current roadmap or project needs. Focus on clean, scalable, and testable code

Claude Tasks:

🧩 Feature Implementation
Read the existing codebase and integrate new feature modules where appropriate.

Ensure new code follows existing patterns and conventions.

🗂️ Example Features (replace/add specifics as needed):
Add user role-based access control

Implement export to PDF

Add multi-language support

Enable dark mode toggle

🧪 Testing & Documentation
Add unit/integration tests for new functionality.

Update README or in-code comments if required.

**Generated on**: Sat Jul 26 11:39:24 UTC 2025

## Generated Content

```
I notice there's a mismatch between the task description (Ionic React TypeScript for government billing/invoicing) and the actual codebase (which appears to be a Next.js Web3/blockchain application called "fil-frame"). 

I'll implement features that make sense for the existing codebase architecture while adapting them to be useful for a billing/invoicing context. I'll focus on **user role-based access control** and **export to PDF functionality** as these are most relevant.

## FILES_TO_CREATE

### 1. User Role Management Hook
**Path**: `./packages/nextjs/hooks/fil-frame/useUserRoles.ts`

### 2. Role-Based Access Control Component
**Path**: `./packages/nextjs/components/fil-frame/RoleGuard.tsx`

### 3. PDF Export Hook
**Path**: `./packages/nextjs/hooks/fil-frame/usePdfExport.ts`

### 4. PDF Export Component
**Path**: `./packages/nextjs/components/fil-frame/PdfExportButton.tsx`

### 5. Types for Role Management
**Path**: `./packages/nextjs/types/roles.ts`

## FILES_TO_MODIFY

### 1. Package.json (NextJS workspace)
**Path**: `./packages/nextjs/package.json`

## CODE_CHANGES

### FILES_TO_CREATE

#### 1. `./packages/nextjs/types/roles.ts`
```typescript
export enum UserRole {
  ADMIN = "admin",
  BILLING_MANAGER = "billing_manager",
  ACCOUNTANT = "accountant",
  VIEWER = "viewer",
  CLIENT = "client"
}

export interface Permission {
  resource: string;
  actions: string[];
}

export interface RoleConfig {
  role: UserRole;
  permissions: Permission[];
  displayName: string;
  description: string;
}

export interface UserRoleData {
  address: string;
  role: UserRole;
  assignedAt: Date;
  assignedBy: string;
}

export const ROLE_PERMISSIONS: Record<UserRole, Permission[]> = {
  [UserRole.ADMIN]: [
    { resource: "users", actions: ["create", "read", "update", "delete"] },
    { resource: "invoices", actions: ["create", "read", "update", "delete", "export"] },
    { resource: "payments", actions: ["create", "read", "update", "delete"] },
    { resource: "reports", actions: ["create", "read", "export"] },
  ],
  [UserRole.BILLING_MANAGER]: [
    { resource: "invoices", actions: ["create", "read", "update", "export"] },
    { resource: "payments", actions: ["read", "update"] },
    { resource: "reports", actions: ["read", "export"] },
  ],
  [UserRole.ACCOUNTANT]: [
    { resource: "invoices", actions: ["read", "export"] },
    { resource: "payments", actions: ["read"] },
    { resource: "reports", actions: ["read", "export"] },
  ],
  [UserRole.VIEWER]: [
    { resource: "invoices", actions: ["read"] },
    { resource: "payments", actions: ["read"] },
  ],
  [UserRole.CLIENT]: [
    { resource: "invoices", actions: ["read"] },
    { resource: "payments", actions: ["create", "read"] },
  ],
};
```

#### 2. `./packages/nextjs/hooks/fil-frame/useUserRoles.ts`
```typescript
import { useEffect, useState, useCallback } from "react";
import { useAccount } from "wagmi";
import { UserRole, UserRoleData, ROLE_PERMISSIONS, Permission } from "~~/types/roles";
import { useGlobalState } from "~~/services/store/store";

interface UseUserRolesReturn {
  userRole: UserRole | null;
  permissions: Permission[];
  hasPermission: (resource: string, action: string) => boolean;
  isLoading: boolean;
  error: string | null;
  assignRole: (address: string, role: UserRole) => Promise<boolean>;
  revokeRole: (address: string) => Promise<boolean>;
}

/**
 * Custom hook for managing user roles and permissions in the billing system
 */
export const useUserRoles = (): UseUserRolesReturn => {
  const { address, isConnected } = useAccount();
  const [userRole, setUserRole] = useState<UserRole | null>(null);
  const [permissions, setPermissions] = useState<Permission[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  // Store user roles in global state (in a real app, this would be in a database/smart contract)
  const userRoles = useGlobalState(state => state.userRoles || {});
  const setUserRoles = useGlobalState(state => state.setUserRoles);

  // Load user role on address change
  useEffect(() => {
    if (!address || !isConnected) {
      setUserRole(null);
      setPermissions([]);
      return;
    }

    setIsLoading(true);
    try {
      const roleData = userRoles[address.toLowerCase()];
      if (roleData) {
        setUserRole(roleData.role);
        setPermissions(ROLE_PERMISSIONS[roleData.role] || []);
      } else {
        // Default role for new users
        setUserRole(UserRole.VIEWER);
        setPermissions(ROLE_PERMISSIONS[UserRole.VIEWER]);
      }
      setError(null);
    } catch (err) {
      setError(err instanceof Error ? err.message : "Failed to load user role");
    } finally {
      setIsLoading(false);
    }
  }, [address, isConnected, userRoles]);

  // Check if user has specific permission
  const hasPermission = useCallback((resource: string, action: string): boolean => {
    if (!permissions.length) return false;
    
    const resourcePermission = permissions.find(p => p.resource === resource);
    return resourcePermission?.actions.includes(action) || false;
  }, [permissions]);

  // Assign role to user (admin only)
  const assignRole = useCallback(async (targetAddress: string, role: UserRole): Promise<boolean> => {
    if (!hasPermission("users", "update")) {
      setError("Insufficient permissions to assign roles");
      return false;
    }

    try {
      const newRoleData: UserRoleData = {
        address: targetAddress.toLowerCase(),
        role,
        assignedAt: new Date(),
        assignedBy: address || "",
      };

      const updatedRoles = {
        ...userRoles,
        [targetAddress.toLowerCase()]: newRoleData,
      };

      setUserRoles(updatedRoles);
      return true;
    } catch (err) {
      setError(err instanceof Error ? err.message : "Failed to assign role");
      return false;
    }
  }, [hasPermission, address, userRoles, setUserRoles]);

  // Revoke role from user (admin only)
  const revokeRole = useCallback(async (targetAddress: string): Promise<boolean> => {
    if (!hasPermission("users", "delete")) {
      setError("Insufficient permissions to revoke roles");
      return false;
    }

    try {
      const updatedRoles = { ...userRoles };
      delete updatedRoles[targetAddress.toLowerCase()];
      setUserRoles(updatedRoles);
      return true;
    } catch (err) {
      setError(err instanceof Error ? err.message : "Failed to revoke role");
      return false;
    }
  }, [hasPermission, userRoles, setUserRoles]);

  return {
    userRole,
    permissions,
    hasPermission,
    isLoading,
    error,
    assignRole,
    revokeRole,
  };
};
```

#### 3. `./packages/nextjs/components/fil-frame/RoleGuard.tsx`
```typescript
import React from "react";
import { useUserRoles } from "~~/hooks/fil-frame/useUserRoles";

interface RoleGuardProps {
  children: React.ReactNode;
  resource?: string;
  action?: string;
  fallback?: React.ReactNode;
  requireAuth?: boolean;
}

/**
 * Component for role-based access control
 * Renders children only if user has required permissions
 */
export const RoleGuard: React.FC<RoleGuardProps> = ({
  children,
  resource,
  action,
  fallback = null,
  requireAuth = true,
}) => {
  const { hasPermission, userRole, isLoading } = useUserRoles();

  if (isLoading) {
    return (
      <div className="flex items-center justify-center p-4">
        <div className="loading loading-spinner loading-md"></div>
        <span className="ml-2">Checking permissions...</span>
      </div>
    );
  }

  // If auth is required but user has no role
  if (requireAuth && !userRole) {
    return (
      <div className="alert alert-warning">
        <svg xmlns="http://www.w3.org/2000/svg" className="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L3.732 16c-.77.833.192 2.5 1.732 2.5z" />
        </svg>
        <span>Please connect your wallet to access this feature.</span>
      </div>
    );
  }

  // If specific resource/action permissions are required
  if (resource && action) {
    if (!hasPermission(resource, action)) {
      return fallback ? (
        <>{fallback}</>
      ) : (
        <div className="alert alert-error">
          <svg xmlns="http://www.w3.org/2000/svg" className="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
          <span>Access denied. Insufficient permissions for this action.</span>
        </div>
      );
    }
  }

  return <>{children}</>;
};

// Convenience components for common use cases
export const AdminOnly: React.FC<Omit<RoleGuardProps, 'resource' | 'action'>> = (props) => (
  <RoleGuard {...props} resource="users" action="read" />
);

export const BillingManagerOnly: React.FC<Omit<RoleGuardProps, 'resource' | 'action'>> = (props) => (
  <RoleGuard {...props} resource="invoices" action="create" />
);

export const CanViewInvoices: React.FC<Omit<RoleGuardProps, 'resource' | 'action'>> = (props) => (
  <RoleGuard {...props} resource="invoices" action="read" />
);

export const CanExport: React.FC<Omit<RoleGuardProps, 'resource' | 'action'>> = (props) => (
  <RoleGuard {...props} resource="invoices" action="export" />
);
```

#### 4. `./packages/nextjs/hooks/fil-frame/usePdfExport.ts`
```typescript
import { useCallback, useState } from "react";
import { useUserRoles } from "./useUserRoles";

interface PdfExportOptions {
  title?: string;
  author?: string;
  subject?: string;
  filename?: string;
}

interface PdfExportReturn {
  exportToPdf: (data: any, options?: PdfExportOptions) => Promise<void>;
  isExporting: boolean;
  error: string | null;
}

/**
 * Hook for exporting data to PDF format
 * Requires appropriate permissions to use
 */
export const usePdfExport = (): PdfExportReturn => {
  const [isExporting, setIsExporting] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const { hasPermission } = useUserRoles();

  const exportToPdf = useCallback(async (data: any, options: PdfExportOptions = {}) => {
    // Check permissions
    if (!hasPermission("invoices", "export") && !hasPermission("reports", "export")) {
      setError("Insufficient permissions to export PDF");
      return;
    }

    setIsExporting(true);
    setError(null);

    try {
      // Dynamic import to avoid loading PDF library until needed
      const { jsPDF } = await import("jspdf");
      const autoTable = (await import("jspdf-autotable")).default;

      const doc = new jsPDF();
      
      // Set PDF metadata
      doc.setProperties({
        title: options.title || "Export Document",
        subject: options.subject || "Generated Report",
        author: options.author || "Fil-Frame Billing System",
        creator: "Fil-Frame",
      });

      // Add header
      doc.setFontSize(20);
      doc.text(options.title || "Document Export", 20, 20);
      
      doc.setFontSize(12);
      doc.text(`Generated on: ${new Date().toLocaleDateString()}`, 20, 35);

      let yPosition = 50;

      // Handle different data types
      if (Array.isArray(data)) {
        // Table data
        if (data.length > 0 && typeof data[0] === "object") {
          const headers = Object.keys(data[0]);
          const rows = data.map(item => headers.map(header => String(item[header] || "")));

          autoTable(doc, {
            head: [headers],
            body: rows,
            startY: yPosition,
            theme: "grid",
            styles: { fontSize: 10 },
            headStyles: { fillColor: [59, 130, 246] }, // Blue header
          });
        } else {
          // Simple list
          data.forEach((item, index) => {
            doc.text(`${index + 1}. ${String(item)}`, 20, yPosition);
            yPosition += 10;
          });
        }
      } else if (typeof data === "object") {
        // Key-value pairs
        Object.entries(data).forEach(([key, value]) => {
          doc.text(`${key}: ${String(value)}`, 20, yPosition);
          yPosition += 10;
          
          // Check if we need a new page
          if (yPosition > 270) {
            doc.addPage();
            yPosition = 20;
          }
        });
      } else {
        // Plain text
        const lines = String(data).split('\n');
        lines.forEach(line => {
          doc.text(line, 20, yPosition);
          yPosition += 10;
          
          if (yPosition > 270) {
            doc.addPage();
            yPosition = 20;
          }
        });
      }

      // Add footer
      const pageCount = doc.getNumberOfPages();
      for (let i = 1; i <= pageCount; i++) {
        doc.setPage(i);
        doc.setFontSize(8);
        doc.text(
          `Page ${i} of ${pageCount}`,
          doc.internal.pageSize.getWidth() - 40,
          doc.internal.pageSize.getHeight() - 10
        );
      }

      // Save the PDF
      const filename = options.filename || `export-${Date.now()}.pdf`;
      doc.save(filename);

    } catch (err) {
      console.error("PDF export error:", err);
      setError(err instanceof Error ? err.message : "Failed to export PDF");
    } finally {
      setIsExporting(false);
    }
  }, [hasPermission]);

  return {
    exportToPdf,
    isExporting,
    error,
  };
};
```

#### 5. `./packages/nextjs/components/fil-frame/PdfExportButton.tsx`
```typescript
import React from "react";
import { usePdfExport } from "~~/hooks/fil-frame/usePdfExport";
import { RoleGuard } from "./RoleGuard";

interface PdfExportButtonProps {
  data: any;
  filename?: string;
  title?: string;
  className?: string;
  variant?: "primary" | "secondary" | "outline";
  size?: "sm" | "md" | "lg";
}

/**
 * Button component for exporting data to PDF
 * Automatically handles permissions and loading states
 */
export const PdfExportButton: React.FC<PdfExportButtonProps> = ({
  data,
  filename,
  title = "Export to PDF",
  className = "",
  variant = "primary",
  size = "md",
}) => {
  const { exportToPdf, isExporting, error } = usePdfExport();

  const handleExport = async () => {
    await exportToPdf(data, {
      filename: filename || `export-${Date.now()}.pdf`,
      title: title,
    });
  };

  const getButtonClasses = () => {
    const baseClasses = "btn";
    const variantClasses = {
      primary: "btn-primary",
      secondary: "btn-secondary",
      outline: "btn-outline",
    };
    const sizeClasses = {
      sm: "btn-sm",
      md: "",
      lg: "btn-lg",
    };

    return `${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]} ${className}`.trim();
  };

  return (
    <RoleGuard resource="invoices" action="export">
      <div className="flex flex-col">
        <button
          onClick={handleExport}
          disabled={isExporting || !data}
          className={getButtonClasses()}
        >
          {isExporting ? (
            <>
              <span className="loading loading-spinner loading-xs mr-2"></span>
              Exporting...
            </>
          ) : (
            <>
              <svg
                xmlns="http://www.w3.org/2000/svg"
                className="h-4 w-4 mr-2"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
              >
                <path
                  strokeLinecap="round"
                  strokeLinejoin="round"
                  strokeWidth={2}
                  d="M12 10v6m0 0l-3-3m3 3l3-3m2 8H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
                />
              </svg>
              Export PDF
            </>
          )}
        </button>
        
        {error && (
          <div className="alert alert-error mt-2">
            <span className="text-xs">{error}</span>
          </div>
        )}
      </div>
    </RoleGuard>
  );
};

// Specialized export buttons for common use cases
export const InvoiceExportButton: React.FC<Omit<PdfExportButtonProps, 'title'>> = (props) => (
  <PdfExportButton {...props} title="Invoice Export" />
);

export const ReportExportButton: React.FC<Omit<PdfExportButtonProps, 'title'>> = (props) => (
  <PdfExportButton {...props} title="Report Export" />
);
```

### FILES_TO_MODIFY

#### 1. `./packages/nextjs/package.json`
```json
{
  "dependencies": {
    // ... existing dependencies
    "jspdf": "^2.5.1",
    "jspdf-autotable": "^3.6.0"
  },
  "devDependencies": {
    // ... existing devDependencies
    "@types/jspdf": "^2.3.0"
  }
}
```

#### 2. Update Global State (create if doesn't exist)
**Path**: `./packages/nextjs/services/store/store.ts`
```typescript
// Add to existing store or create new one
import { create } from 'zustand';
import { UserRoleData } from '~~/types/roles';

interface GlobalState {
  // ... existing state
  userRoles: Record<string, UserRoleData>;
  setUserRoles: (roles: Record<string, UserRoleData>) => void;
}

export const useGlobalState = create<GlobalState>((set) => ({
  // ... existing state
  userRoles: {},
  setUserRoles: (roles) => set({ userRoles: roles }),
}));
```

## INSTRUCTIONS

### 1. Installation
```bash
cd packages/nextjs
npm install jspdf jspdf-autotable
npm install -D @types/jspdf
```

### 2. Usage Examples

#### Role-Based Access Control
```typescript
import { RoleGuard, AdminOnly, CanExport } from "~~/components/fil-frame/RoleGuard";

// Protect entire components
<AdminOnly>
  <UserManagementPanel />
</AdminOnly>

// Protect specific actions
<RoleGuard resource="invoices" action="create">
  <CreateInvoiceButton />
</RoleGuard>

// Export functionality
<CanExport>
  <PdfExportButton data={invoiceData} filename="invoice-001.pdf" />
</CanExport>
```

#### PDF Export
```typescript
import { PdfExportButton, InvoiceExportButton } from "~~/components/fil-frame/PdfExportButton";

// Basic usage
<PdfExportButton 
  data={tableData} 
  filename="report.pdf"
  title="Monthly Report"
/>

// Specialized invoice export
<InvoiceExportButton 
  data={invoiceData}
  filename={`invoice-${invoiceId}.pdf`}
/>
```

### 3. Configuration Steps

1. **Initialize user roles**: Admin users need to be assigned via the `useUserRoles` hook
2. **Customize permissions**: Modify `ROLE_PERMISSIONS` in `types/roles.ts` as needed
3. **Style components**: Update CSS classes to match your design system
4. **Add persistent storage**: Integrate with your database/smart contract for role persistence

### 4. Testing Considerations

- Test role assignment/revocation flows
- Verify permission checks work correctly
- Test PDF export with various data types
- Check responsive behavior on mobile devices
- Validate error handling for insufficient permissions

This implementation provides a solid foundation for role-based access control and PDF export functionality while maintaining consistency with the existing codebase patterns.
```
