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

- Dynamic routes :

  - `/users/[userId]` maps to `/users/[userId]/page.tsx`

- Nested routes :
  - `/users/[userId]/posts/[postId]` : folders inside folders
  - Every group can have `layout.tsx` for their own custom layout

## Next/Link vs Next/Navigation

- `a` tag performs a full page reload
- `next/link` only performs a client-side navigation

  - Next.js automatically code splits your application by route segments. This means that the code for a given route is only loaded when the user navigates to that route.
  - Next.js automatically prefetches the page data for the next route so that the user doesn't have to wait for the page to load when they navigate to the next route.

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
