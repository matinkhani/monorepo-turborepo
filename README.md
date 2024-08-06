## What is Turborepo?

A tool from vercel that allows you to manage your monorepo in a more efficient way and run your taks much faster.

Turborepo understands your monorepo, it understands that your packages needs to be built before your apps can be built.

Turborepo also never does the same work twice. If you run a build it makes a hash out of them. If it doesn’t find that in its cache, then it just runs the build and then stores it in its cache for later. That means the next time that you run that build on the same source files it just doesn’t do the build, it restores what it had from the cache.

> *turborepo also has remote caching. with remote caching you can share the turborepo cache across your entire team and ci. resulting in even faster builds and days of time saved.*
> 

---

## What is Monorepo?

A monorepo is basically a collection of many different applications and packages in a single code base. the alternative setup is called *polyrepo (one project, one git repository)* and that is where you would have multiple code bases which are published and versioned separately. That helps us avoid a lot of code duplication between projects. But often you don’t need a monorepo until you get to a scale that you start needing multiple projects and start needing to have what we called packages in a monorepo.

Imagine you have three different separate repositories, one for your *nextjs* application, one for your *reactjs* application and one for your *shared package*, that can be a shared ui package that’s used in both your applications, or it can be shared configs for your apps.

When you want to make a change in the *shared package/utility* that’s used by *nextjs* app and *reactjs* app, you need to make that change and need to publish that to npm, and then need to update the *reactjs* and *nextjs* and bump up the version in their depenencies and then make a commit there and then your apps now sychronized with the new feature in this *shared package/utility*.

‌But if you want to do the same thing in a monorepo, they would all live in this same single code base. You can basically make any changes you want to the *utilities* and then it’s shared between the *reactjs* and *nextjs* apps. they’re not dependng on a specific version of this shared ui package or config package, they’re just depending on the local version in the same code base which makes it much more easier.

---

## Workspace

The main building block of a monorepo is workspaces, each application and package you build will be its own workspace. a workspace is basically a folder containing a `package.json` file, each workspace can declare its own dependencies, they can depend on each other, they can export code for other to use and they can have their own scripts.

---

## Package.json in different package managers

It doesn't matter which package manager you use (e.g., npm, pnpm, or yarn); in the `package.json` file for a *reactjs* or *nextjs* application or any other, we define dependencies for *shared packages*. This way, we specify that we want to use this library or package in this application.

npm or yarn:

```jsx
{
  "dependencies": {
    "next": "latest", // External dependency
    "@repo/ui": "*" // Internal dependency
  }
}
```

pnpm:

```jsx
{
  "dependencies": {
    "next": "latest", // External dependency
    "@repo/ui": "workspace:*" // Internal dependency
  }
}
```

> *these stars * indicate that it’s not resolving to any specific version, it’s just resolving to the one that’s in the workspace.*
> 

---

## Root Workspace

On top of the workspaces that you would have in your different applications and packages, there is also a root workspace that’s containing these whole packages and apps together, this is a good place for you to specify dependencies that are present across the entire monorepo. You can write task that belongs to the whole monorpeo as opposed to just individual workspaces.

root `package.json`:

npm:

```jsx
{
  "name": "TURBOREPO",
  "private": true,
  "scripts": {
    "build": "turbo build",
    "dev": "turbo dev",
    "lint": "turbo lint",
    "format": "prettier --write \"**/*.{ts,tsx,md}\""
  },
  "devDependencies": {
    "prettier": "^3.2.5",
    "turbo": "^2.0.11",
    "typescript": "^5.4.5"
  },
  "engines": {
    "node": ">=18"
  },
  "packageManager": "npm@10.8.1",
  "workspaces": [
    "apps/*",
    "packages/*"
  ]
}

```

pnpm:

```jsx
{
  "name": "TURBOREPO",
  "private": true,
  "scripts": {
    "build": "turbo build",
    "dev": "turbo dev",
    "lint": "turbo lint",
    "format": "prettier --write \"**/*.{ts,tsx,md}\""
  },
  "devDependencies": {
    "prettier": "^3.2.5",
    "turbo": "^2.0.11",
    "typescript": "^5.4.5"
  },
  "packageManager": "pnpm@8.15.6",
  "engines": {
    "node": ">=18"
  }
}
```

> *also in pnpm workspaces define in a file called `pnpm-workspace.yaml`:*
> 

```jsx
packages:
  - "apps/*"
  - "packages/*"
```

---

## Getting Started

