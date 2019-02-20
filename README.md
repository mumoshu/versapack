# anydep

`anydep` is a general-purpose project/application dependency manager. It is a tool like `godep`, `bundler`, `npm`, but for any sets of files.

## Rationale

To be written

## Getting started

Install `helm` and `helm-s3`:

```
$ brew install kubernetes-helm
$ helm init --client-only
$ helm plugin install https://github.com/hypnoglow/helm-s3.git
```

Grab the latest release of `variant` and put the executable under any directory listed in `PATH`:

https://github.com/mumoshu/variant/releases/tag/v0.22.0

## Usage

### Creating your app with deps managed by anydep

Place a `anydep.yaml`:

```yaml
name: myapp
version: 1.0.0
#Description: any description here
repository: s3://examplecom-anydep-ap-northeast-1/charts
dependencies:
- name: commons
  version: 0.5.0
  repository: s3://examplecom-anydep-ap-northeast-1/charts
```

Resolve and fetch all the dependencies into `vendor/anydep/<pkg name>` by running `ensure -update`:

```
$ anydep ensure --update
```

This will create a lock file named `anydep.lock`. It is strongly encounrated to commit this into a versioned repository along with your `anydep.yaml`, usually a Git repository.

```
$ git add anydep.yaml anydep.lock
```

Note that you are not necessarily required to commit `vendor/anydep/<pkg name>` into the repository. Just run `anydep ensure` to restore all the dependencies listed in the lock file:

```
$ anydep ensure
```

If you're familiar with golang, `anydep ensure` corresponds to `dep ensure`, and `anydep ensure --update` is `dep ensure -update`.

If you're familiar with ruby/bundler, take `anydep ensure` as `bundle vendor`, and `anydep ensure --update` as `bundle update`.

### Publishing a package

`anydep up` packages up the whole project under the current working directory and uploads it into the Helm Chart repository described in your `anydep.yaml`.

Let's say you have a `anydep.yaml` like:

```yaml
name: testdep
version: 1.1.0
repository: s3://examplecom-anydep-ap-northeast-1/charts
```

and the project root looks like:

```console
$ tree ..
├── anydep.yaml
├── anydep.lock
├── myconfig.yaml
├── myscripts
│   └── myshinyscript
```

Running `anydep up` packages up all the files listed above into a tarball, and then uploads it to the S3 bucket `examplecom-anydep-ap-northeast-1` with an object key prefixed with `charts/`:

```
$ anydep up
```

Great! Now that your anydep package is uploaded to the repository backed by the S3 bucket, you can import any files contained within the package into another anydep-managed projects.

## Multi-Project Support

You can collocate two or more `anydep` projects at the same level of the project tree.

Just create one `anydep.yaml` variant per your project:

```console
$ tree ..
├── variantdep.yaml
├── helmfiledep.yaml
```

In each `anydep.*.yaml`, please don't forget to set `vendorPath` to something other than the default `vendor/anydep`, so that deps from one collocated project doesn't override those from the another:

```yaml
# variantdep.yaml
vendorPath: vendor/variant
lockFile: variantdep.lock
```

```yaml
# helmfiledep.yaml
vendorPath: vendor/helmfile
lockFile: helmfiledep.lock
```

Finally, provide a `-c CONFIG_FILE` flag to `anydep` so that it uses the config file to resolve and fetch dependencies:

```console
$ anydep -c variantdep.yaml ensure --update
$ anydep -c variantdep.yaml ensure

$ anydep -c helmfiledep.yaml ensure --update
$ anydep -c helmfiledep.yaml ensure
```

## Use-cases

Use it with `helmfile`, `variant`, or any shared configuration files or bash scripts. Your imagination is the only limit!
