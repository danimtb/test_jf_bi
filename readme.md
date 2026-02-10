# Testing JFrog CLI Conan integration to generate build-info

The aim of this repository is to test the Conan commands that can produce a Build-info
with the JFrog CLI integration and provide the equivalent commands for
the Conan Artifactory extension commands for comparison.

## Requirements

- JFrog CLI version 2.88.0
- Conan client (any version 2.2X version, tested with 2.24.0)
- JFrog Artifactory instance with a conan-local repository

## Configure the Conan client

```bash
$ conan remote add conan-local <url-of-the-artifactory-repo>
$ conan remote login conan-local <user> -p <password>
$ conan config install https://github.com/conan-io/conan-extensions.git
$ conan art:server add my_artifactory <artifactory-instance-url> --user=<user> --password=<password>
```

Here is an example of the configuration steps when running an Artifactory instance on your local machine
with a `conan-local` repo created and a user `admin` and password `password`:

```bash
$ conan remote add conan-local http://localhost:8081/artifactory/api/conan/conan-local
$ conan remote login conan-local admin -p password
$ conan config install https://github.com/conan-io/conan-extensions.git
$ conan art:server add my_artifactory http://localhost:8081/artifactory --user=admin --password=password
```

## Expected results summary by case

| Case | Scenario | Expected (summary) |
|------|----------|--------------------|
| **1** | `conan create` without deps | Build-info with **hello** package and its **artifacts** (conanfile.py, manifests, conan_package.tgz). |
| **2** | `conan create` with deps | Build-info with **hello** and its artifacts, plus **zlib** and its artifacts (optional: include deps). |
| **3** | `conan install` with conanfile.txt | Build-info with **full modules** (e.g. zlib as its own module) and **artifacts** for each (incl. cached deps via `--add-cached-deps`). |
| **4** | `conan install --requires ... --build ...` without local recipe | Must **work without a conanfile** in the folder; build-info with the built package (zlib) and its artifacts. |

---

## Test cases

### Test case 1 ([``create``](/create/) folder)

```bash
$ cd create
```

JFrog CLI commands:

```bash
$ jf conan create --build-name my_build --build-number 1
$ jf rt bp my_build 1 --dry-run > cli-bi1.json
```

- **Compare:** [cli-bi1.json](create/cli-bi1.json) (CLI) · [conan-bi1.json](create/conan-bi1.json) (Conan extension)

Conan commands:

```bash
$ conan create -f json > create1.json
$ conan upload hello/1.0 --remote conan-local
$ conan art:build-info create create1.json my_build 1 conan-local --server my_artifactory > conan-bi1.json
```

### Test case 2 ([``create_with_deps``](/create_with_deps/) folder)

```bash
$ cd create_with_deps
```

JFrog CLI commands:

```bash
$ jf conan create --build-name my_build --build-number 2
$ jf rt bp my_build 2 --dry-run > cli-bi2.json
```

- **Compare:** [cli-bi2.json](create_with_deps/cli-bi2.json) (CLI) · [conan-bi2.json](create_with_deps/conan-bi2.json) (Conan extension)

Conan commands:

```bash
$ conan create --build=missing -f json > create2.json
$ conan upload hello/1.0 --remote conan-local
$ conan art:build-info create create2.json my_build 2 conan-local --with-dependencies --server my_artifactory > conan-bi2.json
```

### Test case 3 ([``install``](/install/) folder)

```bash
$ cd install
```

JFrog CLI commands:

```bash
$ jf conan install --build-name install --build-number 1
$ jf rt bp install 1 --dry-run > cli-bi-install.json
```

- **Compare:** [cli-bi-install.json](install/cli-bi-install.json) (CLI) · [conan-bi-install.json](install/conan-bi-install.json) (Conan extension)

Conan commands:

```bash
$ conan install -f json > install.json
$ conan upload hello/1.0 --remote conan-local
$ conan art:build-info create install.json install 1 conan-local --add-cached-deps --server my_artifactory > conan-bi-install.json
```

### Test case 4 ([``no_recipe``](/no_recipe/) folder)

```bash
$ cd no_recipe
```

JFrog CLI commands:

```bash
$ jf conan install --requires zlib/1.3.1 --build zlib/1.3.1 --build-name no_recipe --build-number 1
...
Error] failed to collect Conan build info: failed to initialize: failed to load conanfile: no conanfile.py or conanfile.txt found in C:\Users\user\test_jf_bi\no_recipe
```

- **Compare:** *(no CLI output — command fails)* · [conan-bi-no_recipe.json](no_recipe/conan-bi-no_recipe.json) (Conan extension)

Conan commands:

```bash
$ conan install --requires zlib/1.3.1 --build zlib/1.3.1 -f json > no_recipe.json
$ conan upload zlib/1.3.1 -r conan-local
$ conan art:build-info create no_recipe.json no_recipe 1 conan-local --server my_artifactory > conan-bi-no_recipe.json
```
