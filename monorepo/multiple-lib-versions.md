---
layout: page
title: Multiple Lib Versions
---

**You:**

In a monorepo managed by nx which uses npm as the the package manager, is it possible to us two different versions of a library, for example, ag-grid@29 and ag-grid@30?

------------------

**ChatGPT:**

In a monorepo managed by Nx and using npm as the package manager, it is indeed possible to use two different versions of a library, such as `ag-grid@29` and `ag-grid@30`, but it requires careful management. Here's how you can achieve this:


- **Using Aliases in `package.json`**: You can leverage npm's aliasing feature to install two versions of the same package under different names. In your `package.json`, you can alias `ag-grid` like this:

```json
Copy code
{
  "dependencies": {
    "ag-grid-v29": "npm:ag-grid@29",
    "ag-grid-v30": "npm:ag-grid@30"
  }
}

```
This approach will install `ag-grid@29` as `ag-grid-v29` and `ag-grid@30` as `ag-grid-v30`. You can then import these versions in your projects as needed.
- **Scoped Imports in Different Applications/Libraries**: In an Nx monorepo, you typically have multiple applications and libraries. You can configure each application or library to use a different version of the library. This is done by importing the aliased version in the respective application or library.
- **Manage Dependencies in `nx.json` and `workspace.json`**: Ensure that your dependencies are correctly managed in the `nx.json` and `workspace.json` files. This ensures that the right versions are used in the right context, avoiding conflicts.
- **Be Aware of Potential Issues**: Using multiple versions of a library in the same project can lead to issues, especially if there are peer dependencies or if the library interacts with global states or styles. Test thoroughly to ensure compatibility.
- **Consider Using Nx's Module Federation (if applicable)**: If you are working with micro-frontends, Nx's Module Federation support can help isolate different versions of a library in different micro-frontends, reducing the risk of conflicts.

Remember, while this approach is technically feasible, it can introduce complexity and potential maintenance challenges. It's generally recommended to keep your dependencies consistent across a monorepo when possible.



------------------

**You:**

Let's say that there is already an application in the monorepo that uses ag-grid 30 and a standard package.json without a project.json.  I now run a command, nx g @nx/react:app app2 to create a new app.  How would I modify app2 to use ag-grid version 31?

------------------

**ChatGPT:**

To modify the new application `app2` in your Nx monorepo to use `ag-grid` version 31, while another application is already using `ag-grid` version 30, you can follow these steps:


- **Install `ag-grid` Version 31 as an Alias**: First, you need to install `ag-grid` version 31 in your monorepo as an alias, to avoid conflicts with the existing version 30. You can do this by modifying your `package.json` to include the aliased version. Add the following line to the `dependencies`:

```json
Copy code
"ag-grid-v31": "npm:ag-grid@31"

```
Then run `npm install` or `npm update` to install the aliased package.
- **Modify `app2` to Use the Aliased Version**: In your new application `app2`, you will now import `ag-grid` using the alias. Wherever you would normally import from `ag-grid`, replace it with `ag-grid-v31`. For example:

```javascript
Copy code
import { AgGridReact } from 'ag-grid-v31';

```
This ensures that `app2` uses version 31 of `ag-grid`, while the rest of your monorepo can continue using version 30.
- **Update `tsconfig` if Necessary**: If TypeScript is being used, you may need to update the `tsconfig.json` in `app2` to correctly resolve the aliased package. Add a `paths` configuration if necessary:

```json
Copy code
{
  "compilerOptions": {
    "paths": {
      "ag-grid-v31": ["node_modules/ag-grid-v31"]
    }
  }
}

```
- **Test Your Application**: After making these changes, thoroughly test `app2` to ensure that it is using `ag-grid` version 31 correctly and that there are no compatibility issues with other parts of your monorepo.
- **Consider Project-Level `package.json` (Optional)**: In some cases, especially in larger monorepos, it might be beneficial to have a `package.json` specific to `app2` to manage its dependencies separately. This can be done in Nx, but it adds complexity and may not be necessary depending on the size and scope of your project.