Install turborepo globally so you can run turbo commands in your terminal (if you don’t want to use turbo commands you can use npm or pnpm or yarn):

npm: `npm install turbo --global`

yarn: `yarn global add turbo`

pnpm: `pnpm install turbo --global`

> *If you want you can install examples from turborepo document like (with-docker, with-prisma, with-tailwind, with-vue-nuxt and etc)*
> 

### Install Turborepo

`npx create-turbo@latest`

after installing you see starter repository with:

- two deployable nextjs applications in apps folder:
1. web
2. docs
- node_modules: it’s a shared folder of all the modules and packages we’re using between all of our apps and packages.
- packages: in the packages we are having different packages or libraries that we want to share between our apps:
1. eslint config: to share our eslint config between our apps.
2. typescript config: to share typescript configuration in our apps.
3. ui: a reactjs library to be shared between our apps.

Now you can see we’re having everything in the same code base which makes changing them easier and running build tasks and scripts faster.

### Turbo.json

On top of the root `package.json` we also have this `turbo.json` which defiens the tasks for running different scripts or tasks. Anything that you would want to run with turbo on `package.json` scripts needs to be registered in your `turbo.json` :

```jsx
{
  "$schema": "https://turbo.build/schema.json",
  "ui": "tui",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": ["$TURBO_DEFAULT$", ".env*"],
      "outputs": [".next/**", "!.next/cache/**"]
    },
    "lint": {
      "dependsOn": ["^lint"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

### Web App Package.json:

If we dive into web app which is a nextjs app, we can see a `package.json` over there like this:

```jsx
{
  "name": "web",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev --turbo",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@repo/ui": "*",
    "react": "19.0.0-rc-f994737d14-20240522",
    "react-dom": "19.0.0-rc-f994737d14-20240522",
    "next": "15.0.0-rc.0"
  },
  "devDependencies": {
    "@repo/eslint-config": "*",
    "@repo/typescript-config": "*",
    "typescript": "^5",
    "@types/node": "^20",
    "@types/react": "^18",
    "@types/react-dom": "^18",
    "eslint": "^8",
    "eslint-config-next": "15.0.0-rc.0"
  }
}
```

Now the name is important because that’s the name of our workspace and that’s how we’re going to refer to this specific workspace when we are running scripts or when we are defining dependencies between local workspaces.

Every package we build it’s own workspace, they can have their own scripts, they can have their own dependencies, they can depend on each other.

As you can see our web app depends on our ui library which is a workspace package in dependencies:

```jsx
 "dependencies": {
    ...
    "@repo/ui": "*",
    ...
  },
```

And we have some devDependencies, we’re depending on eslint config and tyspcript config that you see in the packages:

```jsx
  "devDependencies": {
    ...
    "@repo/eslint-config": "*",
    "@repo/typescript-config": "*",
    ...
  }
```

### Run Our Monorepo

Let’s go ahead and actually run our monorepo:

npm: `npm run dev`

yarn: `yarn dev`

pnpm: `pnpm run dev`

As you can see it is running the Web App and Docs App, so basically when we’re running `npm run dev` in this case it runs `turbo dev` in root `package.json` and this turbo will go to each of our packages and apps and run their dev script. So in the Web and Docs workspaces it is running the dev script which is two nextjs applications.

### How UI Library is gonna work in apps:

Let’s go in web/app/page.tsx and make some changes:

```jsx
import { Button } from "@repo/ui/button";

export default function Home() {
  return <Button appName="web">Click ME</Button>;
}
```

You can see the button is coming from our shared ui library and this made possible by refrencing this shared ui library in the web dependency.

If we go to that local ui workspace in the packages, in `package.json` we can see:

```jsx
{
  "name": "@repo/ui",
  "version": "0.0.0",
  "private": true,
  "exports": {
    "./button": "./src/button.tsx",
    "./card": "./src/card.tsx",
    "./code": "./src/code.tsx"
  },
  "scripts": {
    "lint": "eslint . --max-warnings 0",
    "generate:component": "turbo gen react-component"
  },
  "devDependencies": {
    "@repo/eslint-config": "*",
    "@repo/typescript-config": "*",
    "@turbo/gen": "^1.12.4",
    "@types/node": "^20.11.24",
    "@types/eslint": "^8.56.5",
    "@types/react": "^18.2.61",
    "@types/react-dom": "^18.2.19",
    "eslint": "^8.57.0",
    "typescript": "^5.3.3"
  },
  "dependencies": {
    "react": "^18.2.0"
  }
}
```

We can see “exports”, so if we want to use ui components in other packages or in the apps, we should export that component in here.

Or we can just do this to export all components:

```jsx
  "exports": {
    "./components/*": [
      "./src/components/*.tsx", // use src if you have a src folder
      "./src/components/*.ts" // same for the components
    ]
  },
