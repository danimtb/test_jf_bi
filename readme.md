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

## Test cases

### Test case 1 ([``create``](/create/) folder)

```bash
$ cd create
```

JFrog CLI commands:

```bash
$ jf conan create --build-name my_build --build-number 1
$ jf rt bp my_build 1 --dry-run
```

Conan commands:

```bash
$ conan create -f json > create1.json
$ conan upload hello/1.0 --remote conan-local
$ conan art:build-info create create1.json my_build 1 conan-local --server my_artifactory
```

### Test case 2 ([``create_with_deps``](/create_with_deps/) folder)

```bash
$ cd create_with_deps
```

JFrog CLI commands:

```bash
$ jf conan create --build-name my_build --build-number 2
$ jf rt bp my_build 2 --dry-run
```

Conan commands:

```bash
$ conan create -f json > create2.json
$ conan upload hello/1.0 --remote conan-local
$ conan art:build-info create create2.json my_build 2 conan-local --server my_artifactory
```

### Test case 3 ([``txt``](/txt/) folder)

```bash
$ cd txt
```

JFrog CLI commands:

```bash
$ jf conan install --build-name install --build-number 1
$ jf rt bp install 1 --dry-run
```

Conan commands:

```bash
$ conan install -f json > install.json
$ conan upload hello/1.0 --remote conan-local
$ conan art:build-info install.json install 1 conan-local --server my_artifactory
```

### Test case 4 ([``no_recipe``](/no_recipe/) folder)

```bash
$ cd no_recipe
```

JFrog CLI commands:

```bash
$ jf conan install --requires zlib/1.2.11 --build zlib/1.2.11 --build-name no_recipe --build-number 1
...
Error] failed to collect Conan build info: failed to initialize: failed to load conanfile: no conanfile.py or conanfile.txt found in C:\Users\user\test_jf_bi\no_recipe
```

Conan commands:

```bash
$ conan install --requires zlib/1.2.11 --build zlib/1.2.11 -f json > no_recipe.json
$ conan upload zlib/1.2.11 -r conan-local
$ conan art:build-info create no_recipe.json no_recipe 1 conan-local --server my_artifactory
```
