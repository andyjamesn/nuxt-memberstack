- [Memberstack Module for Nuxt 3](#memberstack-module-for-nuxt-3)
  - [Setup - Get Started](#setup---get-started)
    - [Requirements](#requirements)
    - [Nuxt Config](#nuxt-config)
  - [Using the module](#using-the-module)
    - [Global instance available](#global-instance-available)
    - [Composables](#composables)
      - [Auth Composable](#auth-composable)
        - [Login](#login)
        - [Reset password](#reset-password)
          - [Request token email](#request-token-email)
          - [Use token to reset password](#use-token-to-reset-password)
      - [Member Composable](#member-composable)
  - [Plans](#plans)
    - [Setting a plan (Saving it to store/state)](#setting-a-plan-saving-it-to-storestate)
    - [Displaying plans](#displaying-plans)
  - [Middleware for route protection and redirects](#middleware-for-route-protection-and-redirects)
    - [Global Middleware - On All routes](#global-middleware---on-all-routes)
    - [Named middleware - On specific routes](#named-middleware---on-specific-routes)
    - [Memberstack Role based Middleware](#memberstack-role-based-middleware)
  - [Development](#development)

# Memberstack Module for Nuxt 3

This module is a wrapper around the `@memberstack/dom package`.

The aim is to make the Memberstack object available across your Nuxt application and implements the Composition API.

Under the hood the Memberstack API get and update calls update a Pinia store through composables to easily make all data available across your application and also makes everything reactive.

Example of using the `useMemberStackMember()` to get and update user data. Since all values are using Pinia with `storeToRefs` all values are reactive and the data in the pages update as changes are made.

Here is an example of using the member composable

```javascript
const { getMember, updateMember } = useMemberStackMember()
```

Using the above composable will update the DOM when a user logs in so the user data is updated and displayed on the page. Also when updating the members data the page updates without the need to reload.

## Setup - Get Started

### Requirements

This module (@nuxt/memberstack) and Pinia (@nuxt/pinia) are required.

_Pinia is already installed as a dependancy of the module, but it does need to be set in your nuxt.config.js as a module._

### Nuxt Config

You can add your Memberstack public key directly into the config but we suggest adding it as an environment variable in a `.env` file

```javascript
  modules: ['@nuxt/memberstack', '@pinia/nuxt'],
  memberstack: {
    publicKey: process.env.MEMBERSTACK_PUBLIC_KEY
  },
```

## Using the module

### Global instance available

A global instance of the Memberstack object is available as part of the Nuxt context.

This means anything you can do in the Memberstack documentation you can **do in Nuxt anywhere**

```Javascript
const { $memberstack } = useNuxtApp()
```

Then you can use it how you normally would as per the Memberstack Docs, eg:

```Javascript
const member = await $memberstack.getCurrentMember()
```

**Please note:** You will need to build in the reactivity yourself as this interacts directly with Memberstack. If you want built in reactivity we suggest you use the composables provided.

### Composables

#### Auth Composable

```javascript
const auth = useMemberStackAuth()
```

Then call any methods in the composable directly

```javascript
await auth.login()
```

Or destructure out all options

```javascript
const { login, logout, register } = useMemberStackAuth()
```

##### Login

When a user is logged in the user data is added to the global Reactive member object.

You can set a redirect to redirect the user to a specific page after login.

```Javascript
await login({
  email: 'john@doe.com',
  password: '123123123',
  redirect: 'dashboard'
})
```

##### Reset password

There is no composable for this as there it no need to update the state

Get memberstack context from Nuxt and use to trigger reset email

```javascript
const { $memberstack: memberstack } = useNuxtApp()
```

###### Request token email

```Javascript
await memberstack.sendMemberResetPasswordEmail({
  email: "john@doe.com",
})
```

###### Use token to reset password

After token is received, use this call to reset the password. You can also set a redirect back to the login page after password has been successfully reset.

The member object is returned if you want to use it to display a message to the user. But the user does need to login again.

```Javascript
const member = await resetPassword({
  token: '123456',
  newPassword: '54321qwerty',
  redirect: '/login'
})
```

#### Member Composable

```javascript
const { getMember: member, updateMember } = useMemberStackMember()
```

```jsx
<template>
  Welcome, {{member.auth.email}}
</template>
```

## Plans

Use the `useMemberStackPlans()` composable to set and get plans.

```javascript
const { setPlan, setPlans, plan, plans } = useMemberStackPlans()
```

Setting a plan or plans makes it available as reactive state in the store. Once set you can then access the them and display them on your page

### Setting a plan (Saving it to store/state)

Set a single plan

```javascript
await setPlan('pln_cl1....')
```

Set all active or inactive plans

```javascript
await setPlans('ACTIVE')
```

### Displaying plans

Now you can use them in your page

```jsx
<template>
  <div v-for="(plan, index) in plans" :key="index">{{ plan.name }}</div>
</template>
```

## Middleware for route protection and redirects

### Global Middleware - On All routes

Create a global middleware file: eg `./middleware/auth.global.ts`

This will run on all routes and redirect a NOT logged in user to the login page.

```javascript
export default defineNuxtRouteMiddleware((to) => {
  const { getMember: member } = useMemberStackMember()

  // We need to specify the login page to prevent a circular route error
  if (to.path !== '/login' && !member.value) {
    return navigateTo('/login')
  }
})
```

### Named middleware - On specific routes

Create a global middleware file: eg `./middleware/auth.ts`

```javascript
export default defineNuxtRouteMiddleware((to, _from) => {
  const { getMember: member } = useMemberStackMember()

  if (!member.value) {
    return navigateTo('/login')
  }
})
```

For named routes you need to specify in the page which middleware to use

```jsx
<script setup lang="ts">
definePageMeta({
  middleware: ['auth']
})
</script>
```

### Memberstack Role based Middleware

You can check if a user has a certain role.

For example check if a user is an admin and if not redirect them

Create a global middleware file: eg `./middleware/auth-admin.ts`

```javascript
export default defineNuxtRouteMiddleware(() => {
  const { getMember: member } = useMemberStackMember()

  if (typeof window !== 'undefined') {
    if (member !== null) {
      const { permissions } = member.value

      const hasAdminPermissions = permissions.includes('admin')
      if (!hasAdminPermissions) {
        return navigateTo('/pricing')
      }
    }
  }
})
```

Now on any pages where only admin users are allowed you just need to define the middleware

```jsx
<script setup lang="ts">
definePageMeta({
  middleware: ['auth-admin']
})
</script>
```

## Development

- Run `npm run dev:prepare` to generate type stubs.
- Use `npm run dev` to start [playground](./playground) in development mode.
