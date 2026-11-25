# Project Overview & Architecture
Full-stack application featuring a React + TypeScript frontend (Vite, Tailwind CSS, TanStack Query) and a structured RESTful API backend (.NET / Node).

---

## 1. Tooling & Commands

### Frontend (/client or root)
- Dev Server: npm run dev
- Type Check: npx tsc --noEmit
- Lint & Fix: npm run lint / npm run lint -- --fix
- Production Build: npm run build
- Unit / Component Tests: npm run test

### Backend (/server or /api)
- Run API: dotnet run (or npm run dev:server)
- Build / Compile: dotnet build (or npm run build:server)
- Run Tests: dotnet test (or npm run test:server)
- Database Migrations: Run explicit migration commands; never alter schemas via raw ad-hoc manual SQL.

---

## 2. Styling & Design System Rules (Tailwind CSS)

### Utility Class Discipline
- Use Tailwind utility classes for all layouts, spacing, colors, and typography.
- No Arbitrary Values: Avoid arbitrary pixel values like w-[342px] or mt-[13px]. Use standard Tailwind spacing scales (w-80, mt-3, gap-4).
- No Inline Styles: Disallow style={{ ... }} completely. Inline styles are strictly reserved for dynamic, runtime-computed properties (e.g., drag-and-drop coordinates, canvas math, dynamic SVG path values).
- Class Organization: Order classes systematically:
  1. Layout / Positioning (absolute, relative, flex, grid, items-center, justify-between)
  2. Box Model & Sizing (w-full, max-w-md, p-4, m-2)
  3. Typography & Colors (text-sm, font-semibold, text-slate-900, bg-white)
  4. Borders & Effects (rounded-xl, border, border-slate-200, shadow-sm)
  5. Interactive & Pseudo-states (hover:bg-slate-50, focus:ring-2, disabled:opacity-50)

### Responsive & Accessible Layouts
- Mobile-First Breakpoints: Design mobile first using bare utilities, then scale upward (sm:, md:, lg:, xl:). Never use desktop styles that get overridden at mobile.
- Accessibility:
  - Interactive elements must include visible :focus-visible ring outlines for keyboard navigation.
  - Icon-only buttons must specify an explicit aria-label.
  - Semantic HTML only: <button> for actions, <a> for navigation, <section>, <article>, <nav>.

### UI Component Architecture
- Place reusable primitive UI components in src/components/ui/ (Button, Input, Modal, Badge, Card).
- Use clsx and tailwind-merge (via a shared cn() helper) for merging conditional classes and handling variant overrides cleanly:
    import { clsx, type ClassValue } from "clsx";
    import { twMerge } from "tailwind-merge";
    export function cn(...inputs: ClassValue[]) { return twMerge(clsx(inputs)); }

---

## 3. Frontend Architecture (React + TypeScript)

### TypeScript Standards
- Respect strict: true across all compiler targets.
- Disallow any and unsafe type assertions (as unknown as ...). Use discriminated unions and explicit interfaces.
- Component prop types must be explicitly typed using interface ComponentNameProps. Do not use React.FC.
- Keep domain entities in src/types/ and share contract models with API responses.

### React Component & State Rules
- Component Style: Use functional components and hooks only.
- State Management:
  - Local UI State: useState / useReducer for inputs, modals, and local toggles.
  - Server / Remote State: Use TanStack Query (useQuery, useMutation). Do not manually orchestrate loading/error flags with useEffect.
  - Derived State: Calculate values directly during rendering or memoize with useMemo. Never duplicate props or state into synchronization useEffects.
- Component Size: Keep components under 200 lines. Break complex cards or views into sub-components.

---

## 4. Backend & API Architectural Standards

### RESTful API Contracts
- Endpoints must follow pluralized resource naming conventions (/api/v1/projects, /api/v1/users/{id}).
- Return standard HTTP status codes:
  - 200 OK (success with body), 201 Created (resource created), 204 No Content (success without body)
  - 400 Bad Request (validation failures), 401 Unauthorized (missing/invalid token), 403 Forbidden (permission denied)
  - 404 Not Found, 409 Conflict, 500 Internal Server Error

### Response Payload Structure
- Standard Success Response:
    { "data": { ... }, "error": null, "meta": { "page": 1, "pageSize": 20, "total": 100 } }
- Standard Error Response:
    { "data": null, "error": { "code": "VALIDATION_ERROR", "message": "Invalid payload provided.", "details": [{ "field": "email", "issue": "Must be a valid email format" }] } }

### Data Access & Validation
- Validate all incoming payloads before processing business logic (using Zod on Node or Data Annotations / FluentValidation on .NET).
- Never expose internal database entities or raw table models directly over the wire. Always map to explicit Request/Response DTOs.
- Use parameterized queries or ORM abstractions to prevent SQL injection.
- Wrap multi-table write operations inside atomic database transactions.

---

## 5. Security & Environment Configuration

- Secrets Isolation: Never hardcode secrets, API keys, or connection strings. Access all configuration through environment variables (.env.local, appsettings.json, or environment variables).
- Client Boundary: Prefix client-accessible environment variables strictly according to bundler rules (VITE_* or NEXT_PUBLIC_*). Never expose database credentials or third-party service private keys to the client bundle.
- CORS: Configure explicit origin whitelists; avoid wildcards (*) for routes handling credentials or auth cookies.

---

## 6. Claude Operational Rules & Execution Workflow

### Step-by-Step Task Execution Protocol
- Plan Mode First: When working on multi-file features or database migrations, provide a concise 3-point implementation plan detailing affected files before writing or modifying any code.
- Atomic Edits: Modify only the files directly relevant to the current objective. Do not reformat untouched files, reorganize imports globally, or perform unsolicited refactorings.
- Zero Package Hallucinations: Never add new libraries or modify package.json / .csproj without explicit confirmation from the user.
- Preserve Existing Logic: When updating components or endpoints, keep existing business logic, validation rules, error handling, accessibility tags, and edge cases intact.

### Verification & Self-Correction Feedback Loop
- Compile Verification: Mentally simulate or run npx tsc --noEmit and backend build commands before declaring any task complete. Never output code that introduces type-checking errors under strict mode.
- Automated Fix Loop: If a linting or type error is flagged, diagnose the root cause immediately, fix the error in-place, and verify zero regressions.
- Concise, Functional Code: Output production-ready, complete code blocks. Do not replace implementation bodies with placeholders like // TODO: implement later or /* ... existing code ... */.