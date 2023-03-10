# better_git_for_bazel
A more efficient `git_repository()` for Bazel when used with large repos.

If you need to bring in just a tiny slice of a large repo, this can improve fetching time drastically.

This is a fork of the `git_repository()` that ships with Bazel, so it is a drop-in replacement.
It adds a new attribute called `sparse`, which maps to Git's
[sparse checkout](https://git-scm.com/docs/git-sparse-checkout) feature, making it possible to
depend only on specific directories in an external repo without having to checkout the whole tree.
It also utilizes Git's [partial clone](https://git-scm.com/docs/partial-clone) feature to avoid
cloning all the files in the tree.

These capabilities depend both on a new enough local Git and a repository server with the necessary
features. If `git_repository()` fails when using these features, it will degrade gracefully
and revert to the old behavior.

Example `WORKSPACE` file:
```Starlark
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

BETTER_GIT_COMMIT = "90fd341fe40950a328cec69d4d8f65cff9f3a89d"
BETTER_GIT_SHA256 = "8c55b1f3d01eaa35c46afed5aa18842fae8a8d63e67519a0af3ba670e51d1803"

http_archive(
    name = "better_git_for_bazel",
    urls = ["https://github.com/rpwoodbu/better_git_for_bazel/archive/{}.tar.gz".format(BETTER_GIT_COMMIT)],
    strip_prefix = "better_git_for_bazel-{}".format(BETTER_GIT_COMMIT),
    sha256 = BETTER_GIT_SHA256,
)

load("@better_git_for_bazel//:git.bzl", "git_repository")

git_repository(
    name = "my_enormous_repo",
    remote = "ENORMOUS_REPO_REMOTE",  # Replace with your repo.
    commit = "ENORMOUS_REPO_COMMIT",  # Replace with your commit.
    sparse = [
        # These are directories in the external repo that you want to checkout.
        # The only other files that will be downloaded are those in the root of the repo.
        # If there are transitive dependencies on directories not in this list,
        # things will break.
        #
        # If `sparse` is omitted, the original behavior is preserved:
        # it does a checkout of the full tree.
        "some_project/src",
        "toolchains",
    ],
)
```

[Compare](https://github.com/rpwoodbu/better_git_for_bazel/compare/upstream...master) the changes against the upstream version.
