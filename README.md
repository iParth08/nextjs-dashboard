# Learn NEXTJS with DOCUMENTAION

## Package Manager

- Changing package manager to pnpm

```bash
    npm install -g pnpm
```

## Download Sample Project

```bash
npx create-next-app@latest nextjs-dashboard --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example" --use-pnpm

```

## File Structure

1. `app` - The main directory for your application code.

   - Routes
   - Apis
   - Components
   - UI , Lib..etc

2. `layout.tsx` - The main layout for your application and parent for pages
3. `page.tsx` - The pages that get rendered on the client-side.
4. `global.css` - Global styles for your application.
5. `route.tsx` - The routes that get rendered on the server-side.
6. `loading.tsx` - The loading UI for your application, for complete page loading.
7. `error.tsx` - The error UI for your application, for all error page.
8. `not-found.tsx` - The not-found UI for your application, for all 404 page.

## Tailwind CSS

1. Make Shapes with Border class

   ```jsx
   <div className="relative w-0 h-0 border-l-[15px] border-r-[15px] border-b-[26px] border-l-transparent border-r-transparent border-b-black" />
   ```

   - with css modules and limit scope (resolve conflict & reuseable)

   ```jsx
   import styles from "@/app/ui/home.module.css";

   return <div className={styles.shape} />;
   ```

