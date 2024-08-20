---

Create a Turborepo with NextJs & Tailwindcss & Shadcn
Turborepo, developed by Vercel, is a powerful tool that makes managing monorepos easier by optimizing workflows, speeding up build times, and ensuring consistency across projects. In this article, we'll explain what monorepos are and guide you through setting up a Turborepo with Next.js, Tailwind CSS, and Shadcn - a UI library that enhances your design process. By the end, you'll know how to create a scalable monorepo setup that improves development and ensures smooth integration across your projects. Whether you're handling multiple projects or just looking to streamline your workflow, this guide will provide the tools and knowledge you need.
What is Monorepo?
A monorepo is essentially a single codebase that contains multiple applications and packages. The alternative setup is called a polyrepo (one project, one Git repository), where each project has its own codebase, published and versioned separately. Monorepos help avoid code duplication between projects and can be particularly useful as you scale and start needing multiple projects, each potentially containing shared packages.
For example, imagine you have three separate repositories: one for your Next.js application, one for your React.js application, and one for a shared package. This shared package could be a UI component library or shared configuration files used by both applications.
In a polyrepo setup, if you want to make a change to the shared package used by both the Next.js and React.js apps, you would need to:
Make the change in the shared package.
Publish the updated package to npm.
Update the dependencies in both the React.js and Next.js apps to use the new version.
Commit the changes to each repository.

Only then will your applications be synchronized with the new feature in the shared package.‌
In contrast, with a monorepo, all your projects live in a single codebase. You can make changes directly to the shared utilities or packages, and they will be immediately available to both the React.js and Next.js applications. Since the apps depend on the local version within the same codebase, there's no need to manage versions or publish updates, making the process much easier and more streamlined.

---

What is Turborepo?
Turborepo, helps you manage your monorepo more efficiently and run tasks much faster.
Turborepo understands your monorepo structure, recognizing that packages need to be built before your apps can be compiled.
It never performs the same work twice. When you run a build, Turborepo generates a hash for the process. If it doesn't find the hash in its cache, it runs the build and then stores the result in the cache for future use. This means that the next time you run a build on the same source files, Turborepo skips the build process and restores the results from the cache.
Turborepo also supports remote caching. With remote caching, you can share the Turborepo cache across your entire team and CI/CD pipeline, resulting in even faster builds and significant time savings.
GitHub repository of the final Turborepo app:
https://github.com/matinkhani/monorepo-turborepo

---

Workspace
The main building block of a monorepo is the workspace. Each application and package you build will be its own workspace. A workspace is essentially a folder that contains a package.json file. Each workspace can declare its own dependencies, rely on other workspaces, export code for others to use, and have its own scripts.

---

Package.json in different package managers
Regardless of which package manager you use (e.g., npm, pnpm, or Yarn), the package.json file in your React.js, Next.js, or any other application defines the dependencies for shared packages. This file specifies which libraries or packages you want to use in your application.
npm or yarn:
{
  "dependencies": {
    "next": "latest", // External dependency
    "@repo/ui": "*" // Internal dependency
  }
}
pnpm:
{
  "dependencies": {
    "next": "latest", // External dependency
    "@repo/ui": "workspace:*" // Internal dependency
  }
}
The asterisks (*) indicate that the dependency is not tied to a specific version; instead, it resolves to the version available within the workspace.

---