By following these steps, you can manage multiple versions of the same package in a Nx monorepo. However, be mindful of the potential for increased complexity and maintenance overhead, especially when dealing with multiple versions of key dependencies.



------------------

**You:**

Can you be more specific regarding step 1? Right now there is a package.json at the workspace root and in app1.  I'm assuming you mean "run npm install ag-grid@31 at the workspace root and then edit the package.json after the install?

------------------

**ChatGPT:**

Certainly! Let's clarify Step 1 with more specific instructions. Since you have a `package.json` at the workspace root and in `app1`, and you want to use a different version of `ag-grid` in `app2`, you'll need to handle the dependency management carefully to avoid conflicts. Here's how to do it:


- **Install `ag-grid` Version 31 with an Alias at the Workspace Root**:
   - First, open the terminal at the root of your monorepo workspace.
   - Run the following command to install `ag-grid` version 31 as an alias. This alias will allow you to install a specific version of `ag-grid` for use in `app2` without affecting `app1`.```bash
   Copy code
   npm install ag-grid-v31@npm:ag-grid@31

```
   - This command tells npm to install `ag-grid@31` but under the alias `ag-grid-v31`. This means `ag-grid@31` will be installed in your `node_modules` but you will import it as `ag-grid-v31` in your code.
- **Confirm the Changes in `package.json`**:
   - After running the install command, open your `package.json` at the workspace root.
   - You should see a new entry under `dependencies` (or `devDependencies` depending on where you want it) similar to:```json
   Copy code
   "ag-grid-v31": "npm:ag-grid@31"

```
   - This change in `package.json` ensures that anyone else working on the monorepo or any build systems will also install this specific version of `ag-grid` when they install dependencies.
- **Use the Aliased Version in `app2`**:
   - In your `app2` project, whenever you need to use `ag-grid`, import it using the alias:```javascript
   Copy code
   import { AgGridReact } from 'ag-grid-v31';

```
   - This will make sure `app2` is using version 31 of `ag-grid`.

By following these steps, you add `ag-grid@31` to your workspace in a way that it does not conflict with the existing `ag-grid@30` used in `app1`. This approach is beneficial in a monorepo setup where different projects might need to use different versions of the same package.



------------------

**You:**

You mentioned using a package.json for app2.  What would be the benefit?  This would have to be created so what would go into the package.json?  It's ok to have a project.json and a package.json?

------------------

**ChatGPT:**

Having a separate `package.json` for `app2` in a monorepo, alongside the existing `project.json`, is an optional approach that can offer several benefits, especially in complex projects. However, it also introduces additional complexity. Let's explore the benefits and what you would include in such a `package.json`:

### Benefits of a Project-Specific `package.json`

- **Dependency Isolation**: It allows you to manage dependencies specific to `app2` without affecting other applications or libraries in the monorepo. This is particularly useful when different projects within the monorepo have divergent dependency requirements.
- **Clearer Dependency Management**: It makes it easier to see which dependencies are being used by `app2`, rather than having to determine which of the root-level dependencies are relevant to this specific app.
- **Version Management**: If you need to upgrade or modify dependencies for `app2` without impacting other applications, a dedicated `package.json` simplifies this process.
- **Simplified CI/CD**: In some CI/CD setups, having a separate `package.json` can streamline the build process for `app2`, especially if it has unique build steps or dependency requirements.

### Contents of the Project-Specific `package.json`
If you decide to create a `package.json` for `app2`, it should include:


- **Project-Specific Dependencies**: List dependencies that are unique or have different versions than those in the root `package.json`. For example, your `ag-grid-v31` dependency would go here.
- **Scripts**: Any npm scripts specific to `app2`. For instance, scripts for building, testing, or running development servers specifically for `app2`.
- **Other Standard Fields**: Like any `package.json`, it would include fields such as `name`, `version`, `description`, etc., relevant to `app2`.