2. Conditional Styling with `clsx` or `cn`

   - [clsx](https://github.com/lukeed/clsx)

   ```jsx
   import clsx from "clsx";

   return (
     <span
      className={clsx(
        'inline-flex items-center rounded-full px-2 py-1 text-xs',
        {
          'bg-gray-100 text-gray-500': status === 'pending',
          'bg-green-500 text-white': status === 'paid',
        },
      )}
    >
   );

   <!-- Syntax -->

   className={clsx( "static class",
   {
    "dynamic class": condition,
    "dynamic class": condition,
   }
   ```

## Next/Font

- `next/font` automatically loads the fonts in build time and serve it as static assets reducing chances of font loading issues or _layout shift_

- app/ui/`fonts.ts`

  ```js
  import { Inter, Lusitana } from "next/font/google";

  export const inter = Inter({ subsets: ["latin"] });
  export const lusitana = Lusitana({ weight: ["400"], subsets: ["latin"] });
  ```

## Next/Image

- it supports lazy loading
- it supports responsive images
- It's good practice to set the `width` and `height` of your images to avoid layout shift, these should be an aspect ratio _identical_ to the source image.

  ```jsx
  import Image from "next/image";

  <Image
  src={invoice.image_url}
  alt={`${invoice.name}'s profile picture`}
  className="h-10 w-10 rounded-full hidden md:block"
  width={40}
  height={40}
  />;
  <!-- Hidden on mobile -->
  ```

## More Page Routes

- File system based routing :
  @ [docs-for-route](https://nextjs.org/docs/app/building-your-application/routing#colocation)

  - Each folder represents a route segment that maps to a URL segment.
  - `/` maps to `/page.tsx`
  - `/about` maps to `/about/page.tsx`

- [Dynamic] routes :

  - `/users/[userId]` maps to `/users/[userId]/page.tsx`
  - [slug] takes dynamic value from user/server

- Nested routes :

  - `/users/[userId]/posts/[postId]` : folders inside folders
  - Every group can have `layout.tsx` for their own custom layout

- (Group) :
  Route groups (not a part of URL), holds common logic for a group of routes.

## Next/Link vs Next/Navigation

### Next/Link

- `a` tag performs a full page reload
- `next/link` only performs a client-side navigation

  - Next.js automatically code splits your application by route segments. This means that the code for a given route is only loaded when the user navigates to that route.
  - Next.js automatically prefetches the page data for the next route so that the user doesn't have to wait for the page to load when they navigate to the next route.

### Next/Navigation [CLIENT-SIDE]

- [DOCS](https://nextjs.org/learn/dashboard-app/adding-search-and-pagination)
- `usePathname` hook to get current route path from _next/navigation_

  - It is strickly client-side

  ```tsx
  "use client";

  import { usePathname } from "next/navigation";
  //....

  const pathname = usePathname();

  return (
    <Link
      key={link.name}
      href={link.href}
      className={clsx(
        "...",
        pathname === link.href && "bg-sky-100 text-blue-600"
      )}
    >
      Link will be highlighted
    </Link>
  );
  ```

- `useSearchParams` hook to get query params from _next/navigation_

  - It is a hook for client side query params

  ```tsx
  "use client";

  import { useSearchParams, usePathname, useRouter } from "next/navigation";

  export default function Search() {
    const searchParams = useSearchParams();
    const pathname = usePathname();
    const { replace } = useRouter();

    function handleSearch(term: string) {
      const params = new URLSearchParams(searchParams);
      if (term) {
        params.set("query", term);
      } else {
        params.delete("query");
      }
      replace(`${pathname}?${params.toString()}`);
    }
  }
  ```

- For server side, resolve `searchParams` promise with `await`

```tsx
export default async function Page(props: {
  searchParams?: Promise<{
    query?: string;
    page?: string;
  }>;
}) {
  const searchParams = await props.searchParams;
  const query = searchParams?.query || "";
  const currentPage = Number(searchParams?.page) || 1;
  //...
}
```

- `useRouter` hook to navigate between pages programmatically
- `redirect` function to redirect to another page

  ```tsx
  import { redirect } from "next/navigation";

  //...

  redirect("/");
  ```

> [!WARNING]
>
> _defaultValue vs. value / Controlled vs. Uncontrolled_
>
> - If you're using state to manage the value of an input, you'd use the value attribute to make it a controlled component. This means React would manage the input's state.
>
> - However, since you're not using state, you can use defaultValue. This means the native input will manage its own state. This is okay since you're saving the search query to the URL instead of state.

- **Debouncing** can help reduce number of API calls/DB Queries
  - use `useDebouncedCallback` from `use-debounce`

```tsx
"use client";

import { useSearchParams, usePathname, useRouter } from "next/navigation";
import { useDebouncedCallback } from "use-debounce";

export default function Search() {
  const searchParams = useSearchParams();
  const pathname = usePathname();
  const { replace } = useRouter();

  //note: limit the search to 300ms
  const handleSearch = useDebouncedCallback((term) => {
    console.log(`Searching... ${term}`);

    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set("query", term);
    } else {
      params.delete("query");
    }
    replace(`${pathname}?${params.toString()}`);
  }, 300);
}
```

## Database Postgres with Neon (Vercel)

### Setup

1. Install SDK

   ```bash
        pnpm i @vercel/postgres
   ```

2. `app/seed/route.ts` to connect to database

   ```tsx
   import { db } from "@vercel/postgres";

   const client = await db.connect();
   ```

   - this connected to database once using `db.connect`
   - this is a one time run to seed the database with placeholder data

3. `app/query/route.ts` to query database

   ```tsx
   export async function GET() {
     try {
       return Response.json(await listInvoices());
     } catch (error) {
       return Response.json({ error }, { status: 500 });
     }
   }
   ```

- listInvoices function to fetch data from database,
- it uses sql query to fetch data from database

```tsx
async function listInvoices() {
  const data = await client.sql`
    SELECT invoices.amount, customers.name
    FROM invoices
    JOIN customers ON invoices.customer_id = customers.id
    WHERE invoices.amount = 666;
  `;

  return data.rows;
}
```

### Fetch Data from Database

1. _API_

- Async/Await or Promises can be resolved to fetch data from database
- It works perfectly in server components

> [!WARNING]
>
> - Client-side can exposes db secret key
> - Client-side will require API layer and useEffect, useState to resolve
>   async/await to fetch data.

2. _SQL_ for relational databases

> [!IMPORTANT]
>
> - Use SQL query to fetch data from database
> - The Vercel Postgres SDK provides protection against SQL injections
> - You can call sql inside any Server Component.
> - SQL is best for data fetching and manipulation

3. _Request Waterfall_

- A "waterfall" refers to a sequence of network requests that depend on the completion of previous requests.
- In a waterfall, the next request is only sent after the previous request has completed.

> [!WARNING]
>
> It is good when fetching should start after once condition is met.
> But it is not good for fetching data in real-time, unrelated to each other.

4. _Parallel Requests_
   - using `Promise.all` or `Promise.allSettled` to fetch data in parallel
   - Learn it in JS

```js
const data = await Promise.all([
  invoiceCountPromise,
  customerCountPromise,
  invoiceStatusPromise,
]);
```

### Data Mutation (Server Actions)

- _What is Data Mutation?_
  CRUD operations on database and data.

- _What are Server Actions?_
  React Server Actions allow you to run asynchronous code directly on the server. They eliminate the need to create API endpoints to mutate your data. Instead, you write asynchronous functions that execute on the server and can be invoked from your Client or Server Components.

  - **Javascript Disabled**
    An advantage of invoking a Server Action within a Server Component is progressive enhancement - forms work even if JavaScript is disabled on the client.

  - **Revalidate Data**
    You can also revalidate the associated cache using APIs like `revalidatePath` and `revalidateTag`.

#### Server Actions

1. Create a **server action** file

   _lib/_`actions.ts`

   ```ts
   "use server";
   import { sql } from "@vercel/postgres";
   import { revalidatePath } from "next/cache";

   export async function createInvoice(formData: FormData) {
     const { customerId, amount, status } = CreateInvoice.parse({
       customerId: formData.get("customerId"),
       amount: formData.get("amount"),
       status: formData.get("status"),
     });
     //OR : const rawFormData = Object.fromEntries(formData.entries())
     const amountInCents = amount * 100;
     const date = new Date().toISOString().split("T")[0];

     // Insert the invoice in the database
     await sql`
            INSERT INTO invoices (customer_id, amount, status, date)
            VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
        `;

     revalidatePath("/dashboard/invoices");
     //refresh for new data
   }
   ```

   - By adding the `use server`, you mark all the exported functions within the file as Server Actions. These server functions can then be imported and used in Client and Server components.

   - Behind the scenes, Server Actions create a POST API endpoint. This is why you don't need to create API endpoints manually when using Server Actions.

2. **Binding ID or Args to Server Actions**

```tsx
import { createInvoice } from "@/app/lib/actions";

