# fetch-git

Fetch a git repository by tag or by a branch and revision.

The result can be composed with `builtins.filterSource`.

**Note:** specifying a tag name is much faster than specifying a
branch and revision, as Git is able to perform a much shallower fetch
with the tag itself.

## Arguments for fetching a tag

* `name`: Optional, the name of the source archive
* `url`: Required, the git-compatible URL to a repository
* `tag`: Required, the name of the tag to fetch

### Example:

```nix
fetch-git {
  url = "https://github.com/nixos/nixpkgs.git";
  tag = "18.03";
}
```

or:

```nix
fetch-git rec {
  name = "nixpkgs-at-tag-${tag}";
  url = "https://github.com/nixos/nixpkgs.git";
  tag = "18.03";
}
```

## Arguments for fetching a branch

* `name`: Optional, the name of the source archive
* `url`: Required, the git-compatible URL to a repository
* `branch`: Required, the name of the branch which contains `revision`
* `revision`: Required, the specific revision of `branch` to fetch

### Example

```nix
fetch-git {
  url = "https://github.com/nixos/nixpkgs.git";
  branch = "release-18.03";
  revision = "8bf6df2b8e8fc9e2f012901af48214feff918d9d";
}
```

or:

```nix
fetch-git rec {
  name = "nixpkgs-${branch}-at-${revision}";
  url = "https://github.com/nixos/nixpkgs.git";
  branch = "release-18.03";
  revision = "8bf6df2b8e8fc9e2f012901af48214feff918d9d";
}
```

## Return value

### Success: a derivation

* `outPath`: a Path to an archive of the repository
* `debug`: a list of strings, debug messages from the fetcher.

**Note:** this can be used directly as a source, you don't need to
explicitly reference the `outPath`:

```nix
runCommand "copy-nixpkgs-readme"
  {
    buildInputs = [ git ];
    src = fetch-git rec {
      name = "nixpkgs-${branch}-at-${revision}";
      url = "https://github.com/nixos/nixpkgs.git";
      branch = "release-18.03";
      revision = "8bf6df2b8e8fc9e2f012901af48214feff918d9d";
    };
  }
  ''
    cp $src/README.md $out
  '';
```

### Failure

* `error`: possibly `no-cache-directory`, `commit-not-found`,
  `tag-not-found`.
* `message`: Error details
* `debug`: An array of strings, each line being a debug message from
the fetcher

## Notes

Successful results are cached in [XDG_CACHE_HOME], under
`nix-fetchers`.

## DIFFERENCES FROM builtins.fetchGit

### Morally pure, assuming good Git hygiene and tags are immutable

Specifying a git commit and branch is effectively immutable and
pure

Specifying a tag name, in a team where you can trust all actors
to behave responsibly, is also immutable and pure. Note: this
can probably only be feasibly true in an organization with strict
repository protection!

This purity allows for reliable caching. The upstream fetchGit, by
contrast, allows fetching the tip of any branch (or arbitrary ref) and
has a convenience cache with a configurable TTL unless a specific
commit is specified.

### Uses separate Git repositories per upstream Git repository

Sacrifices some caching possibilities for much faster initial
syncs. Upstream's fetch-git uses a single Git repository for all
cloned repos. This works great if all you're fetching is Nixpkgs,
but is extremely slow for initial syncs of other, non-nixpkgs
repositories.


[XDG_CACHE_HOME]: https://specifications.freedesktop.org/basedir-spec/0.7/ar01s03.html
