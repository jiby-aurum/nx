---
title: Storybook 7 Migration Generator Examples
description: This page contains examples for the @nrwl/storybook:migrate-7 generator.
---

{% callout type="info" title="Available on Nx v15.9" %}
This is a new feature available on Nx v15.9.0. If you are using an older version of Nx, please [upgrade](/packages/nx/documents/migrate).
{% /callout %}

{% callout type="warning" title="Storybook 7 is in beta" %}
[Storybook version 7 is still in beta](https://storybook.js.org/blog/7-0-beta/), and so is the Nx support for it. Things are evolving dynamically, so it would be better to _avoid using in production_ on Nx. If you want to use the stable, [6.5 version](https://storybook.js.org/releases/6.5), please go to the [Storybook plugin overview guide](/packages/storybook) to get started.
{% /callout %}

{% callout type="info" title="Setting up Storybook 7 in a new workspace" %}
For setting up Storybook version 7 in a new Nx workspace, or a workspace that does NOT already have Storybook configured already, please refer to our [Storybook 7 setup guide](/packages/storybook/documents/storybook-7-setup).
{% /callout %}

Storybook 7 is a major release that brings a lot of new features and improvements. You can read more about it in the [Storybook 7 beta announcement blog post](https://storybook.js.org/blog/7-0-beta/). Apart from the new features and improvements it introduces, it also brings some breaking changes. You can read more about them in the [Storybook 7 migration docs](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#from-version-65x-to-700) and the [Storybook 7 migration guide](https://chromatic-ui.notion.site/Storybook-7-migration-guide-dbf41fa347304eb2a5e9c69b34503937). Do note that _version 7 is still in beta_, and so is the Nx support for it. Things are evolving dynamically, so it would be better to _avoid using in production_ on Nx.

You can now migrate your existing Nx workspace with Storybook configuration to use Storybook version 7. To help you, Nx offers the `@nrwl/storybook:migrate-7` generator. This generator will help you migrate your existing Storybook setup to version 7.

Now let's see how this generator works and how you can use it!

## How the generator works under the hood

The `@nrwl/storybook:migrate-7` generator essentially does the following things:

### Step 01

It calls the Storybook CLI upgrade script:

```bash
npx storybook@next upgrade --prerelease
```

This script will upgrade your Storybook dependencies to the latest version, as explained in the [Storybook documentation](https://storybook.js.org/docs/7.0/react/configure/upgrading).

### Step 02

It prepares all your project-level `.storybook/main.js|ts` files, so that
the Storybook automigration scripts can run successfully. This means that it makes the following adjustments to your files:

- Remove the "`as StorybookConfig`" typecast from the `.storybook/main.ts` files, if you have any `.storybook/main.ts` files with typecast, since it is not needed any more
- Remove the "`path.resolve`" calls from the Next.js Storybook configuration in project-level `.storybook/main.js|ts` files, if it exists, since it breaks the Storybook automigration scripts

### Step 03

It calls the Storybook CLI automigrate script, for each one of your projects that have Storybook configured. It does that by passing the `--config-dir` flag and the `--renderer` flag, for each one of your projects that has Storybook configured. An example command would look like this:

```bash
npx sb@next automigrate --config-dir apps/my-react-app/.storybook --renderer @storybook/react
```

This script will make changes to your Storybook configuration files, and other changes to your repository, to make it work for Storybook 7, as explained in the [Storybook documentation](https://storybook.js.org/docs/7.0/react/configure/upgrading).

### Step 04

After the Storybook CLI automigrate scripts have run, some additional adjustments are made to your workspace, to make sure that everything is working as expected. These adjustments are as follows:

- Remove the "`vite-tsconfig-paths`" plugin from the Storybook configuration files for Vite projects, since it's no longer needed in v7
- Add the "`viteConfigPath`" option to the Storybook builder options for Vite projects, since now Storybook needs the path to the Vite config file
- Change the import package for the `StorybookConfig` type to be framework specific (e.g. from `@storybook/common` to `@storybook/react-vite` for React projects using Vite)
- Add the "`lit`" package to your workspace, if you are using Web Components
- Remove the "`uiFramework`" option from your project's Storybook targets

Our generator is based on the guide to migration using the Storybook CLI, sp please refer to the [Storybook 7 migration guide](https://chromatic-ui.notion.site/Storybook-7-migration-guide-dbf41fa347304eb2a5e9c69b34503937) for more information.

{% callout type="warning" title="Don't use in production" %}
Please take extra care when migrating your existing Storybook setup to version 7 on your Nx workspace. Do not use in production, since it's still in beta, and the Nx support is not stable yet.
{% /callout %}

Let's see how you can use the generator effectively!

{% callout type="info" title="Commit your changes" %}
It is advised that you start with a clean git history before running this generator, since it is going to be making lots of changes to your workspace.
{% /callout %}

## Run the generator

```bash
npx nx g @nrwl/storybook:migrate-7
```

You can run this generator using the above command, without passing any options. This will start the migration process for all your projects that have Storybook configured. The logs will explain what is happening in every step, and the logs are mixed Nx and Storybook CLI logs. During the process you will be prompted by the Storybook CLI to accept the automigration scripts. You can read more about that in the next section.

When the generator finishes, you will see a summary of the changes that were made to your workspace, and it will also create a new file, called `storybook-migration-summary.md` at the root of your project, which will contain a list of all the changes that were made to your workspace.

### Accept the automigration prompts

The Storybook CLI (running through our generator) will prompt you to run some code generators and modifiers.

You can say `yes` to these prompts, which are usually the following (there may be more or less, depending on your setup,
and depending on the latest versions of the Storybook CLI - this code is NOT managed by Nx, but by Storybook):

- `mainjsFramework`: It will try to add the `framework` field in your project's `.storybook/main.js|ts` file.
- `eslintPlugin`: installs the `eslint-plugin-storybook`
- `newFrameworks`: removes unused dependencies (eg. `@storybook/builder-webpack5`, `@storybook/manager-webpack5`, `@storybook/builder-vite`)
- `autodocsTrue`: adds `autodocs: true` to your project's `.storybook/main.js|ts` file

### Check the result

Once the generator finishes, and the Storybook CLI automigration scripts have run, you should check the result. Examples of migrated `.storybook/main.js|ts` files would look like this:

#### Full example for Angular projects

Here is an example of a project-level `.storybook/main.js|ts` file for an Angular project that has been migrated to Storybook version 7:

```ts {% fileName="apps/my-angular-app/.storybook/main.js" %}
const config = {
  stories: ['../src/app/**/*.stories.@(js|jsx|ts|tsx|mdx)'],
  addons: ['@storybook/addon-essentials'],
  framework: {
    name: '@storybook/angular',
    options: {},
  },
};

export default config;
```

#### Full example for React projects with Vite

Here is an example of a project-level `.storybook/main.js|ts` file for a React project using Vite that has been migrated to Storybook version 7:

```ts {% fileName="apps/my-react-app/.storybook/main.js" %}
const config = {
  stories: ['../src/app/**/*.stories.@(js|jsx|ts|tsx|mdx)'],
  addons: ['@storybook/addon-essentials'],
  framework: {
    name: '@storybook/react-vite',
    options: {
      builder: {
        viteConfigPath: 'apps/rv1/vite.config.ts',
      },
    },
  },
};

export default config;
```

### Make sure that all works by running Storybook

You can now use Storybook 7 beta! 🎉

```bash
npx nx build-storybook PROJECT_NAME
```

and

```bash
npx nx storybook PROJECT_NAME
```

## Run the generator by automatically accepting the Storybook CLI prompts

You can run the generator with the `--autoAcceptAllPrompts` flag, which will automatically accept all the Storybook CLI prompts. This is useful if you want to run the generator in a CI environment, or if you want to run the generator in a script. Or if you are sure that you want to accept all the prompts!

```bash
npx nx g @nrwl/storybook:migrate-7 --autoAcceptAllPrompts
```

The Storybook CLI may still ask you about some things, but mostly it should just run the whole migration suite uninterrupted.

## Run the migration manually

Nx gives you the ability to run all the migration steps one by one, manually, but still with the help of our migrator. To help you out with the commands that you need to run, Nx will print out the instructions if you run the generator with the `--onlyShowListOfCommands` flag, like this:

```bash
npx nx g @nrwl/storybook:migrate-7 --onlyShowListOfCommands
```

Essentially, the way to run the migration manually is the following:

1. Call the Nx generator to show you the list of commands:
   `npx nx g @nrwl/storybook:migrate-7 --onlyShowListOfCommands`
2. Call the Storybook upgrade script:
   `npx storybook@next upgrade --prerelease`
3. Call the Nx generator to prepare your files for migration. The steps are explained in [Step 02](#step-02) above.
   `nx g @nrwl/storybook:migrate-7 --onlyPrepare`
4. Call the Storybook automigrate scripts for each one of the projects using Storybook (the `@nrwl/storybook:migrate-7` will give you the list of all the commands)
5. Call the Nx generator to finish the migration. The steps are explained in [Step 04](#step-04) above.
   `nx g @nrwl/storybook:migrate-7 --afterMigration`

## Report any issues and bugs

Since this is a beta version, and the Nx support is still evolving, there are bound to be some issues and bugs. Please report any issues and bugs you find [on the Nx GitHub page](https://github.com/nrwl/nx/issues/new/choose) or on the [Storybook GitHub page](https://github.com/storybookjs/storybook/issues/new/choose).