//...

const createInvoiceId = createInvoice.bind(null, id);
return (
<form action={createInvoiceId}>
  <input type="text" name="customerId" />
  <input type="text" name="amount" />
  <input type="text" name="status" />
  <button type="submit">Create Invoice</button>
</form>
);

<!-- Invalid for Server Actions -->
<form action={createInvoice(id)}>
```

## Error Handling

[DOCS](https://nextjs.org/learn/dashboard-app/error-handling)

1. Create a **error page** file

   - It must be named `error.tsx`
   - It must be a client component

   _app/_`error.tsx`

   ```tsx
   "use client";

   import { useEffect } from "react";

   export default function Error({
     error,
     reset,
   }: {
     error: Error & { digest?: string };
     reset: () => void;
   }) {
     useEffect(() => {
       // Optionally log the error to an error reporting service
       console.error(error);
     }, [error]);

     return (
       <main className="flex h-full flex-col items-center justify-center">
         <h2 className="text-center">Something went wrong!</h2>
         <button
           className="mt-4 rounded-md bg-blue-500 px-4 py-2 text-sm text-white transition-colors hover:bg-blue-400"
           onClick={
             // Attempt to recover by trying to re-render the invoices route
             () => reset()
           }
         >
           Try again
         </button>
       </main>
     );
   }
   ```

2. **Try-Catch** block

   ```tsx
   try {
     await sql`
              UPDATE invoices
              SET customer_id = ${customerId}, amount = ${amountInCents}, status = ${status}
              WHERE id = ${id}
            `;
   } catch (error) {
     return { message: "Database Error: Failed to Update Invoice." };
   }

   revalidatePath("/dashboard/invoices");
   redirect("/dashboard/invoices");
   ```

> [!WARNING]
>
> - redirect works by throwing an error, which would be caught by the catch block.
> - To avoid this, you can call redirect after try/catch. redirect would only be reachable if try is successful.

3. **Throw Error**

   > Throw a custom error for testing

   ```tsx
   export async function deleteInvoice(id: string) {
     throw new Error("Failed to Delete Invoice"); //custom error

     // Unreachable code block
     try {
       await sql`DELETE FROM invoices WHERE id = ${id}`;
       revalidatePath("/dashboard/invoices");
       return { message: "Deleted Invoice" };
     } catch (error) {
       return { message: "Database Error: Failed to Delete Invoice" };
     }
   }
   ```

4. **Not Found error**

   ```tsx
   import { notFound } from "next/navigation";

   export async function page(invoice: string) {
     if (!invoice) {
       notFound();
     }
   }
   ```

> [!NOTE]
>
> A `not-found.tsx` page is rendered when a route is not found.

## Static and Dynamic Rendering

- _What is Static Rendering?_
  With _static rendering_, data fetching and rendering happens on the server at build time (when you deploy) or when revalidating data.

- _What is Dynamic Rendering?_
  With dynamic rendering, content is rendered on the server for each user at request time (when the user visits the page).

  - Params and Path URL are passed to the component

- _What is Incremental Static Regeneration?_
  With incremental static regeneration, data fetching and rendering happens on the server at request time, but only for the specific page that was requested.

### Streaming

- _What is Streaming?_
  Streaming is a way to render data from a server to a client in a real-time manner, without having to fetch all the data at once.

  Streaming is a data transfer technique that allows you to break down a route into smaller "chunks" and progressively stream them from the server to the client as they become ready.

  - Tackle slow data fetching
  - [docs](https://nextjs.org/learn/dashboard-app/streaming)

- `loading.tsx`
  Loading page works for the whole page rendering and not specific component or segment

- `Suspense`
  Suspense allows you to defer rendering parts of your application until some condition is met (e.g. data is loaded).

  - _dashboard/(overview)/page.tsx_

  ```tsx
  import { Suspense } from "react";
  import { CardsSkeleton } from "./ui/skeletons";
  import { CardWrapper } from "./ui/dashboard/cards";

  export default function Page() {
    return (
      <Suspense fallback={<CardsSkeleton />}>
        <CardWrapper />
      </Suspense>
    );
  }
  ```

  - _ui/dashboard/cards.tsx_

  ```tsx
  import { fetchCardData } from "@/app/lib/data";

  export default function CardWrapper() {
    const {
    numberOfInvoices,
    numberOfCustomers,
    totalPaidInvoices,
    totalPendingInvoices,
    } = await fetchCardData();

    return (
      // ...
    );
  }

  ```

## Authentication and Authorization

- _What is Authentication?_
  Authentication is the process of verifying that a user is who they claim to be.

  A secure website often uses multiple ways to check a user's identity. For instance, after entering your username and password, the site may send a verification code to your device or use an external app like Google Authenticator. This 2-factor authentication (2FA) helps increase security. Even if someone learns your password, they can't access your account without your unique token.

- _What is Authorization?_
  Authorization is the process of determining what resources a user has access to.

- _What is NextAuth?_
  NextAuth.js is an authentication library for Next.js that provides a simple and secure way to add authentication to your Next.js application.

# Additionals

## Visualization

```js
// This component is representational only.
// For data visualization UI, check out:
// https://www.tremor.so/
// https://www.chartjs.org/
// https://airbnb.io/visx/
```

## Validation [Zod](https://zod.dev/)

```ts
import { z } from "zod";