### Project.json vs. Package.json

- **`project.json`**: In the Nx monorepo, `project.json` is used for configuring project-specific settings like builder options, architect/target configurations, etc. It's more about the build and development process than about dependency management.
- **`package.json`**: This file, when used at the project level, focuses on the dependencies and potentially scripts specific to that project.

### Considerations

- **Maintenance Overhead**: Having separate `package.json` files for different projects in a monorepo adds an extra layer of maintenance. You need to ensure dependencies are up to date across multiple files.
- **Duplication**: There's a risk of duplicating dependencies between the root and project-level `package.json` files, which can lead to confusion.
- **Tooling Support**: Ensure that your tooling and CI/CD processes are capable of handling project-specific `package.json` files appropriately.

In conclusion, whether or not to use a project-specific `package.json` depends on the complexity of your project, your team's workflow, and the specific requirements of `app2`. For many projects, especially smaller ones, sticking to a single root `package.json` is sufficient and easier to manage. But for larger, more complex projects, the additional granularity can be beneficial.



------------------

**You:**

After I create a app-level package.json, How does the install of ag-grid work?  Do I change the current directory to app2 and install? Do I still need to use aliases?  If I have one dependency in app2's package.json do I have to add all the others or does nx combine  the root package.json with the app package.json?

------------------

**ChatGPT:**

Creating an app-level `package.json` in an Nx monorepo for `app2` and managing its dependencies requires a slightly different approach compared to managing dependencies at the root level. Let's go through the steps and considerations:


- **Creating and Setting Up the App-Level `package.json`**:
   - Navigate to the `app2` directory in your monorepo.
   - Create a `package.json` file specifically for `app2`. This can be done manually or by running `npm init` in the `app2` directory to generate a basic `package.json` file.
- **Installing Dependencies Locally in `app2`**:
   - In the `app2` directory, you can now install dependencies specific to this application. For example, to install `ag-grid` version 31, you would run:```bash
   Copy code
   npm install ag-grid@31

```
   - This will add `ag-grid@31` as a dependency in the `app2/package.json`.
- **Dependency Management**:
   - The dependencies in `app2/package.json` are local to `app2`. Nx will not automatically combine dependencies from the root `package.json` and `app2/package.json`. Each `package.json` is independent in terms of the dependencies it lists.
   - You need to explicitly add all the dependencies that `app2` requires in its `package.json`. This includes not only `ag-grid` but also any other libraries specific to `app2`. However, common dependencies that are shared across the monorepo can still be managed at the root level.
- **Avoiding Aliases**:
   - If you are using a separate `package.json` for `app2`, you might not need to use aliases unless there are conflicting versions of the same package required by different apps in your monorepo. In the case of `ag-grid`, if no other app is using a different version or if you don't mind updating them all to the same version, you can directly install the required version without an alias.
- **Building and Running the App**:
   - Nx should be able to recognize and handle the separate `package.json` for building and serving `app2`. Ensure that your `nx.json` and `angular.json` (or the equivalent configuration files for your Nx setup) are correctly configured for `app2`.
- **Considerations**:
   - Having separate `package.json` files can lead to duplication and additional maintenance, so it's important to carefully consider whether this approach is necessary for your project.
   - Ensure that your development team is aware of this structure and understands how to manage dependencies in this setup.
- **Version Control**:
   - Don't forget to add the new `package.json` for `app2` to your version control system to ensure consistency across your team and CI/CD pipelines.

By following these steps, you can manage `app2`'s dependencies independently, which can be particularly useful for larger projects with specific needs or when dealing with multiple versions of the same package across different apps in a monorepo.



------------------

**You:**

So having a separate package.json more the dependency declaration from the root to the app level, but it does not eliminate the need to use an alias if two different versions of ag-grid are used in different apps?

------------------

**ChatGPT:**

