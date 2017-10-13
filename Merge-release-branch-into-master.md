# Git workflow.
* `git checkout master`
* `git pull`
* `git checkout -b [user]-merge-relase-[tag version]` e.g. `git checkout -b fabian-merge-release-ios-v3.6.0`
* `git checkout [release branch]`
* `git pull`
* `git checkout [user]-merge-relase-[tag version]` e.g. `git checkout fabian-merge-release-ios-v3.6.0`
* `git merge [tag]` e.g. `git merge ios-v3.6.0`

# Conflict resolution.
It is likely you will encounter merge conflicts the solving strategy would be:
* For core gl files (`*.hpp` & `*.cpp`) prefer `master` version.
* For Android and iOS files (`*.java`, `*.graddle`, `*.xml`, `*.h`, `*.m`, `*.mm`) manual merge, prioritize release changes.

# Merge test.
Before making a PR use the following commands to test the merge.
* Core gl
  * `make test`
  * `make run-test`

* Android
  * `make android-check`

* iOS & macOS
  * `make iproj`
  * Select iosapp click run.
  * `make xproj`
  * Select macosapp click run.

# Pull Request.
If above tests passed.
* `git push origin [user]-merge-relase-[tag version]` e.g. `git push origin fabian-merge-release-ios-v3.6.0`
* Ping each team member whose changes will be affected by this merge.

# Final merge.
Once the PR is approved.
* `git checkout master`
* `git pull`
* `git merge [user]-merge-relase-[tag version]` e.g. `git merge fabian-merge-release-ios-v3.6.0` 
* `git push origin master`
* `git branch -D [user]-merge-relase-[tag version]` e.g. `git branch -D fabian-merge-release-ios-v3.6.0` 

**_Merging using github is discouraged._** 

In any case do not use `squash and merge` option as this will break git history and introduce orphaned commits.

# Troubleshooting.
## Android.
Error message:
```
* What went wrong:
Execution failed for task ':MapboxGLAndroidSDK:lint'.
> Lint found errors in the project; aborting build.
```

What went wrong: There are missing translations.

Solution: Notify the Android team.