```

That’s how easy it is to share a ui library between two completely separate nextjs application.

And also in the devDependencies you can see we use eslint and typescript configs. So we can use packages in packages.

### Typescript Configuration

Let’s quickly look at the shared typescript config since both of our Web and Docs applications are typescript applications. If we go to our pakcages/typescript-config, we can see there are three json files over here; one is base, one is nextjs and one is react-library. In the `package.json` we can see there is a name for this workspace and that’s how we can refrence this config as a dependency in our other workspaces:

```jsx
{
  "name": "@repo/typescript-config",
  "version": "0.0.0",
  "private": true,
  "license": "MIT",
  "publishConfig": {
    "access": "public"
  }
}
```

Now if we go back to the Web App and open `tsconfig.json` , we can see it extends the typescript-config workspace and it’s directly importing the `nextjs.json` file which allows us to just get those configs and extend those configs inside of our Web Application:

```jsx
{
  "extends": "@repo/typescript-config/nextjs.json",
  "compilerOptions": {
    "plugins": [
      {
        "name": "next"
      }
    ]
  },
  "include": [
    "next-env.d.ts",
    "next.config.mjs",
    "postcss.config.mjs",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts"
  ],
  "exclude": ["node_modules"]
}
```

### How to Run Separately

As you saw, we’re running the dev script on both of our applications in parallel together. Let’s see how we can run a dev script or any scripts on a specefic workspace:

npm: `npm run dev -w web`

pnpm: `pnpm --filter web run dev`

yarn: `yarn workspace web run dev`

> *you can use our workspace name instead of web*
> 

### How to add a dependency to a specific package

Let’s see how we would go about adding a dependency to a specific workspace. Now suppose we want to add axios to Web App, similiar to what we did in the previous step we can just pass a filter flag and specify the name of the workapce that we want to add the dependency:

pnpm: `pnpm --filter web add -D axios`

npm: `npm install axios -D -w web`

yarn: `yarn workspace web add --dev axios`

> *you can use our workspace name instead of web*
> 

And we can confirm this by looking at Web `package.json`:

```jsx
  "devDependencies": {
    ...
    "axios": "^1.7.3",
    ...
  }
```

But if we go to the Docs App and check out the `package.json` there, the axios is not present over here and definitely not in the root folder.

> *also you can change directory to the desired workspace and install your package.*
> 

---

## Build Process and Cache Behavior

Now let’s look at the build process and some of the caching behavior in turborepo that will beneficial. If we run `pnpm run build` , this is going to run the `turbo run build` which you can see in `turbo.json`:

```jsx
  "build": {
    "dependsOn": ["^build"],  
    "inputs": ["$TURBO_DEFAULT$", ".env*"],
    "outputs": [".next/**", "!.next/cache/**"]
  },
```

It runs the build script on every package or app that has a specified a build script.

![cache-miss.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/9612ae31-d95a-40ae-b4be-995b966afe3b/c78b92f6-4b3d-457e-b1e1-074bfea81884/cache-miss.jpeg)

Let’s look at the terminal and see what happend over here so as you can see it ran the build script on the Docs and Web workspace, you can see cache miss becuase if you haven’t run this build process before, so it’s executing the build script on this workspace and there is no cache.

Now let’s run the build process again:

![cache-hit.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/9612ae31-d95a-40ae-b4be-995b966afe3b/80aa8808-d7c0-41f0-8dae-575f168fa6e0/cache-hit.jpeg)

This time just happend so quickly and it is becuase the outputs haven’t changed since the previous run so cache hit, it’s just replaying the output we had from previous step, and compare the time of first build(cache miss) and second build(cache hit).

---

## Create a Turborepo with Shared Ui (Shadcn and Tailwindcss)

Now that we have learned how to setup workspaces and dependencies in a monorepo and shared ui or config packages using turborepo, let’s actually go ahead and create a turborepo with two nextjs apps and a shared ui library using shadcn and tailwindcss.

### Install Turborepo

`npx create-turbo@latest`

After running that command, it asks us these questions:

*Where would you like to create your Turborepo? .*

*Which package manager do you want to use? pnpm (i perefer to continue with pnpm)*

And after it’s finished installing, we’re gonna see this:

![installed-turborepo.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/9612ae31-d95a-40ae-b4be-995b966afe3b/57784dc8-d522-49d0-9bde-76c8247bd9f5/installed-turborepo.jpeg)

Now we have two nextjs apps and three packges. We’re going to add tailwind and shadcn to UI library.

### Install Tailwindcss & Shadcn

First of all we need to install tailwind css in our ui package:

`pnpm --filter ui add tailwindcss` 

And we can confirm this by looking at ui `package.json`:

```jsx
  "devDependencies": {
    ...
    "tailwindcss": "^3.4.7",
    ...
  },
