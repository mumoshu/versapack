# versapack

`versapack` is a versatile package manager. `gem` rather than `bundler` for Ruby, but for any sets of files.

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

### Creating your app with deps managed by versapack

Place a `versapack.yaml`:

```yaml
name: myapp
version: 1.0.0
#Description: any description here
repository: s3://examplecom-versapack-ap-northeast-1/charts
dependencies:
- name: commons
  version: 0.5.0
  repository: s3://examplecom-versapack-ap-northeast-1/charts
```

Resolve and fetch all the dependencies into `vendor/versapack/<pkg name>` by running `ensure -update`:

```
$ versapack ensure --update
```

This will create a lock file named `versapack.lock`. It is strongly encounrated to commit this into a versioned repository along with your `versapack.yaml`, usually a Git repository.

```
$ git add versapack.yaml versapack.lock
```

Note that you are not necessarily required to commit `vendor/versapack/<pkg name>` into the repository. Just run `versapack ensure` to restore all the dependencies listed in the lock file:

```
$ versapack ensure
```

If you're familiar with golang, `versapack ensure` corresponds to `dep ensure`, and `versapack ensure --update` is `dep ensure -update`.

If you're familiar with ruby/bundler, take `versapack ensure` as `bundle vendor`, and `versapack ensure --update` as `bundle update`.

### Publishing a package

`versapack up` packages up the whole project under the current working directory and uploads it into the Helm Chart repository described in your `versapack.yaml`.

Let's say you have a `versapack.yaml` like:

```yaml
name: testdep
version: 1.1.0
repository: s3://examplecom-versapack-ap-northeast-1/charts
```

and the project root looks like:

```console
$ tree ..
├── versapack.yaml
├── versapack.lock
├── myconfig.yaml
├── myscripts
│   └── myshinyscript
```

Running `versapack up` packages up all the files listed above into a tarball, and then uploads it to the S3 bucket `examplecom-versapack-ap-northeast-1` with an object key prefixed with `charts/`:

```
$ versapack up
```

Great! Now that your versapack package is uploaded to the repository backed by the S3 bucket, you can import any files contained within the package into another versapack-managed projects.

## Multi-Project Support

You can collocate two or more `versapack` projects at the same level of the project tree.

Just create one `versapack.yaml` variant per your project:

```console
$ tree ..
├── variantdep.yaml
├── helmfiledep.yaml
```

In each `versapack.*.yaml`, please don't forget to set `vendorPath` to something other than the default `vendor/versapack`, so that deps from one collocated project doesn't override those from the another:

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

Finally, provide a `-c CONFIG_FILE` flag to `versapack` so that it uses the config file to resolve and fetch dependencies:

```console
$ versapack -c variantdep.yaml ensure --update
$ versapack -c variantdep.yaml ensure

$ versapack -c helmfiledep.yaml ensure --update
$ versapack -c helmfiledep.yaml ensure
```

## Use-cases

Use it with `helmfile`, `variant`, or any shared configuration files or bash scripts. Your imagination is the only limit!