That's correct. Having a separate `package.json` for `app2` in your Nx monorepo does indeed allow you to declare dependencies specific to `app2`, independent from the root-level `package.json`. However, this alone does not eliminate the need for using aliases if you have different applications within the same monorepo that require different versions of the same package, such as `ag-grid`.

Here's a more detailed explanation:


- **Purpose of Separate `package.json`**:
   - The primary purpose of having a separate `package.json` in `app2` is to isolate its dependencies. This means you can declare and manage `app2`'s dependencies independently of the rest of the monorepo.
   - This approach is especially useful when `app2` has specific dependency versions or sets of dependencies that are different from other applications in the monorepo.
- **Need for Aliases**:
   - If your monorepo has multiple applications that need different versions of the same package (e.g., `app1` uses `ag-grid@30` and `app2` uses `ag-grid@31`), you'll still need to use aliases.
   - The reason for this is that npm and Node.js resolve dependencies from the `node_modules` folder at the root of the monorepo. Without aliases, installing different versions of the same package would cause conflicts.
   - By using aliases, you can install multiple versions of the same package under different names. For example, `ag-grid-v30` for version 30 and `ag-grid-v31` for version 31. These versions can then coexist in the monorepo without conflicts.
- **Implementing Aliases with Separate `package.json`**:
   - Even with a separate `package.json` in `app2`, you would still use aliases for `ag-grid` in the root `package.json` to ensure that both versions are available in the monorepo.
   - In `app2`, you would import `ag-grid` using the alias corresponding to the version you need. The same applies to other applications in the monorepo.
- **Version Control and Consistency**:
   - It's important to ensure consistency across different applications in your monorepo regarding dependency versions, especially when dealing with complex dependencies like `ag-grid`.
   - Properly documenting and communicating these dependency management strategies within your development team is crucial for maintaining a healthy and functional monorepo environment.

In summary, while a separate `package.json` in `app2` allows for independent management of dependencies, the use of aliases is still necessary when different applications within the same monorepo require different versions of the same package. This approach helps to avoid conflicts and ensure that each application gets the correct version of the package it depends on.



------------------

**You:**

Does nx provide a way of seeing or visualizing which applications or libs are using which versions of ag-grid or whether there are conflicts?

------------------

**ChatGPT:**

As of my last update in April 2023, Nx, the popular monorepo management tool, offers several features for visualizing and understanding dependencies within the monorepo, but it does not provide a built-in feature specifically for visualizing which versions of a particular library (like `ag-grid`) are used across different applications or libraries, or for identifying version conflicts directly.

However, you can leverage the existing Nx tools and other strategies to gain insight into your dependency usage:


- **Nx Dependency Graph**:
   - Nx provides a powerful Dependency Graph feature that visualizes the dependencies between the applications and libraries in your monorepo.
   - You can generate and view this graph by running `nx dep-graph` in your terminal.
   - While this graph shows the relationships between projects in your monorepo, it does not detail the versions of third-party libraries used in each project.
- **Manual Inspection**:
   - For third-party dependencies, you usually need to manually inspect the `package.json` files of each application or library to see which version of a library like `ag-grid` is being used.
   - If you have separate `package.json` files for different applications or libraries, you'll need to check each one. Otherwise, you'll refer to the root `package.json`.
- **Scripts and Tooling**:
   - You can write custom scripts to parse the `package.json` files in your monorepo and extract the information about specific dependencies. This could be a Node.js script that reads all `package.json` files and reports the versions of `ag-grid` used.
   - There are also third-party tools available for analyzing `package.json` files, although they may require customization to provide a concise report on specific dependencies like `ag-grid`.
- **Nx Console and Other Nx Tools**:
   - The Nx Console, a graphical interface for Nx, can make it easier to manage and understand your workspace, though it doesn’t specifically target dependency version visualization.
   - Other Nx tools and plugins might offer additional insights into your project structure and dependencies.
- **Consistency Checks**:
   - Regularly reviewing and updating your dependencies can help prevent conflicts. This can be part of your development workflow, where you periodically check for version inconsistencies or conflicts.