```

After tailwindcss installed, we should create a tailwind config with run `npx tailwindcss init`, and we can see a `tailwind.config.js` created in ui package:

```jsx
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

And we need to create a `globals.css` file, i prefer to create it in src in ui package. And add the `@tailwind` directives into it:

`cd .\packages\ui\src\`

`echo. > globals.css`

And paste this lines into it:

```jsx
@tailwind base;
@tailwind components;
@tailwind utilities;
```

And after that we need to install shadcn, we’re going to change directory to packages/ui and install shadcn:

`cd .\packages\ui\`

`pnpm dlx shadcn-ui@latest init` 

After running that command, it asks us these questions:

*Would you like to use TypeScript (recommended)? yes*

*Which style would you like to use? Default (i perefer to use defualt style)*

*Which color would you like to use as base color? Slate*

*Where is your global CSS file? src/globals.css*

*Would you like to use CSS variables for colors? no*

*Are you using a custom tailwind prefix eg. tw-? ui-*

*Where is your tailwind.config.js located? » tailwind.config.js*

*Configure the import alias for components: » @repo/ui/components*

*Configure the import alias for utils: » @repo/ui/lib/utils*

*Are you using React Server Components? no*

 *Write configuration to components.json. Proceed? yes*

After installing is finished, you can see `lib` and `components` folders created in `src` and `components.json` created in root of ui:

```jsx
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "config": "tailwind.config.js",
    "css": "src/globals.css",
    "baseColor": "slate",
    "cssVariables": false,
    "prefix": "ui-"
  },
  "aliases": {
    "components": "src/components",
    "utils": "src/lib/utils"
  }
}
```

Now we should make this changes in `tsconfig.json` in ui:

```jsx
{
  "extends": "@repo/typescript-config/react-library.json",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@repo/ui/*": ["./src/*"]
    }
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

After that i want to make some changes in `tailwind.config.js` in ui package, first of all change the file name to `tailwind.config.ts` , and then paste this lines into that:

```jsx
import type { Config } from "tailwindcss";
import tailwindcssAnimate from "tailwindcss-animate";

const config = {
  darkMode: ["class"],
  content: [
    "./pages/**/*.{ts,tsx}",
    "./components/**/*.{ts,tsx}",
    "./app/**/*.{ts,tsx}",
    "./src/**/*.{ts,tsx}",
    "../../packages/ui/src/**/*.{ts,tsx}",
  ],
  prefix: "ui-",
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: {
        "2xl": "1400px",
      },
    },
    extend: {
      colors: {
        myColor: "#a16207",
      },
      keyframes: {
        "accordion-down": {
          from: { height: "0" },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: "0" },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
      },
    },
  },
  plugins: [tailwindcssAnimate],
} satisfies Config;

export default config;
```

Now let’s go ahead and add shadcn button to see how’s gonna work.

run `pnpm dlx shadcn-ui@latest add button`

and you can see the Button component created in `components/ui` folder:

```jsx
import * as React from "react";
import { Slot } from "@radix-ui/react-slot";
import { cva, type VariantProps } from "class-variance-authority";

import { cn } from "@repo/ui/lib/utils";