const FormSchema = z.object({
  id: z.string(),
  customerId: z.string(),
  amount: z.coerce.number(),
  status: z.enum(["pending", "paid"]),
  date: z.string(),
});

const CreateInvoice = FormSchema.omit({ id: true, date: true });
```

- Validating with error messages

```tsx
const FormSchema = z.object({
  id: z.string(),
  customerId: z.string({
    invalid_type_error: "Please select a customer.",
  }),
  amount: z.coerce
    .number()
    .gt(0, { message: "Please enter an amount greater than $0." }),
  status: z.enum(["pending", "paid"], {
    invalid_type_error: "Please select an invoice status.",
  }),
  date: z.string(),
});
```

## Improving Accessibility

> [!NOTE]
>
> - Let's see how to implement server-side validation with Server Actions, and how you can show form errors using React's `useActionState` hook - while keeping accessibility in mind!
>
> - It's a vast topic that covers many areas, such as keyboard navigation, semantic HTML, images, colors, videos, etc.

1. `lint` : warns about semantic issues

   ```json
   <!-- package.json -->

    "scripts": {
    "build": "next build",
    "dev": "next dev --turbopack",
    "start": "next start",
    "lint": "next lint" //this enables linting
   },
   ```

   - run lint with `pnpm lint`

2. `useActionState` hook
   > [!NOTE]
   >
   > - For form validation, we can use the `useActionState` hook to show form errors.
   > - It is a hook that allows you to access the state of a form submission from a server action.

**By validating forms on the server, you can:**

- Ensure your data is in the expected format before sending it to your database.
- Reduce the risk of malicious users bypassing client-side validation.
- Have one source of truth for what is considered valid data.

```tsx
"use client";

import { useActionState } from "react";
import { createInvoice, State } from "./actions";

export default function Form() {
  const initialState: State = { message: null, errors: {} };
  const [state, formAction] = useActionState(createInvoice, initialState);

  return <form action={formAction}>{/* ... */}</form>;
}
```

**actions.ts**

```ts
export type State = {
  errors?: {
    customerId?: string[];
    amount?: string[];
    status?: string[];
  };
  message?: string | null;
};

export async function createInvoice(prevState: State, formData: FormData) {
  // ...safeParse : safeParse() will return an object containing either a success or error field.
  const validatedFields = CreateInvoice.safeParse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      message: "Missing Fields. Failed to Create Invoice.",
    };
  }
}
```

**Error placement: aria-related**

```tsx
<select
  id="customer"
  name="customerId"
  className="..."
  defaultValue=""
  aria-describedby="customer-error" //related
>
  {" "}
</select>

<div id="customer-error" aria-live="polite" aria-atomic="true">
    {state.errors?.customerId &&
      state.errors.customerId.map((error: string) => (
        <p className="mt-2 text-sm text-red-500" key={error}>
         {error}
        </p>
    ))}
  </div>
```

# ISSUES

1. Issue with form action and bind

   ```jsx
   // const deleteInvoiceWithId = deleteInvoice.bind(null, id);
   const deleteInvoiceAction = async (formData: FormData) => {
     try {
       await deleteInvoice(id); // Call your delete function
     } catch (error) {
       console.error("Failed to delete invoice:", error);
     }
   };
   return <form action={deleteInvoiceAction}>{/* ... */}</form>;
   ```

2. Server component not working properly on VERCEL