- **Use of Aliases**:
   - If you're using aliases for managing different versions of the same library, it becomes easier to track which application/library is using which version by simply looking at the import statements or the specific alias used in the `package.json` files.

As of now, Nx focuses more on the structural and build dependencies within a monorepo rather than on the specifics of third-party library versions. For detailed tracking of library versions, a combination of manual inspection and custom scripting is often the most effective approach.



------------------

**You:**

When a different version of ag-grid is installed by npm using an alias, does it separate that install by using a folder named from the alias?

------------------

**ChatGPT:**

When you install a different version of a package using an alias in npm, npm does not create a separate folder named after the alias in the `node_modules` directory. Instead, the package is installed in a folder named after the original package, but the version and references in the dependency tree are handled according to the alias.

Here's how it works:


- **Installation**: When you install a package with an alias, for example:

```bash
Copy code
npm install ag-grid-v31@npm:ag-grid@31

```
This command tells npm to install `ag-grid@31` but refer to it in your project as `ag-grid-v31`.
- **Node Modules Structure**: In the `node_modules` directory, you will find the `ag-grid` package installed under its original package name (e.g., `node_modules/ag-grid`). The version installed will be the one specified in the alias command (`31` in this case).
- **package.json and package-lock.json**: In your `package.json`, the dependency will be listed under the alias name you provided (`ag-grid-v31`). The `package-lock.json` file will also reflect this alias, linking `ag-grid-v31` to the correct version of `ag-grid`.
- **Importing in Code**: In your JavaScript or TypeScript files, you would import the package using the alias:

```javascript
Copy code
import { AgGridReact } from 'ag-grid-v31';

```
This tells your module bundler or Node.js to look for `ag-grid-v31` in your `node_modules`, which is then resolved to the correct version of `ag-grid`.
- **Dependency Resolution**: Npm handles the resolution of the aliased package internally, ensuring that when you request `ag-grid-v31`, it uses the version installed under the `ag-grid` package directory in `node_modules`.
- **Coexistence of Multiple Versions**: If you have multiple versions of `ag-grid` installed via aliases (e.g., `ag-grid-v30` and `ag-grid-v31`), npm manages these versions so that the correct one is used based on how you import it in your code. Each version will be resolved to the appropriate `ag-grid` installation in `node_modules`, but they won't interfere with each other due to the way npm handles the dependency tree.

In summary, npm uses aliases more as a reference mechanism in the dependency management system rather than creating separate directories for each aliased version. The key is how npm resolves these aliases when your application requests a specific package version.



------------------

**You:**

What folder structure does npm create when both ag-grid 30 and 31 are installed?

------------------

**ChatGPT:**

When you install two different versions of the same package (like `ag-grid` versions 30 and 31) in an npm project, npm manages these versions within the `node_modules` directory. However, the exact folder structure can vary depending on how these packages are installed (directly or as dependencies of other packages). Let's consider two common scenarios:

### 1. Direct Installation as Aliased Dependencies
If you install both versions directly using aliases, like this:

```bash<button class="flex gap-1 items-center"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" class="icon-sm"><path fill-rule="evenodd" clip-rule="evenodd" d="M12 4C10.8954 4 10 4.89543 10 6H14C14 4.89543 13.1046 4 12 4ZM8.53513 4C9.22675 2.8044 10.5194 2 12 2C13.4806 2 14.7733 2.8044 15.4649 4H17C18.6569 4 20 5.34315 20 7V19C20 20.6569 18.6569 22 17 22H7C5.34315 22 4 20.6569 4 19V7C4 5.34315 5.34315 4 7 4H8.53513ZM8 6H7C6.44772 6 6 6.44772 6 7V19C6 19.5523 6.44772 20 7 20H17C17.5523 20 18 19.5523 18 19V7C18 6.44772 17.5523 6 17 6H16C16 7.10457 15.1046 8 14 8H10C8.89543 8 8 7.10457 8 6Z" fill="currentColor"></path></svg>Copy code</button>
npm install ag-grid-v30@npm:ag-grid@30
npm install ag-grid-v31@npm:ag-grid@31

```
The resulting folder structure in `node_modules` will be something like:

