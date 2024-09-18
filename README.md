# Supabase ❤️ Next.js

Use Supabase Auth in your Next.js app with ease.

## Installation

```sh
npm install @supabase/nextjs
```

## Usage

Configure Supabase using environment variables in your `.env.local` file:

```bash
NEXT_PUBLIC_SUPABASE_URL="<your-supabase-url>"
NEXT_PUBLIC_SUPABASE_ANON_KEY="<your-anon-key>"
```

Use `supabaseMiddleware` in your `middleware.ts` file:

```ts
import { supabaseMiddleware, createRouteMatcher } from '@supabase/nextjs/server'

const isPublicRoute = createRouteMatcher(['/login(.*)', '/signup(.*)'])

export default supabaseMiddleware(
  async (auth, request) => {
    const session = await auth()

    // protect all routes except the public ones
    if (!isPublicRoute(request) && !session.user) {
      return session.redirectToSignIn()
    }

    // redirect to home if user is logged in and on public route
    if (isPublicRoute(request) && session.user) {
      return session.redirectToHome()
    }
  },
  {
    paths: {
      // custom signIn path
      signIn: '/login'
    }
  }
)

export const config = {
  // don't run the middleware on static assets
  matcher: ['/((?!api|_next/static|_next/image|.*\\.png$).*)']
}
```

In your components, use the Supabase Client to handle authentication:

### Sign up

```tsx
// app/signup/route.tsx
import { createClient } from '@supabase/nextjs/server'

export function SignUpForm() {
  async function signUp(formData: FormData) {
    'use server'
    const email = formData.get('email')
    const password = formData.get('password')
    const supabase = createClient()
    const { error } = await supabase.auth.signUp({
      email,
      password
    })
  }

  return (
    <form action={signUp}>
      <input type="email" name="email" />
      <input type="password" name="password" />
      <button>Sign Up</button>
    </form>
  )
}
```

### Sign in

```tsx
// app/login/route.tsx
import { createClient } from '@supabase/nextjs/server'

export function SignInForm() {
  async function signIn(formData: FormData) {
    'use server'
    const email = formData.get('email')
    const password = formData.get('password')
    const supabase = createClient()
    const { error } = await supabase.auth.signIn({
      email,
      password
    })
  }

  return (
    <form action={signIn}>
      <input type="email" name="email" />
      <input type="password" name="password" />
      <button>Sign in</button>
    </form>
  )
}
```

### Sign out

```tsx
// components/logout.tsx
import { redirect } from 'next/navigation'
import { createClient } from '@supabase/nextjs/server'

export function SignOutForm() {
  async function signOut() {
    'use server'
    const supabase = createClient()
    await supabase.auth.signOut()
    redirect('/')
  }

  return (
    <form action={signOut}>
      <button>Sign Out</button>
    </form>
  )
}
```

## Credits

Props to [Clerk](https://clerk.com) for their excellent middleware API.