Root Workspace
In addition to the individual workspaces for your different applications and packages, there's also a root workspace that contains all of these packages and apps together. The root workspace is a good place to specify dependencies that are used across the entire monorepo. You can also define tasks that apply to the entire monorepo, rather than just individual workspaces.
root package.json:
npm:
{
  "name": "TURBOREPO",
  "private": true,
  "scripts": {
    "build": "turbo build",
    "dev": "turbo dev",
    "lint": "turbo lint",
    "format": "prettier --write \\"**/*.{ts,tsx,md}\\""
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
pnpm:
{
  "name": "TURBOREPO",
  "private": true,
  "scripts": {
    "build": "turbo build",
    "dev": "turbo dev",
    "lint": "turbo lint",
    "format": "prettier --write \\"**/*.{ts,tsx,md}\\""
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
also in pnpm workspaces define in a file called pnpm-workspace.yaml:
packages:
  - "apps/*"
  - "packages/*"

---

Getting Started
To begin, install Turborepo globally so you can run Turbo commands in your terminal. If you prefer not to use Turbo commands, you can use npm, pnpm, or Yarn instead.
npm: npm install turbo --global
yarn: yarn global add turbo
pnpm: pnpm install turbo --global
If you like, you can also install example projects from the Turborepo documentation, such as with-docker, with-prisma, with-tailwind, with-vue-nuxt, and more.
Install Turborepo
To install Turborepo, run the following command:
npx create-turbo@latest
After the installation, you will see a starter repository with the following structure:
Apps Folder: Contains two deployable Next.js applications:

web
docs

node_modules: A shared folder for all the modules and packages used across all of your apps and packages.
Packages Folder: Contains different packages or libraries that are shared between your apps, including:

eslint-config: Shared ESLint configuration for your apps.
typescript-config: Shared TypeScript configuration for your apps.
ui: A React.js library to be shared between your apps.

Now, everything is in the same codebase, making it easier to manage changes and run build tasks and scripts faster.
Turbo.json
In addition to the root package.json, you will also have a turbo.json file. This file defines the tasks for running various scripts or processes. Any script you want to run using Turbo commands from package.json must be registered in your turbo.json file:
{
  "$schema": "<https://turbo.build/schema.json>",
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
Web App Package.json:
If we dive into the web app, which is a Next.js application, we can find a package.json file with the following content:
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
The name field is important because it specifies the name of the workspace. This name is used to refer to this specific workspace when running scripts or defining dependencies between local workspaces.
Each package is its own workspace. They can have their own scripts, their own dependencies, and they can depend on each other.
As you can see, our web app depends on our UI library, which is a workspace package, as indicated in the dependencies section:
"dependencies": {
    ...
    "@repo/ui": "*",
    ...
  },
We also have some devDependencies, such as eslint-config and typescript-config, which are listed in the packages folder:
"devDependencies": {
    ...
    "@repo/eslint-config": "*",
    "@repo/typescript-config": "*",
    ...
  }
Run Our Monorepo
To run our monorepo, use the following commands:
npm: npm run dev
Yarn: yarn dev
pnpm: pnpm run dev

This will start both the Web App and the Docs App. When you run npm run dev, it executes the turbo dev command specified in the root package.json. Turborepo then runs the dev script for each of your packages and apps. In this case, it runs the dev script for both the Web and Docs workspaces, which are Next.js applications.
How the UI Library Works in Apps
Let's examine how the UI library is used in the apps. Navigate to web/app/page.tsx and make some changes to see how the UI library integrates:
import { Button } from "@repo/ui/button";

export default function Home() {
  return <Button appName="web">Click ME</Button>;
}
You can see that the button is coming from our shared UI library. This is made possible by referencing the shared UI library in the dependencies of the web app.
If we navigate to the local UI workspace in the packages folder, we can find the following in its package.json:
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
In the package.json, you'll find an "exports" field. To use UI components in other packages or apps, you need to export those components here.
Alternatively, you can export all components by configuring the "exports" field like this:
"exports": {
    "./components/*": [
      "./src/components/*.tsx", // use src if you have a src folder
      "./src/components/*.ts" // same for the components
    ]
  },
That's how easy it is to share a UI library between two completely separate Next.js applications.
In the devDependencies, you can also see that we use ESLint and TypeScript configurations. This demonstrates how packages can depend on other packages within the monorepo.
Typescript Configuration
Let's take a quick look at the shared TypeScript configuration, as both our Web and Docs applications use TypeScript. In the packages/typescript-config directory, you'll find three JSON files: base.json, nextjs.json, and react-library.json.
In the package.json of this workspace, you can see the name assigned to it. This name allows us to reference this configuration as a dependency in other workspaces.
{
  "name": "@repo/typescript-config",
  "version": "0.0.0",
  "private": true,
  "license": "MIT",
  "publishConfig": {
    "access": "public"
  }
}
Now, if we return to the Web App and open the tsconfig.json file, we can see that it extends the typescript-config workspace. It directly imports the nextjs.json file, allowing us to inherit and extend those configurations within our Web Application.
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
How to Run Separately
As you saw, we ran the dev script for both applications in parallel. To run a script on a specific workspace individually, use the following commands:
npm: npm run dev --workspace web
pnpm: pnpm --filter web run dev
Yarn: yarn workspace web run dev

You can replace web with the name of any other workspace as needed.
How to Add a Dependency to a Specific Package
To add a dependency to a specific workspace, such as adding axios to the Web App, follow these steps. Similar to running scripts, you can use the filter flag to specify the workspace where you want to add the dependency:
pnpm: pnpm --filter web add -D axios
npm: npm install axios -D -w web
Yarn: yarn workspace web add --dev axios

Replace web with the name of your target workspace as needed.
You can confirm the addition by checking the package.json file in the Web workspace:
"devDependencies": {
    ...
    "axios": "^1.7.3",
    ...
  }
However, if we check the package.json in the Docs App, we'll see that axios is not listed there, and it's definitely not present in the root folder either. This confirms that the dependency was only added to the Web App.
Alternatively, you can navigate to the desired workspace directory and install the package directly from there.

---

Build Process and Cache Behavior
Let's examine the build process and caching behavior in Turborepo, which can be quite beneficial. When you run pnpm run build, it executes the turbo run build command, as specified in the turbo.json file:
"build": {
    "dependsOn": ["^build"],  
    "inputs": ["$TURBO_DEFAULT$", ".env*"],
    "outputs": [".next/**", "!.next/cache/**"]
  },
It runs the build script for every package or app that has a specified build script.
Let's check the terminal to see what happens during this process. As you can see, it ran the build script for both the Docs and Web workspaces. You'll notice a "cache miss" because this is the first time the build process is being executed, so there is no cached build available yet.
Now let's run the build process again:
This time, the build process completes much more quickly. This is because the outputs haven't changed since the previous run, resulting in a cache hit. Turborepo simply replays the cached output from the previous step. Compare the time taken for the first build (cache miss) with the second build (cache hit) to see the difference.

---

Create a Turborepo with Shared UI (Shadcn and Tailwindcss)
Now that we've covered setting up workspaces and managing dependencies in a monorepo with Turborepo, let's create a Turborepo that includes two Next.js applications and a shared UI library using Shadcn and Tailwind CSS.
Install Turborepo
Run the following command to install Turborepo:
npx create-turbo@latest
After running this command, you'll be prompted with the following questions:
Where would you like to create your Turborepo? .
Which package manager do you want to use? Choose pnpm (I prefer to continue with pnpm).

Once the installation is complete, you'll see the following structure:
Two Next.js applications
Three packages

Next, we'll add Tailwind CSS and Shadcn to the UI library.
Install Tailwindcss & Shadcn
First, we need to install Tailwind CSS in our UI package:
pnpm --filter ui add tailwindcss
You can confirm the installation by checking the package.json file in the UI package.
"devDependencies": {
    ...
    "tailwindcss": "^3.4.7",
    ...
  },
After installing Tailwind CSS, create a Tailwind configuration file by running npx tailwindcss init. This will generate a tailwind.config.js file in the UI package:
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [],
  theme: {
    extend: {},
  },
  plugins: [],
}
Next, we need to create a globals.css file. I prefer to place it in the src directory of the UI package. Create the file and add the necessary Tailwind CSS directives:
cd .\packages\ui\src\
echo. > globals.css
And paste this lines into it:
@tailwind base;
@tailwind components;
@tailwind utilities;
After setting up Tailwind CSS, we need to install Shadcn. Change to the packages/ui directory and install Shadcn:
cd .\packages\ui\
pnpm dlx shadcn-ui@latest init

After running the installation command, you'll be prompted with the following questions:
Would you like to use TypeScript (recommended)?
Answer: yes
Which style would you like to use?
Answer: Default (I prefer to use the default style)
Which color would you like to use as the base color?
Answer: Slate
Where is your global CSS file?
Answer: src/globals.css
Would you like to use CSS variables for colors?
Answer: no
Are you using a custom Tailwind prefix (e.g., tw-)?
Answer: ui-
Where is your tailwind.config.js located?
Answer: tailwind.config.js
Configure the import alias for components:
Answer: @repo/ui/components
Configure the import alias for utils:
Answer: @repo/ui/lib/utils
Are you using React Server Components?
Answer: no
Write configuration to components.json. Proceed?
Answer: yes

After the installation is complete, you will see the following changes:
src Directory: Contains lib and components folders.
components.json: Created in the root of the UI package.

{
  "$schema": "<https://ui.shadcn.com/schema.json>",
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
Next, we need to update the tsconfig.json in the ui package to reflect the changes made for Shadcn and Tailwind CSS. Modify the tsconfig.json file to include the new paths and configurations:
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
Next, we need to modify the tailwind.config.js file in the ui package. First, rename the file to tailwind.config.ts. Then, update the file with the following content:
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
Now, let's add a Shadcn button to see how it integrates with our setup. Follow these steps to include a Shadcn button component in your UI package:
run pnpm dlx shadcn-ui@latest add button
and you can see the Button component created in components/ui folder:
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
If you need you can create a global postcss.config.mjs in ui package to use in other apps:
/** @type {import('postcss-load-config').Config} */
const config = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};

export default config;
Now we're going to ui's package.json to change exports so we can use ui configs and ui components in other apps and packages:
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
How to Use the Shared Library in Apps
We have created this UI library to be used across all apps. Additionally, we can utilize the Tailwind CSS configuration from the UI library in all apps. By having a main Tailwind configuration, we can derive multiple Tailwind configurations from it.
For example, in the Web App, we want to use the main Tailwind configuration while also adding app-specific classes and styles. To achieve this, we can create a tailwind.config.ts for the Web App that imports the main Tailwind configuration and extends it with additional classes specific to the Web App.
First, we need to install the UI library in our app. If you haven't deleted the Web and Docs apps, they already have the UI library included in their dependencies:
web/package.json:
"dependencies": {
	   ...
    "@repo/ui": "workspace:*",
		 ...
  },
If you haven't ui in your app dependency, you must to add it like above and run pnpm install.
Now we can use all exported components and configs in our Web App.
First, to use Tailwind CSS in the Web App, follow these steps:
1.run pnpm --filter web add -D tailwindcss postcss autoprefixer
2.create tailwind.config.ts :
if you just want to use global tailwind config in Web App you just need do this:
export * from "@repo/ui/tailwind.config";
but if you want also add new classes and styles just for use in Web App you can do this:
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
i just added a test color to see if it's working.
and after that we have to create a postcss.config.mjs in web and paste this into it:
export { default } from "@repo/ui/postcss.config";
and for the last part, we need to import our globals css into web layout.tsx like this:
import "@repo/ui/globals.css";
Now we can use components and configs from ui in our Web App.
web/app/page.tsx:
import { Button } from "@repo/ui/components/ui/button";

export default function Home() {
  return <Button variant="destructive">Click Me</Button>;
}
you can see we used button from ui package in web application.
That's it! We have successfully created a monorepo using Turborepo and set up a UI library that shares components and configurations across all our applications.
