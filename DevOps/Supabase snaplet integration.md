---
tags: dev, devOps, 창업
---

# Supabase

![I absolutely love Supabase!](/recipes/supabase/snappy-holding-supabase-logo.svg)

Snaplet can provide data for your Supabase project!

Pick your own path:

- [Generate data for your Supabase local development stack](/recipes/supabase#seeding-a-supabase-local-development-stack)
- [Restore data from your production Supabase project to your local development stack](/recipes/supabase#restoring-to-a-supabase-local-development-stack)

<Callout>
We've spent a lot of time ensuring that Snaplet works seamlessly with Supabase, however, if you experience any issues with this integration or this guide, feel free to come chat with us on [Discord](https://app.snaplet.dev/chat).
</Callout>

## Generate data for your Supabase local development stack

Supabase makes it easy to set up a [local environment](https://supabase.com/docs/guides/cli/local-development) linked to your Supabase project. Snaplet can improve your local development by seeding your local database with data, making coding and testing more efficient and accurate.

In this short guide, we will use the [Nextjs + Supabase template](https://vercel.com/templates/next.js/supabase), to demonstrate how one could use the `snaplet generate` command to seed their local Supabase stack.

### Prerequisites

- Supabase account
- [Supabase CLI installed](https://supabase.com/docs/guides/cli/getting-started)
- [Snaplet CLI installed](https://docs.snaplet.dev/getting-started/start-here#getting-started-with-snaplet-cli)

#### 1. Create a Supabase project

Create a new project in the Supabase Dashboard by visiting https://supabase.com/

![Creating a supabase project](/recipes/supabase/supabase_new_project.webp)

#### 2. Clone the Nextjs + Supabase template

2.1. Create a Next.js app using the Supabase starter template by running the `npx` command:

```bash
npx create-next-app -e with-supabase
```

**2.2.** Use `cd` to change into the newly created directory:

```bash
cd name-of-new-app
```

#### 3. Apply your migrations to the Supabase project.

**3.1.** Link your Nextjs starter template to your Supabase project

```bash
supabase init
```

**3.2.** Apply the migrations in `/supabase/migrations` to your project

> Migrations are required to be applied to your target database, as the `snaplet generate` needs a database structure to generate data for.

```bash
supabase db push
```

**3.3.** Start your Supabase local stack

```bash
supabase start
```

This command will print out your the following:

```bash
Started supabase local development setup.

         API URL: http://localhost:54321
          DB URL: postgresql://postgres:postgres@localhost:54322/postgres
      Studio URL: http://localhost:54323
    Inbucket URL: http://localhost:54324
        anon key: eyJh......
service_role key: eyJh......
```

**3.4.** Copy the DB URL - we will use this in the next step (step 4) as your target database credentials.

#### 4. Set up your generate config

**4.1.** Create a `snaplet.config.ts` file in the root of your project.

```bash
# select "yes" to create your `.snaplet/config.json`
# input the DB URL copied from the previous step for your target database
snaplet config setup
```

> The target database credentials will be used to introspect your database and generate a base config

**4.2.** In the `snaplet.config.ts` update the `generate` statement to create 10 users and 20 todos, the todos should be assigned to at least one of the 10 users.

```ts snaplet.config.ts
generate: {
    plan({ snaplet, pipe }) {
      return pipe(
        // 10 users
        snaplet.users({ count: 10 }),
        // 20 todos, with one of the 10 users assigned to each todo
        snaplet.todos({ count: 20 }, { autoConnect: true })
      );
    },
},
```

Once you have gone through all the steps, you are now ready to seed your database.

#### 5. Reset and seed your local development database

**5.1.** Run the `snaplet generate` to populate the project seed script in the `supabase/seed.sql` file.

```bash
snaplet generate --sql > supabase/seed.sql
```

**5.2** Reset your local development database, this will run all the existing migrations and applu the seed script.

```bash
supabase db reset
```

**5.3** Check your local database to see if the seed script was applied.

Now if you visit the local studio URL (http://localhost:54323) and inspect the **"todo"** table, you should see 20 todo items.

![20 todos in the studio dashboard](/recipes/supabase/20_todos.webp)

In your project if you move the `app/_examples/client-component/page.tsx` to `app/todos/page.tsx` and then visit http://localhost:3000/todos you should see the following:

![list of todos in nextjs app](/recipes/supabase/nextjs_todos.webp)

### All done!

You have now successfully seeded your local Supabase development stack. You can now use the `snaplet generate` command to seed your local database with any data you need.

If you have any questions or feedback, please feel free to reach out to us on [Discord](https://app.snaplet.dev/chat).

## Restoring to a Supabase local development stack

Supabase makes it easy to start a project with a dedicated PostgreSQL database. However, achieving a development environment populated with data closely resembling producation, still requires some set up.
This guide will help you integrate Snaplet with your Supabase project, allowing you to restore a snapshot of your production database into your local development stack.

#### 1. Connect your source database (Production Database)

The first thing you'll want to do is navigate to https://www.snaplet.dev and sign up for a new account (it's free). Once you have successfully signed up for a new account, you'll begin the onboarding process...

![Snaplet onboarding select team name](/recipes/supabase/onboarding_start.webp)

On the “Connect database” step click on "Connect Supabase" to connect your Supabase account to Snaplet. Proceed through all the steps of authorizing your Supabase organization, selecting a Supabase project and giving us the password associated to the project.

![Snaplet onboarding connect your database](/recipes/supabase/connect_to_supabase.webp)

> The snapshots for your Supabase project will be set to only capture the `public` and `auth` schemas. Learn more [below](#excluding-specific-schemas-from-snapshots).

#### 2: Create a snapshot

Once that is done, continue to the next steps:

- Subsetting (skipping will create a snapshot with all your data).
- Transforming (skipping will leave all your table columns untouched).

Once you have gone through those steps, a new snapshot process will start and you will now be on the “capture step.” Wait for it to finish - this is the snapshot you will restore into your **target database.**

![Snaplet onboarding capturing your database](/recipes/supabase/onboarding_capture.webp)

#### 3: Set up your Supabase local development stack

Below is a summary of commands you will have to run in your terminal to get Supabase set up on your local machine. For a more detailed steps you can visit the Supabase docs [here](https://supabase.com/docs/guides/cli/local-development).

```bash >_&nbsp;terminal
# log in to the Supabase CLI
supabase login

# initial your Supabase configuration in your project folder
supabase init

# with docker running, start your Supabase services
supabase start
```

Running `supabase start` will output your Supabase credentials, copy the **DB_URL** field, as we will use it in your snaplet configuration file (step 5).

```bash >_&nbsp;terminal
Started supabase local development setup.

         API URL: http://localhost:54321
          DB URL: postgresql://postgres:postgres@localhost:54322/postgres
      Studio URL: http://localhost:54323
    Inbucket URL: http://localhost:54324
        anon key: eyJh......
service_role key: eyJh......
```

#### 4: Install the Snaplet CLI

1. Open your terminal and run `curl -sL https://app.snaplet.dev/get-cli/ | bash`
2. Run `snaplet auth login`
3. Navigate to `https://app.snaplet.dev/access-token/cli` to get your access token.
4. Paste the access token in the terminal.

#### 5: Config setup

You're now ready to restore your production snapshot into your Supabase development project.

1. Navigate to your project directory
2. Run `snaplet config setup`

   1. You will be shown a warning to write `.snaplet/config` in your project directory. Select yes (y) to create the file.
   2. Then you will be asked for a **target database connection string.** . Paste in the **DB URL** you copied in step 3.

3. Run `snaplet project setup` - you will be presented with a list of projects, these are databases that are connected to your Snaplet account. Choose the project that contains the snapshot you created in step 2.

#### 6: Restore the data target

With all the above steps complete, we can now restore!

1. Run `snaplet snapshot restore --no-reset` in your project path.

> **Note on `--no-reset`**: This will skip the “reset” step of the restore command. So no existing schemas will be dropped. Learn more about data operations [here](https://docs.snaplet.dev/getting-started/restoring#more-granular-control-over-restorations)

![Supabase is fun!](/recipes/supabase/snappy-with-supabase-ball.svg)

### All done!

If you want to learn more about Snaplet, you can explore our docs. If you have any questions, feel free to reach out on [Discord](https://app.snaplet.dev/chat).

That's it! You're all done, and should have restored a version of your Supabase production database with transformed data into your target database. You can now code against production-realistic data.

### Troubleshooting

#### Warnings after restoring

When running the restore command, you may see warnings such as:

```text >_&nbsp;terminal
Could not drop schema "auth", Snaplet will try to truncate all tables and related objects as a fallback: error: must be owner of schema auth
[Schema] Warning: type "aal_level" already exists, statement: "CREATE TYPE auth.aal_level AS ENUM (...
```

Supabase includes a few schemas that are not owned by the `postgres` user, for example: **auth**, **graphql**, **realtime,** and **storage.** During the capture process, Snaplet will try to capture data for tables under these schemas (if it has permission to read data from them)

Your **target database** may already contain these schemas when restoring. The warnings just mean that when Snaplet attempted to restore these schemas (**auth, graphql**, etc) but was unable to drop the ones already existing in your **target database** (since the user is not an owner of these), and consequently it was not able to create any structure for them (since that structure still exists)

Snaplet will still make sure to clear all data for tables in these schemas and restore data for each of these tables to what is in the snapshot. In other words, **these warnings do not mean that the restore failed, but rather show you what Snaplet tried to do.**

If you aren't actually needing the data for some of these schemas, you can stop these warnings by excluding the schema from the captured snapshots. You can read more on how to do this [over here in our docs](https://docs.snaplet.dev/references/data-operations/exclude):

![Example of excluding a schema](/recipes/supabase/snaplet-supabase-schema-exclude.png)

---

#### Excluding specific schemas from snapshots

We recommand excluding the `storage` and `supabase_functions` schemas from your snapshots. This is because these schemas contain data that may not have relevant context of the target project you are restoring to.
> e.g: the storage schema may contain files that belong to your production project, but may not be relevant to your development project.

Your snapshot config should look something like this:

```ts snaplet.config.ts
defineConfig({
  select: {
    $default: false,
    public: true,
    auth: true,
    // when you enable an extension in your supabase project and have to create
    // a schema for it, you can manually include it here.
  }
})
```

During onboarding, this `select` option is automatically added to your config. In the web editor you will be shown a warning, if you choose to remove the `$default`

#### Restoring into a Supabase project

You may run into problems restoring to a Supabase project hosted in the cloud. This is because the snaplet restore command will drop your database first, before restoring any schemas and tables, which results in your project being in a broken state. The reason for this is that Snaplet was unable to restore schemas that Supabase requires a super user role to perform write operations.

To get around this, run the `restore` command with the `--no-reset` flag, e.g:

이거 개 중요!!
```bash >_&nbsp;terminal
snaplet snapshot restore --no-reset
```