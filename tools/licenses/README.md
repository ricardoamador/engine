# Update License Files

## Apply patch generated by CI
If you're not working on a Linux box then you can't auto-generate license
files. A workaround is provided via CI.

Your build will fail one or more CI checks if your license files are not
correct. Open the failing CI check. In the CI output you will find a
patch diff that represents the changes that need to be made to your
license files.

> [!WARNING]
> Do this only if the diff does not change _which_ licenses are being
> included, but affects just the `FILE:` metadata and so forth in the
> golden files. If the actual content of the licenses has changed, you
> will need to rerun the script from a Linux box to update the actual
> LICENSE file, as described below.

```sh
cd flutter/ci/licenses_golden
```

If the file is to `my/patch/file` locally. The `-p2` strips two path
components from the file paths in the diff. Adjust as necessary.

```sh
patch -p2 < my/patch/file
```

If the patch file is copied in your clipboard. On Mac, you may apply the patch from there.

```sh
pbpaste | patch -p2
```

## Check for license changes

> [!WARNING]
> Only works on Linux.

This script has two sets of output files: "goldens", which describe
the current license state of the repository, and the actual real
LICENSE file, which is what matters.

We look at changes to the goldens to determine if there are any actual
changes to the licenses.

To update the goldens, make sure you've rebased your branch to the
latest upstream main and then run the following in this directory:

```
dart pub get
gclient sync -D
rm -rf ../../../out/licenses
dart --enable-asserts lib/main.dart --src ../../.. --out ../../../out/licenses --golden ../../ci/licenses_golden
```

In order for the license script to work correctly, you need to remove
any untracked files inside `engine/src` (the buildroot), not just
`engine/src/flutter`.

Once the script has finished, copy any affected files from
`../../../out/licenses` to `../../ci/licenses_golden` and add them to
your change, and examine the diffs to see what changed.

```
cp ../../../out/licenses/* ../../ci/licenses_golden
git diff
```

> [!IMPORTANT]
> If the only changes are to what files are included, then you're good
> to go. However, if any of the _licenses_ changed, whether new licenses
> are added, old ones removed, or any have their content changed in
> _any_ way (including, e.g., whitespace changes), or if the affected
> libraries change, **you must update the actual license file**.

The `sky/packages/sky_engine/LICENSE` file is the one actually
included in product releases and the one that should be updated any
time the golden file changes in a way that involves changes to
anything other than the `FILE` lines. To update this file, run:

```
dart pub get
gclient sync -D
dart --enable-asserts lib/main.dart --release --src ../../.. > ../../sky/packages/sky_engine/LICENSE
```

The bots do not verify that you did this step, it's important that you
do it! When reviewing patches, if you see a change to the golden
files, check to see if there's a corresponding change to the LICENSE
file!

## Testing

To run the tests:

```
dart pub get
find -name "*_test.dart" | xargs -n 1 dart --enable-asserts
```