const buttonVariants = cva(
  "ui-inline-flex ui-items-center ui-justify-center ui-whitespace-nowrap ui-rounded-md ui-text-sm ui-font-medium ui-ring-offset-white ui-transition-colors focus-visible:ui-outline-none focus-visible:ui-ring-2 focus-visible:ui-ring-slate-950 focus-visible:ui-ring-offset-2 disabled:ui-pointer-events-none disabled:ui-opacity-50 dark:ui-ring-offset-slate-950 dark:focus-visible:ui-ring-slate-300",
  {
    variants: {
      variant: {
        default:
          "ui-bg-slate-900 ui-text-slate-50 hover:ui-bg-slate-900/90 dark:ui-bg-slate-50 dark:ui-text-slate-900 dark:hover:ui-bg-slate-50/90",
        destructive:
          "ui-bg-red-500 ui-text-slate-50 hover:ui-bg-red-500/90 dark:ui-bg-red-900 dark:ui-text-slate-50 dark:hover:ui-bg-red-900/90",
        outline:
          "ui-border ui-border-slate-200 ui-bg-white hover:ui-bg-slate-100 hover:ui-text-slate-900 dark:ui-border-slate-800 dark:ui-bg-slate-950 dark:hover:ui-bg-slate-800 dark:hover:ui-text-slate-50",
        secondary:
          "ui-bg-slate-100 ui-text-slate-900 hover:ui-bg-slate-100/80 dark:ui-bg-slate-800 dark:ui-text-slate-50 dark:hover:ui-bg-slate-800/80",
        ghost:
          "hover:ui-bg-slate-100 hover:ui-text-slate-900 dark:hover:ui-bg-slate-800 dark:hover:ui-text-slate-50",
        link: "ui-text-slate-900 ui-underline-offset-4 hover:ui-underline dark:ui-text-slate-50",
      },
      size: {
        default: "ui-h-10 ui-px-4 ui-py-2",
        sm: "ui-h-9 ui-rounded-md ui-px-3",
        lg: "ui-h-11 ui-rounded-md ui-px-8",
        icon: "ui-h-10 ui-w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : "button";
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  }
);
Button.displayName = "Button";

export { Button, buttonVariants };
```

If you need you can create a global `postcss.config.mjs` in ui package to use in other apps:

```jsx
/** @type {import('postcss-load-config').Config} */
const config = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};

export default config;
```

Now we’re going to ui’s `package.json` to change exports so we can use ui configs and ui components in other apps and packages:

```jsx
  "exports": {
    "./globals.css": "./src/globals.css",
    "./postcss.config": "./postcss.config.mjs",
    "./tailwind.config": "./tailwind.config.ts",
    "./lib/*": "./src/lib/*.ts",
    "./components/*": [
      "./src/components/*.tsx",
      "./src/components/*.ts"
    ]
  },
```

### How to use Shared Library in apps

We have created this ui library to use in all apps and also we can use tailwind config of ui library in all apps. So we have a main tailwind config and we can create many tailwind config from that.

For example, in web app we want to use main tailwind config and also our app have different classes and styles from main tailwind config, we can create a tailwind config for web and use main tailwind config and add web classes to it.

First of all, we need to install ui library in our app. if you didn’t delete the web and docs apps, they’re have ui library in their dependencies:

`web/package.json`:

```jsx
  "dependencies": {
	   ...
    "@repo/ui": "workspace:*",
		 ...
  },
```

If you haven’t ui in your app dependency, you must to add it like above and run `pnpm install`.

Now we can use all exported components and configs in our Web App.

First of all for using tailwind in web we should install tailwind in our app:

1.run `pnpm --filter web add -D tailwindcss postcss autoprefixer`

2.create `tailwind.config.ts` :

if you just want to use global tailwind config in Web App you just need do this:

```jsx
export * from "@repo/ui/tailwind.config";
```

but if you want also add new classes and styles just for use in Web App you can do this:

```jsx
import type { Config } from "tailwindcss";
import config from "@repo/ui/tailwind.config";

const webConfig = {
  ...config,
  presets: [config],
  theme: {
    extend: {
      colors: {
        test: {
          100: "#f2e8e5",
          200: "#eaddd7",
          300: "#e0cec7",
          400: "#d2bab0",
          500: "#bfa094",
          600: "#a18072",
          700: "#977669",
          800: "#846358",
          900: "#43302b",
        },
      },
    },
  },
} satisfies Config;

export default webConfig;
```

i just added a test color to see if it’s working.

and after that we have to create a `postcss.config.mjs` in web and paste this into it:

```jsx
export { default } from "@repo/ui/postcss.config";
```

and for the last part, we need to import our globals css into web `layout.tsx` like this:

```jsx
import "@repo/ui/globals.css";
```

Now we can use components and configs from ui in our Web App.

`web/app/page.tsx`:

```jsx
import { Button } from "@repo/ui/components/ui/button";

export default function Home() {
  return <Button variant="destructive">Click Me</Button>;
}
```

you can see we used button from ui package in web application.

That’s it! we have created a monorepo using turborepo and a ui library that’s sharing components and configs to all our apps.