```css<button class="flex gap-1 items-center"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" class="icon-sm"><path fill-rule="evenodd" clip-rule="evenodd" d="M12 4C10.8954 4 10 4.89543 10 6H14C14 4.89543 13.1046 4 12 4ZM8.53513 4C9.22675 2.8044 10.5194 2 12 2C13.4806 2 14.7733 2.8044 15.4649 4H17C18.6569 4 20 5.34315 20 7V19C20 20.6569 18.6569 22 17 22H7C5.34315 22 4 20.6569 4 19V7C4 5.34315 5.34315 4 7 4H8.53513ZM8 6H7C6.44772 6 6 6.44772 6 7V19C6 19.5523 6.44772 20 7 20H17C17.5523 20 18 19.5523 18 19V7C18 6.44772 17.5523 6 17 6H16C16 7.10457 15.1046 8 14 8H10C8.89543 8 8 7.10457 8 6Z" fill="currentColor"></path></svg>Copy code</button>
node_modules/
├── ag-grid/
└── .bin/
    ├── ag-grid-v30 -> ../ag-grid/...
    └── ag-grid-v31 -> ../ag-grid/...

```

- **`ag-grid/`**: This folder will contain the actual code of the `ag-grid` package. The specific version that ends up in this folder can depend on the order of installation and npm's resolution strategy. It might be one of the two versions you installed, or even both, if npm decides they can coexist.
- **`.bin/`**: The `ag-grid-v30` and `ag-grid-v31` aliases may appear in the `.bin/` directory as symlinks pointing to the respective `ag-grid` version executables, if such executables exist.

### 2. Installation as Dependencies of Different Local Packages
If `ag-grid@30` and `ag-grid@31` are installed as dependencies of different local packages within a monorepo or a similar setup, you might end up with a structure like this:

```bash<button class="flex gap-1 items-center"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" class="icon-sm"><path fill-rule="evenodd" clip-rule="evenodd" d="M12 4C10.8954 4 10 4.89543 10 6H14C14 4.89543 13.1046 4 12 4ZM8.53513 4C9.22675 2.8044 10.5194 2 12 2C13.4806 2 14.7733 2.8044 15.4649 4H17C18.6569 4 20 5.34315 20 7V19C20 20.6569 18.6569 22 17 22H7C5.34315 22 4 20.6569 4 19V7C4 5.34315 5.34315 4 7 4H8.53513ZM8 6H7C6.44772 6 6 6.44772 6 7V19C6 19.5523 6.44772 20 7 20H17C17.5523 20 18 19.5523 18 19V7C18 6.44772 17.5523 6 17 6H16C16 7.10457 15.1046 8 14 8H10C8.89543 8 8 7.10457 8 6Z" fill="currentColor"></path></svg>Copy code</button>
node_modules/
├── ag-grid/               # Contains one of the versions, say, v31
packages/
├── package1/
│   └── node_modules/
│       └── ag-grid/       # Local node_modules for package1, containing v30
└── package2/
    └── node_modules/      # package2 might use the root node_modules

```

- **Root `node_modules/`**: This will contain one version of `ag-grid`, let's say version 31.
- **Local `node_modules/` in `package1`**: This is a separate `node_modules` directory within `package1` that contains version 30 of `ag-grid`.
- **`package2`**: It might use the version of `ag-grid` from the root `node_modules`, which in this case is version 31.

### Important Considerations

- **Version Resolution**: The exact behavior can depend on the version of npm you're using and the specific project setup. Npm tries to deduplicate and optimize the `node_modules` structure where possible.
- **Lock Files**: The `package-lock.json` file will have entries for both versions, specifying



------------------

